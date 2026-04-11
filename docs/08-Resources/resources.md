---

# HMI Subsystem Resources

**Name:** Christo Jomon Joseph
**Team:** 305
**Board ID:** `0x43` ('C')

---

## Overview

Two async tasks run on the ESP32-S3. One handles the UI. The other handles UART.


The UI task drives an 8-page scrollable menu on the OLED (motor control, sensor readouts, gyro visualizer, TX monitor, snake game), runs the power-on/off animations, and reads button inputs. The UART task feeds incoming bytes into a state machine, validates packets, routes sensor data into the values dict, and sends motor commands to Liam's board on a 500 ms rate limit.

Shared state ties them together. When fresh sensor data lands, the UART task immediately refreshes whatever page is on screen at that moment.


---

### Main code

```python title="main.py"
import random, math, struct
import uasyncio as asyncio
from machine import Pin, SoftI2C, UART
import ssd1306, time
from team_config import *
from uart_protocol import build_packet, build_ack, route_packet, UARTReceiver

# --- hardware setup ---

# oled on i2c, mounted upside down so we flip it
i2c  = SoftI2C(scl=Pin(42), sda=Pin(41))
oled = ssd1306.SSD1306_I2C(128, 64, i2c)
oled.write_cmd(0xa0)   # mirror columns left-right
oled.write_cmd(0xc0)   # flip rows vertically
W, H = 128, 64

# uart to motor controller and sensor nodes
uart = UART(1, baudrate=9600, tx=Pin(43), rx=Pin(44))
uart_rx = UARTReceiver()

# status leds: blue=tx, green=rx, white=debug
led_tx    = Pin(2, Pin.OUT, value=0)
led_rx    = Pin(1, Pin.OUT, value=0)
led_debug = Pin(4, Pin.OUT, value=0)

# 8 nav buttons, active low with internal pullups
btn_up    = Pin(5,  Pin.IN, Pin.PULL_UP)
btn_left  = Pin(6,  Pin.IN, Pin.PULL_UP)
btn_right = Pin(7,  Pin.IN, Pin.PULL_UP)
btn_down  = Pin(15, Pin.IN, Pin.PULL_UP)
btn_start = Pin(17, Pin.IN, Pin.PULL_UP)
btn_stop  = Pin(16, Pin.IN, Pin.PULL_UP)
btn_sel   = Pin(18, Pin.IN, Pin.PULL_UP)
btn_debug = Pin(8,  Pin.IN, Pin.PULL_UP)

# button name -> pin lookup + previous state for edge detection
_BTNS = {
    'up': btn_up, 'dn': btn_down, 'lt': btn_left, 'rt': btn_right,
    'sel': btn_sel, 'start': btn_start, 'stop': btn_stop, 'dbg': btn_debug
}
_prev = {k: b.value() for k, b in _BTNS.items()}

# --- button helpers ---

# single press on falling edge only, no repeat while held
def pressed(key):
    cur = _BTNS[key].value()
    edge = (cur == 0 and _prev[key] == 1)
    _prev[key] = cur
    return edge

# sync button state so held buttons don't fire after an animation
def flush_btns():
    for k in _prev:
        _prev[k] = _BTNS[k].value()

# auto-repeat: first fire immediately, then hold delay, then fast repeat
_held_since = {}
_last_repeat = {}
_HOLD_DELAY  = 400   # ms hold before repeat kicks in
_REPEAT_MS   = 120   # ms between each repeat fire

# returns true on initial press and at each repeat interval
def held(key):
    cur = _BTNS[key].value() == 0
    now = time.ticks_ms()
    if cur:
        if key not in _held_since:
            _held_since[key] = now
            _last_repeat[key] = now
            return True
        elif time.ticks_diff(now, _held_since[key]) > _HOLD_DELAY:
            if time.ticks_diff(now, _last_repeat[key]) > _REPEAT_MS:
                _last_repeat[key] = now
                return True
    else:
        _held_since.pop(key, None)
        _last_repeat.pop(key, None)
    return False

# --- app state ---

_lcd_changed_at = 0    # debounce timestamp for power toggle
lcd_on      = False    # whether display is active
tx_enabled  = True     # uart transmit enabled
sel         = 0        # main menu cursor index
view_top    = 0        # first visible menu row for scrolling
in_sub          = False    # inside a subpage
in_gyro_sub      = False   # inside gyro sub-menu
gyro_sub_sel     = 0       # gyro sub-menu cursor
_rot_x = 0.0   # live gyro rotation in radians, drives 3d cube
_rot_y = 0.0
_rot_z = 0.0
debug_mode      = False    # raw data overlay active
VISIBLE     = 4       # menu rows that fit on screen
motor_speed = 0       # signed, neg=REV
motor_dir   = "FWD"
last_rx_raw = ""      # last uart packet hex for debug page

# live sensor data from uart, "--" means no data yet
values = {
    "temp": "--", "humid": "--", "camera_state": "--", "camera_error": "--",
    "gyro_x": "--", "gyro_y": "--", "gyro_z": "--",
    "distance": "--", "speed": "0%", "motor": "OFF", "batt": "--"
}

MENU = ["Motor Ctrl", "Values", "Env Sensor", "Camera", "Distance", "Gyro Data", "TX Monitor", "Play Game"]

# --- display helpers ---

def clr():   oled.fill(0)      # clear framebuffer
def show():  oled.show()       # push framebuffer to oled

# inverted header bar at top of screen
def header(title):
    oled.fill_rect(0, 0, W, 11, 1)
    x = max(0, (W - len(title) * 8) // 2)
    oled.text(title, x, 2, 0)

# separator line + hint text at bottom
def footer(hint):
    oled.hline(0, H - 10, W, 1)
    oled.text(hint[:16], 0, H - 8)

# --- pages ---

# main menu with scroll, selection highlight, scrollbar
def draw_menu():
    global view_top
    if sel < view_top:              view_top = sel
    elif sel >= view_top + VISIBLE: view_top = sel - VISIBLE + 1
    clr()
    header("HMI CTRL")
    for i in range(VISIBLE):
        idx = view_top + i
        if idx >= len(MENU): break
        y = 13 + i * 12
        if idx == sel:
            oled.fill_rect(0, y - 1, W - 4, 11, 1)
            oled.text(">" + MENU[idx], 2, y, 0)
        else:
            oled.text(" " + MENU[idx], 2, y)
    # scrollbar thumb on right edge
    bar_h = max(4, (H - 13) * VISIBLE // len(MENU))
    bar_y = 13 + ((H - 13 - bar_h) * view_top // max(1, len(MENU) - VISIBLE))
    oled.rect(W - 3, 13, 3, H - 13, 1)
    oled.fill_rect(W - 2, bar_y, 2, bar_h, 1)
    show()

# motor speed bar + direction, up/dn adjust speed, l/r flip direction
def page_motor():
    clr(); header("MOTOR CTRL")
    spd_abs   = abs(motor_speed)
    direction = "FWD" if motor_speed >= 0 else "REV"
    oled.text("Speed: {:3d}%".format(spd_abs), 2, 13)
    bar_w = max(1, (W - 6) * spd_abs // 100) if spd_abs > 0 else 0
    oled.rect(2, 23, W - 6, 7, 1)
    if bar_w: oled.fill_rect(2, 23, bar_w, 7, 1)
    oled.text("Dir:  " + direction, 2, 33)
    oled.text("UP/DN:spd L/R:dir", 0, 43)
    footer("SEL=back")
    show()

# camera state label lookup
_CAM_STATES = {0: "Off", 1: "Idle", 2: "Capturing"}

# combined sensor snapshot: temp, humidity, distance, camera
def page_values():
    clr(); header("VALUES")
    t = "{:.1f}C".format(values["temp"])   if values["temp"]   != "--" else "--"
    h = "{:.1f}%".format(values["humid"])  if values["humid"]  != "--" else "--"
    d = "{:.1f}cm".format(values["distance"]) if values["distance"] != "--" else "--"
    c = _CAM_STATES.get(values["camera_state"], str(values["camera_state"]))
    oled.text("Temp:   " + t, 2, 13)
    oled.text("Humid:  " + h, 2, 24)
    oled.text("Dist:   " + d, 2, 35)
    oled.text("Cam:    " + c, 2, 46)
    footer("SEL=back")
    show()

# temp + humidity detail page
def page_env():
    clr(); header("ENV SENSOR")
    t = "{:.1f}C".format(values["temp"])  if values["temp"]  != "--" else "--"
    h = "{:.1f}%".format(values["humid"]) if values["humid"] != "--" else "--"
    oled.text("Temp:  " + t, 2, 16)
    oled.text("Humid: " + h, 2, 32)
    footer("SEL=back")
    show()

# camera state + error code
def page_camera():
    clr(); header("CAMERA")
    c = _CAM_STATES.get(values["camera_state"], str(values["camera_state"]))
    e = str(values["camera_error"]) if values["camera_error"] != "--" else "--"
    oled.text("State: " + c, 2, 16)
    oled.text("Error: " + e, 2, 32)
    footer("SEL=back")
    show()

# distance sensor single readout
def page_distance():
    clr(); header("DISTANCE")
    d = "{:.1f} cm".format(values["distance"]) if values["distance"] != "--" else "--"
    oled.text("Dist: " + d, 2, 24)
    footer("SEL=back")
    show()

# gyro x/y/z raw values
def page_gyro():
    clr(); header("GYRO DATA")
    oled.text("X: " + str(values["gyro_x"]), 2, 14)
    oled.text("Y: " + str(values["gyro_y"]), 2, 26)
    oled.text("Z: " + str(values["gyro_z"]), 2, 38)
    footer("SEL=back")
    show()

# rotate a 3d point around x, then y, then z axes
def _rot3d(px, py, pz, rx, ry, rz):
    # rotate around x axis
    y1 = py * math.cos(rx) - pz * math.sin(rx)
    z1 = py * math.sin(rx) + pz * math.cos(rx)
    py, pz = y1, z1
    # rotate around y axis
    x1 = px * math.cos(ry) + pz * math.sin(ry)
    z1 = -px * math.sin(ry) + pz * math.cos(ry)
    px, pz = x1, z1
    # rotate around z axis
    x1 = px * math.cos(rz) - py * math.sin(rz)
    y1 = px * math.sin(rz) + py * math.cos(rz)
    return x1, y1, pz

# 3d rectangle that tilts with live gyro angles, perspective projected
def page_gyro3d():
    clr()
    oled.text("GYRO 3D", 40, 0)
    # define a flat rectangle in 3d space
    corners = [(-20,-12,0),(20,-12,0),(20,12,0),(-20,12,0)]
    pts = []
    for px, py, pz in corners:
        rx, ry, rz = _rot3d(px, py, pz, _rot_x, _rot_y, _rot_z)
        scale = 1.0 + rz * 0.01  # fake perspective based on z depth
        pts.append((int(64 + rx * scale), int(30 + ry * scale)))
    # draw edges + diagonals
    for i, j in [(0,1),(1,2),(2,3),(3,0),(0,2),(1,3)]:
        oled.line(pts[i][0], pts[i][1], pts[j][0], pts[j][1], 1)
    gx = values["gyro_x"]; gy = values["gyro_y"]; gz = values["gyro_z"]
    oled.text("X:{} Y:{} Z:{}".format(gx, gy, gz), 0, 55)
    show()

GYRO_MENU = ["3D Visual", "X Y Z Values"]

# gyro sub-menu picker between 3d viz and raw values
def page_gyro_menu():
    clr(); header("GYRO DATA")
    for i, item in enumerate(GYRO_MENU):
        y = 18 + i * 14
        if i == gyro_sub_sel:
            oled.fill_rect(0, y - 1, W - 4, 11, 1)
            oled.text(">" + item, 2, y, 0)
        else:
            oled.text(" " + item, 2, y)
    footer("SEL=enter LT=back")
    show()

# tx on/off indicator with icon, speed + direction readout
def page_tx_monitor():
    clr(); header("TX MONITOR")
    # draw a nested-square icon, solid center when active
    ix, iy = 4, 15
    oled.rect(ix,     iy,     20, 20, 1)
    oled.rect(ix + 4, iy + 4, 12, 12, 1)
    if tx_enabled:
        oled.fill_rect(ix + 7, iy + 7, 6, 6, 1)
    status = "ON " if tx_enabled else "OFF"
    oled.text("TX",    30, 17)
    oled.text(status,  30, 27)
    # current motor speed + direction
    spd = abs(motor_speed)
    dr  = "FWD" if motor_speed >= 0 else "REV"
    oled.text("Spd:{:3d}%  {}".format(spd, dr), 0, 46)
    footer("SEL=toggle LT=back")
    show()

# page dispatch tables: main menu idx -> subpage function
GYRO_SUBPAGES = [page_gyro3d, page_gyro]
SUBPAGES = [page_motor, page_values, page_env, page_camera, page_distance, page_gyro_menu, page_tx_monitor, None]

# --- snake game ---

# pick a random cell not occupied by the snake
def _rand_food(snake, cols, rows):
    while True:
        x = random.randint(0, cols - 1)
        y = random.randint(0, rows - 1)
        if (x, y) not in snake:
            return (x, y)

# full snake game loop, exits on death or button press
async def snake_game():
    global in_sub
    CELL = 4                       # pixel size per grid cell
    COLS = W // CELL               # 32 columns
    ROWS = H // CELL               # 16 rows

    # start in center, 3 cells long, moving right
    snake = [(COLS//2, ROWS//2), (COLS//2-1, ROWS//2), (COLS//2-2, ROWS//2)]
    direction = (1, 0)
    food = _rand_food(snake, COLS, ROWS)
    score = 0
    speed_ms = 200                 # initial step interval
    _last_move = time.ticks_ms()

    # render snake + food + score overlay
    def draw():
        clr()
        for (x, y) in snake:
            oled.fill_rect(x*CELL, y*CELL, CELL-1, CELL-1, 1)
        oled.fill_rect(food[0]*CELL, food[1]*CELL, CELL-1, CELL-1, 1)
        oled.text(str(score), 0, 0)
        show()

    draw()

    while True:
        # direction change, block 180 reversals
        if pressed('up')  and direction != (0,  1): direction = (0, -1)
        if pressed('dn')  and direction != (0, -1): direction = (0,  1)
        if pressed('lt')  and direction != (1,  0): direction = (-1, 0)
        if pressed('rt')  and direction != (-1, 0): direction = (1,  0)
        if pressed('stop') or pressed('sel'):
            break

        now = time.ticks_ms()
        if time.ticks_diff(now, _last_move) >= speed_ms:
            _last_move = now
            hx, hy = snake[0]
            nx, ny = hx + direction[0], hy + direction[1]

            # wall collision = death
            if nx < 0 or nx >= COLS or ny < 0 or ny >= ROWS:
                break
            # self collision = death
            if (nx, ny) in snake:
                break

            snake.insert(0, (nx, ny))
            if (nx, ny) == food:
                score += 1
                food = _rand_food(snake, COLS, ROWS)
                speed_ms = max(60, speed_ms - 8)  # ramp speed each food eaten
            else:
                snake.pop()
            draw()

        await asyncio.sleep_ms(20)

    # game over screen, wait for exit press
    clr()
    oled.text("GAME OVER", 24, 18)
    oled.text("Score: " + str(score), 28, 32)
    oled.text("SEL=back", 32, 46)
    show()
    while not pressed('sel') and not pressed('stop'):
        await asyncio.sleep_ms(50)
    in_sub = False
    draw_menu()

# --- animations ---

# bouncing dot positions for power-on animation
_BOUNCE = [0, -2, -4, -5, -4, -2, 0, 0]

# power on: 3 bouncing dots + "turning on" text
async def anim_on():
    for frame in range(24):
        clr()
        oled.text("Turning on", 24, 18)
        for d in range(3):
            phase = (frame + d * 3) % len(_BOUNCE)
            y = 36 + _BOUNCE[phase]
            oled.fill_rect(45 + d * 16, y, 6, 6, 1)
        show()
        await asyncio.sleep_ms(70)
    flush_btns()
    draw_menu()

# power off: credits + downward black wipe
async def anim_off():
    clr()
    oled.text("Thank you.", 24, 12)
    oled.text("Team 305",  32, 27)
    oled.text("Christo",   36, 42)
    show()
    await asyncio.sleep_ms(2000)
    # wipe: black rectangle grows downward
    for i in range(0, H + 8, 4):
        clr()
        oled.fill_rect(0, H - i, W, i, 1)
        show()
        await asyncio.sleep_ms(40)
    clr(); show()
    flush_btns()

# --- debug overlay ---

# compact live readout of all sensors + last uart packet
def draw_debug():
    clr(); header("DEBUG")
    t = "{:.1f}".format(values["temp"])  if values["temp"]  != "--" else "--"
    h = "{:.1f}".format(values["humid"]) if values["humid"] != "--" else "--"
    d = "{:.1f}".format(values["distance"]) if values["distance"] != "--" else "--"
    oled.text("T:{} H:{}".format(t, h), 2, 13)
    oled.text("D:{} Cam:{}".format(d, values["camera_state"]), 2, 23)
    oled.text("GX:{} GY:{} GZ:{}".format(values["gyro_x"], values["gyro_y"], values["gyro_z"]), 0, 33)
    oled.text(("RX:" + last_rx_raw)[:18], 0, 43)
    footer("DBG again=exit")
    show()

# turn on debug led + overlay, print all values to serial
def enter_debug():
    global debug_mode
    debug_mode = True
    led_debug.value(1)
    draw_debug()
    print("DBG T={} H={} D={} M={} GX={} GY={} GZ={}".format(
        values["temp"], values["humid"], values["distance"], values["motor"],
        values["gyro_x"], values["gyro_y"], values["gyro_z"]))

# turn off debug, return to previous screen
def exit_debug():
    global debug_mode
    debug_mode = False
    led_debug.value(0)
    if lcd_on:
        if not in_sub:
            draw_menu()
        else:
            SUBPAGES[sel]()

# --- uart protocol ---

_last_motor_send = 0   # rate limit timestamp for motor commands

# dispatch incoming uart messages by type, parse payload, update values
def _handle_msg(msg_type, data, sender=0):
    global _rot_x, _rot_y, _rot_z
    print('[HANDLE] type=0x{:02X} sender=0x{:02X} len={}'.format(msg_type, sender, len(data)))

    # type 0x08: temperature from isaiah, int16 x100 LE, data[0]=type hi byte
    if msg_type == 8:
        if len(data) >= 4:
            values["temp"] = struct.unpack('<h', data[1:3])[0] / 100.0

    # type 0x0B: humidity from isaiah, uint16 x100 LE
    elif msg_type == 11:
        if len(data) >= 4:
            values["humid"] = struct.unpack('<H', data[1:3])[0] / 100.0

    # type 0x0A: pressure from isaiah, forwarded only
    elif msg_type == 10:
        pass

    # type 0x04: multi-source, check sender to distinguish gyro vs camera
    elif msg_type == 4:
        # gyro from ragul: 3x int16 LE = x, y, z degrees
        if sender == RAGUL_ID:
            if len(data) >= 6:
                gx, gy, gz = struct.unpack('<hhh', data[0:6])
                values["gyro_x"] = gx
                values["gyro_y"] = gy
                values["gyro_z"] = gz
                _rot_x = gx * 0.01745   # deg -> rad for 3d cube
                _rot_y = gy * 0.01745
                _rot_z = gz * 0.01745
                print('[GYRO] x={} y={} z={}'.format(gx, gy, gz))
        # camera from arianna: u8 state at data[1], u8 error at data[6]
        elif sender == ARIANNA_ID:
            if len(data) >= 7:
                values["camera_state"] = data[1]
                values["camera_error"] = data[6]

    # type 0x03: camera frame data, forwarded only
    elif msg_type == 3:
        pass

    # type 0x0C: distance from myles, float LE cm, data[0]=type hi byte
    elif msg_type == 12:
        if len(data) >= 5:
            values["distance"] = struct.unpack('<f', data[1:5])[0] * 100.0

    # live-update whichever subpage is showing
    if lcd_on and in_sub:
        # temp or humidity arrived, refresh values/env page
        if msg_type in (8, 11) and sel in (1, 2):
            page_values() if sel == 1 else page_env()
        # camera state arrived, refresh values/camera page
        elif msg_type == 4 and sender == ARIANNA_ID and sel in (1, 3):
            page_values() if sel == 1 else page_camera()
        # gyro arrived, refresh values or gyro subpage
        elif msg_type == 4 and sender == RAGUL_ID and sel in (1, 5):
            if sel == 1:
                page_values()
            elif in_gyro_sub:
                GYRO_SUBPAGES[gyro_sub_sel]()
            elif sel == 5:
                page_gyro_menu()
        # distance arrived, refresh values/distance page
        elif msg_type == 12 and sel in (1, 4):
            page_values() if sel == 1 else page_distance()

# send motor speed to liam, rate limited, skip zero, back off if rx pending
def send_motor_speed():
    global _last_motor_send
    if not tx_enabled:
        return
    now = time.ticks_ms()
    if time.ticks_diff(now, _last_motor_send) < 500:
        return  # max 1 send per 500ms
    if uart.any():
        return  # rx bytes waiting, don't collide
    _last_motor_send = now
    spd = max(-100, min(100, motor_speed))
    if spd == 0:
        return  # skip zero-speed, motor is off
    pkt = build_packet(LIAM_ID, 0x01, bytes([spd & 0xFF]))
    if pkt is None:
        return
    print('[TX] 43 -> {:02X} type=0x01 spd={}'.format(LIAM_ID, spd))
    led_tx.value(1)
    uart.write(pkt)
    led_tx.value(0)

# forward a packet out the uart, only when tx enabled
def _forward(pkt):
    if not tx_enabled:
        return
    print('[FWD] {} -> {}'.format('{:02X}'.format(pkt[2]), '{:02X}'.format(pkt[3])))
    uart.write(pkt)

# send ack for a received message, skip ack-of-ack to avoid loops
def _do_send_ack(dest, msg_type):
    if not tx_enabled:
        return
    if msg_type == 0xAA:
        return
    pkt = build_ack(dest, msg_type)
    print('[ACK] 43 -> {:02X} type=0xAA for=0x{:02X}'.format(dest, msg_type))
    uart.write(pkt)

# background task: feed uart bytes into receiver, dispatch packets, poll motor send
async def uart_task():
    global last_rx_raw
    while True:
        for raw_pkt in uart_rx.feed(uart):
            last_rx_raw = ' '.join('{:02X}'.format(x) for x in raw_pkt[:8])
            sender   = raw_pkt[2]
            receiver = raw_pkt[3]
            msg_type = raw_pkt[4]
            print('[RX] {} -> {} type=0x{:02X} | {}'.format(
                '{:02X}'.format(sender), '{:02X}'.format(receiver),
                msg_type, last_rx_raw))
            led_rx.value(1)
            route_packet(
                raw_pkt,
                handle_fn=_handle_msg,
                send_ack_fn=lambda dest, mt: _do_send_ack(dest, mt),
                forward_fn=_forward
            )
            led_rx.value(0)
        send_motor_speed()
        await asyncio.sleep_ms(5)

# --- main ui loop ---

# handles all button input, power toggle, menu nav, subpage controls
async def hmi_loop():
    global lcd_on, _lcd_changed_at, tx_enabled, in_sub, in_gyro_sub, gyro_sub_sel, sel, motor_speed, motor_dir, debug_mode
    _last_refresh = 0
    clr(); show()

    while True:
        # debug toggle, only when screen is on
        if lcd_on and pressed('dbg'):
            enter_debug() if not debug_mode else exit_debug()

        # debug mode: polling refresh, skip normal ui
        if debug_mode:
            for k in _prev: _prev[k] = _BTNS[k].value()
            draw_debug()
            await asyncio.sleep_ms(100)
            continue

        # power toggle with 1s debounce
        if pressed('start'):
            if not lcd_on:
                lcd_on = True; in_sub = False
                _lcd_changed_at = time.ticks_ms()
                await anim_on()
            elif time.ticks_diff(time.ticks_ms(), _lcd_changed_at) > 1000:
                lcd_on = False; in_sub = False
                await anim_off()

        if lcd_on:
            # main menu: scroll + select subpage or launch game
            if not in_sub:
                if held('up'):  sel = (sel - 1) % len(MENU); draw_menu()
                if held('dn'):  sel = (sel + 1) % len(MENU); draw_menu()
                if pressed('sel'):
                    if sel == 7:
                        await snake_game()
                    else:
                        in_sub = True; SUBPAGES[sel]()
            # tx monitor: toggle tx on/off, left returns to menu
            elif sel == 6:
                if pressed('sel'):
                    tx_enabled = not tx_enabled
                    page_tx_monitor()
                if pressed('lt'):
                    in_sub = False; draw_menu()
            # gyro sub-menu: two levels, picker + subpage
            elif sel == 5:
                if in_gyro_sub:
                    # inside gyro subpage, sel goes back to picker
                    if pressed('sel'):
                        in_gyro_sub = False
                        page_gyro_menu()
                    # raw values page auto-refreshes every 500ms
                    now = time.ticks_ms()
                    if gyro_sub_sel == 1 and time.ticks_diff(now, _last_refresh) > 500:
                        _last_refresh = now; page_gyro()
                else:
                    # gyro picker: scroll, select subpage, left returns
                    if pressed('lt'):
                        in_sub = False; draw_menu()
                    if held('up'):
                        gyro_sub_sel = (gyro_sub_sel - 1) % len(GYRO_MENU)
                        page_gyro_menu()
                    if held('dn'):
                        gyro_sub_sel = (gyro_sub_sel + 1) % len(GYRO_MENU)
                        page_gyro_menu()
                    if pressed('sel'):
                        in_gyro_sub = True
                        GYRO_SUBPAGES[gyro_sub_sel]()
            else:
                # generic subpage: sel returns to menu
                if pressed('sel'): in_sub = False; draw_menu()
                # motor page: held up/dn ramp speed, l/r flip direction
                if sel == 0:
                    changed = False
                    if held('up'):
                        if motor_dir == "FWD":
                            motor_speed = min(100, motor_speed + 5)
                        else:
                            motor_speed = max(-100, motor_speed - 5)
                        values["speed"] = "{}%".format(abs(motor_speed)); changed = True
                    if held('dn'):
                        if motor_dir == "FWD":
                            motor_speed = max(0, motor_speed - 5)
                        else:
                            motor_speed = min(0, motor_speed + 5)
                        values["speed"] = "{}%".format(abs(motor_speed)); changed = True
                    if pressed('lt'):
                        motor_dir = "REV"
                        motor_speed = -abs(motor_speed)
                        changed = True
                    if pressed('rt'):
                        motor_dir = "FWD"
                        motor_speed = abs(motor_speed)
                        changed = True
                    if changed: page_motor()

        await asyncio.sleep_ms(20)

# launch both async tasks: ui loop + uart handler
async def main():
    asyncio.create_task(hmi_loop())
    asyncio.create_task(uart_task())
    while True:
        await asyncio.sleep(60)

# run forever, clean up event loop on exit so restart works
try:
    asyncio.run(main())
finally:
    asyncio.new_event_loop()
```

---

## Download

The full HMI subsystem codebase (all files) can be found here:

[Download HMI Subsystem Code (ZIP)](./Christo_HMI_Final.zip)

MicroPython v1.23.0 on ESP32-S3-WROOM-1-N4.
