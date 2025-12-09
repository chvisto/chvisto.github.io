---
title: Microcontroller Code
---

## Overview
I had two separate codes for my demonstrations.
For the team demonstration, the rotary encoder rotated with the motor and treated each count as 4 degrees. When the encoder reached 90 degrees, it signaled the motor to stop since the door was fully open. Likewise, when it returned to 0 degrees, it notified the motor that the door was fully closed and could not rotate any further. Throughout the entire process, it continuously sent analog values to both the Flex and Proximity subsystems so they could track the door’s exact position at all times.

For the individual demonstration, I had to correct the printed output for each step because the encoder readings were not accurate. Using interrupts resolved this issue, and the updated code successfully demonstrated the increment, decrement, and reset functions of the rotary encoder.

## Team code (Calibration/Normal Rotation)
```c
#include "mcc_generated_files/system/system.h"
#include <xc.h>
#include <stdio.h>
#include <stdint.h>
#include <stdbool.h>

#ifndef _XTAL_FREQ
#define _XTAL_FREQ 64000000UL
#endif

// ---------------- GLOBALS ----------------

// Quadrature encoder count
static int16_t encoderCount = 0;

// Encoder resolution
static const int16_t COUNTS_PER_REV   = 80;   // 20 detents x 4 edges
static const int16_t MAX_ANGLE_DEG    = 90;   // door range 0 to 90
static const int16_t COUNTS_AT_90_DEG = (COUNTS_PER_REV * MAX_ANGLE_DEG) / 360; // 80*90/360 = 20

// System state flags
static uint8_t calibMode      = 0;   // 1 = in calibration mode
static uint8_t calibReached90 = 0;   // 1 = hit 90, going back to 0
static uint8_t hasCalibrated  = 0;   // 1 = calibration completed

// Extra modes
static uint8_t runMode        = 0;   // 1 = run mode
static uint8_t detailedMode   = 0;   // 1 = detailed encoder explanation

// Previous encoder states
static uint8_t rb0Prev = 1;          // Channel A (RB0) previous
static uint8_t rb3Prev = 1;          // Channel B (RB3) previous

// Buttons previous states
static uint8_t rd7Prev = 1;          // RD7 mode button
static uint8_t ra5Prev = 1;          // RA5 encoder push reset
static uint8_t sw0Prev = 1;          // RB4 SW0 reset

// Long press tracking for RD7
static uint16_t rd7HoldTicks   = 0;
static uint8_t  rd7LongHandled = 0;

// Tap detection on RD7
static uint8_t  tapCount       = 0;
static uint16_t tapWindowTicks = 0;

// Limit print tracking
static uint8_t at90Printed = 0;      // for calibration 90
static uint8_t at0Printed  = 0;      // for calibration 0

// Timing
static const uint8_t  LOOP_DELAY_MS     = 5;
static const uint16_t LONG_PRESS_MS     = 1000;
static const uint16_t LONG_PRESS_TICKS  = LONG_PRESS_MS / LOOP_DELAY_MS;
static const uint16_t TAP_WINDOW_MS     = 1000;
static const uint16_t TAP_WINDOW_TICKS  = TAP_WINDOW_MS / LOOP_DELAY_MS;

// ---------------- UART HELPER ----------------
static void UART1_WriteText(const char *txt)
{
    while (*txt)
    {
        while (!UART1_IsTxReady()) {}
        UART1_Write(*txt++);
    }
}

// Motor permission on RB2
static inline uint8_t PermSignal(void)
{
    return IO_RB2_GetValue();   // 1 = motor subsystem allows movement
}

void main(void)
{
    SYSTEM_Initialize();

    // -------- INPUT CONFIG --------

    // Encoder Channel B = RB3
    IO_RB3_SetDigitalMode();
    IO_RB3_SetDigitalInput();
    IO_RB3_SetPullup();

    // Encoder Channel A = RB0
    IO_RB0_SetDigitalMode();
    IO_RB0_SetDigitalInput();
    IO_RB0_SetPullup();

    // RD7 mode button
    IO_RD7_SetDigitalMode();
    IO_RD7_SetDigitalInput();
    IO_RD7_SetPullup();

    // RA5 encoder push reset
    IO_RA5_SetDigitalMode();
    IO_RA5_SetDigitalInput();
    IO_RA5_SetPullup();

    // RB4 SW0 reset (onboard button)
    IO_RB4_SetDigitalMode();
    IO_RB4_SetDigitalInput();
    IO_RB4_SetPullup();

    // RB2 motor permission
    IO_RB2_SetDigitalMode();
    IO_RB2_SetDigitalInput();
    IO_RB2_SetPullup();

    // -------- OUTPUT CONFIG --------

    // RC3 LED for 0/90 in system modes
    IO_RC3_SetDigitalMode();
    IO_RC3_SetDigitalOutput();
    IO_RC3_SetLow();

    // RB1: motor open line
    IO_RB1_SetDigitalMode();
    IO_RB1_SetDigitalOutput();
    IO_RB1_SetLow();

    // RC4: motor close / flag
    IO_RC4_SetDigitalMode();
    IO_RC4_SetDigitalOutput();
    IO_RC4_SetLow();

    // RF4: mirror of RC4
    IO_RF4_SetDigitalMode();
    IO_RF4_SetDigitalOutput();
    IO_RF4_SetLow();

    // RC2: calibration line
    IO_RC2_SetDigitalMode();
    IO_RC2_SetDigitalOutput();
    IO_RC2_SetLow();

    // RA2 DAC configured in MCC

    // Init previous values
    rb3Prev = IO_RB3_GetValue();
    rb0Prev = IO_RB0_GetValue();
    rd7Prev = IO_RD7_GetValue();
    ra5Prev = IO_RA5_GetValue();
    sw0Prev = IO_RB4_GetValue();

    char buf[200];

    while (1)
    {
        // per-iteration event flags
        uint8_t hit90Event    = 0;
        uint8_t endCalibEvent = 0;

        // ---------- READ BUTTONS ----------
        uint8_t rd7Now = IO_RD7_GetValue();
        uint8_t ra5Now = IO_RA5_GetValue();
        uint8_t sw0Now = IO_RB4_GetValue();
        uint8_t perm   = PermSignal();

        // ----- RA5: POSITION RESET ANYTIME -----
        if (ra5Prev && !ra5Now)   // falling edge
        {
            encoderCount   = 0;
            calibReached90 = 0;
            at90Printed    = 0;
            at0Printed     = 0;

            UART1_WriteText("RA5: reset to 0\r\n");
        }
        ra5Prev = ra5Now;

        // ----- SW0 (RB4): FULL SYSTEM RESET -----
        if (sw0Prev && !sw0Now)   // falling edge
        {
            // turn everything off
            IO_RB1_SetLow();
            IO_RC4_SetLow();
            IO_RF4_SetLow();
            IO_RC2_SetLow();
            IO_RC3_SetLow();

            // reset all state
            encoderCount   = 0;
            calibMode      = 0;
            runMode        = 0;
            detailedMode   = 0;
            calibReached90 = 0;
            hasCalibrated  = 0;

            at90Printed    = 0;
            at0Printed     = 0;

            tapCount       = 0;
            tapWindowTicks = 0;
            rd7HoldTicks   = 0;
            rd7LongHandled = 0;

            // zero DAC
            DAC1_SetOutput(0);

            UART1_WriteText("SW0: full reset\r\n");
        }
        sw0Prev = sw0Now;

        // ----- RD7: TAP + LONG PRESS -----
        if (!rd7Now)
        {
            rd7HoldTicks++;

            // long press -> exit any mode
            if ((calibMode || runMode || detailedMode) &&
                !rd7LongHandled &&
                rd7HoldTicks >= LONG_PRESS_TICKS)
            {
                calibMode    = 0;
                runMode      = 0;
                detailedMode = 0;

                tapCount       = 0;
                tapWindowTicks = 0;

                IO_RB1_SetLow();
                IO_RC4_SetLow();
                IO_RF4_SetLow();
                IO_RC2_SetLow();
                IO_RC3_SetLow();

                rd7LongHandled = 1;
                UART1_WriteText("Mode: OFF\r\n");
            }
        }
        else
        {
            if (!rd7Prev)
            {
                // short tap if released before long-press duration
                if (rd7HoldTicks < LONG_PRESS_TICKS)
                {
                    if (tapCount == 0)
                    {
                        tapCount       = 1;
                        tapWindowTicks = 0;
                    }
                    else
                    {
                        tapCount++;
                    }
                }
            }

            rd7HoldTicks   = 0;
            rd7LongHandled = 0;
        }

        rd7Prev = rd7Now;

        // ----- TAP WINDOW DECISION -----
        if (tapCount > 0)
        {
            tapWindowTicks++;

            if (tapWindowTicks >= TAP_WINDOW_TICKS)
            {
                // special handling: if we are already in RUN mode and see a double tap, toggle OFF
                if (runMode && (tapCount == 2))
                {
                    runMode = 0;

                    IO_RB1_SetLow();
                    IO_RC4_SetLow();
                    IO_RF4_SetLow();
                    IO_RC2_SetLow();
                    IO_RC3_SetLow();

                    UART1_WriteText("Mode: OFF\r\n");
                }
                else
                {
                    // normal global behavior
                    if (tapCount >= 3)
                    {
                        // triple tap -> DETAIL mode
                        calibMode    = 0;
                        runMode      = 0;
                        detailedMode = 1;

                        at90Printed = 0;
                        at0Printed  = 0;

                        IO_RB1_SetLow();
                        IO_RC4_SetLow();
                        IO_RF4_SetLow();
                        IO_RC2_SetLow();
                        IO_RC3_SetLow();

                        UART1_WriteText("Mode: DETAIL\r\n");
                    }
                    else if (tapCount == 2)
                    {
                        // double tap -> RUN mode (if calibrated)
                        if (!hasCalibrated)
                        {
                            UART1_WriteText("Cannot start: not calibrated\r\n");
                        }
                        else
                        {
                            calibMode    = 0;
                            detailedMode = 0;
                            runMode      = 1;

                            at90Printed = 0;
                            at0Printed  = 0;

                            IO_RB1_SetLow();
                            IO_RC4_SetLow();
                            IO_RF4_SetLow();
                            IO_RC2_SetLow();

                            UART1_WriteText("Mode: RUN\r\n");
                            UART1_WriteText("Waiting for motor to rotate\r\n");
                        }
                    }
                    else if (tapCount == 1)
                    {
                        // single tap -> CALIBRATION mode
                        calibMode      = 1;
                        runMode        = 0;
                        detailedMode   = 0;
                        calibReached90 = 0;
                        at90Printed    = 0;
                        at0Printed     = 0;

                        UART1_WriteText("Mode: CALIBRATION\r\n");
                        UART1_WriteText("Motor: opening to 90\r\n");
                    }
                }

                tapCount       = 0;
                tapWindowTicks = 0;
            }
        }

        // ---------- READ ENCODER CHANNELS ----------
        uint8_t aNow = IO_RB0_GetValue();
        uint8_t bNow = IO_RB3_GetValue();

        uint8_t movementHappened = 0;
        int8_t  lastDelta        = 0;

        uint8_t prevState = (rb0Prev << 1) | rb3Prev;
        uint8_t currState = (aNow   << 1) | bNow;

        if (currState != prevState)
        {
            int8_t delta = 0;

            switch (prevState)
            {
                case 0b00:
                    if (currState == 0b01)      delta = +1;
                    else if (currState == 0b10) delta = -1;
                    break;
                case 0b01:
                    if (currState == 0b11)      delta = +1;
                    else if (currState == 0b00) delta = -1;
                    break;
                case 0b11:
                    if (currState == 0b10)      delta = +1;
                    else if (currState == 0b01) delta = -1;
                    break;
                case 0b10:
                    if (currState == 0b00)      delta = +1;
                    else if (currState == 0b11) delta = -1;
                    break;
                default:
                    break;
            }

            uint8_t canMove = 0;

            if (calibMode || detailedMode)
            {
                canMove = 1;
            }
            else if (runMode && hasCalibrated && perm)
            {
                canMove = 1;
            }

            if (delta != 0 && canMove)
            {
                encoderCount += delta;
                movementHappened = 1;
                lastDelta        = delta;
            }
        }

        rb0Prev = aNow;
        rb3Prev = bNow;

        // ---------- ANGLES ----------
        int16_t angleRawDeg = (int16_t)(((int32_t)encoderCount * 360) / COUNTS_PER_REV);

        int16_t angleWrappedDeg = angleRawDeg % 360;
        if (angleWrappedDeg < 0)
        {
            angleWrappedDeg += 360;
        }

        int16_t angleDeg   = 0;  // system angle (0–90)
        uint8_t closedFlag = 0;
        uint8_t openFlag   = 0;
        uint8_t dacVal     = 0;

        // ---------- CALIBRATION MODE ----------
        if (calibMode)
        {
            angleDeg = angleRawDeg;

            if (angleDeg < 0)
            {
                angleDeg     = 0;
                encoderCount = 0;
            }
            if (angleDeg > MAX_ANGLE_DEG)
            {
                angleDeg     = MAX_ANGLE_DEG;
                encoderCount = COUNTS_AT_90_DEG;
            }

            closedFlag = (angleDeg == 0);
            openFlag   = (angleDeg == MAX_ANGLE_DEG);

            if (closedFlag || openFlag) IO_RC3_SetHigh();
            else                        IO_RC3_SetLow();

            IO_RC2_SetHigh();  // tell subsystems we’re calibrating

            // phase tracking
            if (!calibReached90 && openFlag)
            {
                calibReached90 = 1;
            }

            if (!calibReached90)
            {
                // 0 -> 90 : open direction
                IO_RB1_SetHigh();
                IO_RC4_SetLow();
                IO_RF4_SetLow();
            }
            else
            {
                if (!closedFlag)
                {
                    // 90 -> 0 : close direction
                    IO_RB1_SetLow();
                    IO_RC4_SetHigh();
                    IO_RF4_SetHigh();
                }
                else
                {
                    // reached 0 at end of calibration
                    IO_RB1_SetLow();
                    IO_RC4_SetLow();
                    IO_RF4_SetLow();

                    calibMode      = 0;
                    hasCalibrated  = 1;
                    calibReached90 = 0;

                    IO_RC2_SetLow();

                    endCalibEvent = 1;
                }
            }

            // detect first hit of 90
            if (openFlag && !at90Printed)
            {
                hit90Event  = 1;
                at90Printed = 1;
            }
            if (!openFlag)
            {
                at90Printed = 0;
            }

            dacVal = (uint8_t)(((uint32_t)angleDeg * 255u) / MAX_ANGLE_DEG);
        }
        // ---------- RUN MODE ----------
        else if (runMode)
        {
            angleDeg = angleRawDeg;

            if (angleDeg < 0)
            {
                angleDeg     = 0;
                encoderCount = 0;
            }
            if (angleDeg > MAX_ANGLE_DEG)
            {
                angleDeg     = MAX_ANGLE_DEG;
                encoderCount = COUNTS_AT_90_DEG;
            }

            closedFlag = (angleDeg == 0);
            openFlag   = (angleDeg == MAX_ANGLE_DEG);

            if (closedFlag || openFlag) IO_RC3_SetHigh();
            else                        IO_RC3_SetLow();

            IO_RB1_SetLow();
            IO_RC4_SetLow();
            IO_RF4_SetLow();
            IO_RC2_SetLow();

            dacVal = (uint8_t)(((uint32_t)angleDeg * 255u) / MAX_ANGLE_DEG);
        }
        // ---------- DETAILED MODE ----------
        else if (detailedMode)
        {
            IO_RC3_SetLow();
            IO_RB1_SetLow();
            IO_RC4_SetLow();
            IO_RF4_SetLow();
            IO_RC2_SetLow();

            dacVal = (uint8_t)(((uint32_t)angleWrappedDeg * 255u) / 360u);
        }
        else
        {
            IO_RC3_SetLow();
            IO_RB1_SetLow();
            IO_RC4_SetLow();
            IO_RF4_SetLow();
            IO_RC2_SetLow();
            dacVal = 0;
        }

        // ---------- DAC ----------
        DAC1_SetOutput(dacVal);

        // ---------- UART DEBUG ----------
        if (movementHappened)
        {
            // System-style print for CALIB/RUN or when we just ended calibration
            if (calibMode || runMode || endCalibEvent)
            {
                sprintf(buf, "cnt=%d angle=%d dac=%u\r\n",
                        encoderCount, angleDeg, dacVal);
                UART1_WriteText(buf);

                // extra messages in calibration mode
                if (hit90Event)
                {
                    UART1_WriteText("Limit 90: motor go back to 0\r\n");
                }
                if (endCalibEvent)
                {
                    UART1_WriteText("Limit 0: motor stop\r\n");
                    UART1_WriteText("Calibration complete\r\n");
                }

                // extra messages in RUN mode at 0 and 90
                if (runMode)
                {
                    if (openFlag)
                    {
                        UART1_WriteText("Run: 90 deg - motor stop\r\n");
                    }
                    if (closedFlag)
                    {
                        UART1_WriteText("Run: 0 deg - motor stop\r\n");
                    }
                }
            }
            else if (detailedMode)
            {
                const char *dirStr = (lastDelta > 0) ? "CW" : "CCW";

                sprintf(buf,
                        "DETAIL: cnt=%d delta=%d dir=%s rawAngle=%d wrapped=%d\r\n",
                        encoderCount,
                        lastDelta,
                        dirStr,
                        angleRawDeg,
                        angleWrappedDeg);
                UART1_WriteText(buf);
            }
        }

        __delay_ms(LOOP_DELAY_MS);
    }
}

```

## Individual Code (Increment/Decrement/Reset)
```c

#include "mcc_generated_files/system/system.h"
#include <xc.h>
#include <stdio.h>
#include <stdint.h>
#include <stdbool.h>

#ifndef _XTAL_FREQ
#define _XTAL_FREQ 64000000UL
#endif

// Encoder edges equaling one detent
#define EDGES_PER_STEP     2

// Logging window duration
#define WINDOW_MS          10000
#define WINDOW_TICK_MS     10

// Max number of logged step events
#define LOG_CAPACITY       100

// Structure for storing each step event
typedef struct
{
    int32_t count;
    int8_t  dir;
} StepEvent;

// Global encoder state
volatile int32_t stepCount      = 0;
volatile int32_t edgeCount      = 0;
volatile int8_t  edgeRemainder  = 0;

// Last known encoder state (A,B)
volatile uint8_t encPrevState   = 0x03;

// Step event log
volatile StepEvent stepLog[LOG_CAPACITY];
volatile uint8_t logIndex = 0;

// RA5 reset button tracking
volatile uint8_t ra5Prev = 1;

// Simple increment function
void Encoder_Increment(void) { stepCount++; }

// Simple decrement function
void Encoder_Decrement(void) { stepCount--; }

// UART output for printf
void putch(char data)
{
    while (!UART1_IsTxReady()) {}
    UART1_Write((uint8_t)data);
}

// This function processes the encoder quadrature state
static void Encoder_ProcessIO(void)
{
    // Read current encoder channels
    uint8_t a = IO_RB0_GetValue();
    uint8_t b = IO_RB3_GetValue();

    // Combine into 2-bit state
    uint8_t currState = (uint8_t)((a << 1) | b);
    uint8_t prevState = encPrevState;

    // If no change, do nothing
    if (currState == prevState) return;

    // Direction result
    int8_t delta = 0;

    // Quadrature decoding
    switch (prevState)
    {
        case 0b00:
            if (currState == 0b01) delta = +1;
            else if (currState == 0b10) delta = -1;
            break;

        case 0b01:
            if (currState == 0b11) delta = +1;
            else if (currState == 0b00) delta = -1;
            break;

        case 0b11:
            if (currState == 0b10) delta = +1;
            else if (currState == 0b01) delta = -1;
            break;

        case 0b10:
            if (currState == 0b00) delta = +1;
            else if (currState == 0b11) delta = -1;
            break;
    }

    // If valid movement detected
    if (delta != 0)
    {
        // Count edges
        edgeCount += delta;
        edgeRemainder += delta;

        // Convert edges to steps (detents)
        while (edgeRemainder >= EDGES_PER_STEP)
        {
            edgeRemainder -= EDGES_PER_STEP;
            Encoder_Increment();

            // Log step
            uint8_t i = logIndex;
            if (i < LOG_CAPACITY)
            {
                stepLog[i].count = stepCount;
                stepLog[i].dir   = +1;
                logIndex = i + 1;
            }
        }

        while (edgeRemainder <= -EDGES_PER_STEP)
        {
            edgeRemainder += EDGES_PER_STEP;
            Encoder_Decrement();

            // Log step
            uint8_t i = logIndex;
            if (i < LOG_CAPACITY)
            {
                stepLog[i].count = stepCount;
                stepLog[i].dir   = -1;
                logIndex = i + 1;
            }
        }
    }

    // Update previous state for next interrupt
    encPrevState = currState;
}

// RB0 interrupt handler
void Encoder_RB0_Handler(void)
{
    Encoder_ProcessIO();
}

// RB3 interrupt handler
void Encoder_RB3_Handler(void)
{
    Encoder_ProcessIO();
}

void main(void)
{
    SYSTEM_Initialize();

    // Encoder input pins
    IO_RB0_SetDigitalInput();
    IO_RB3_SetDigitalInput();
    IO_RB0_SetPullup();
    IO_RB3_SetPullup();

    // RA5 reset button
    IO_RA5_SetDigitalInput();
    IO_RA5_SetPullup();

    // Initialize the encoder starting state
    uint8_t a0 = IO_RB0_GetValue();
    uint8_t b0 = IO_RB3_GetValue();
    encPrevState = (uint8_t)((a0 << 1) | b0);

    // Attach interrupt handlers
    IO_RB0_SetInterruptHandler(Encoder_RB0_Handler);
    IO_RB3_SetInterruptHandler(Encoder_RB3_Handler);

    // Enable interrupts
    ei();

    while (1)
    {
        // Reset log for the new time window
        di();
        logIndex = 0;
        edgeRemainder = 0;
        ei();

        uint16_t elapsed = 0;

        // 10-second window
        while (elapsed < WINDOW_MS)
        {
            __delay_ms(WINDOW_TICK_MS);
            elapsed += WINDOW_TICK_MS;

            // RA5 reset button (falling edge)
            uint8_t ra5Now = IO_RA5_GetValue();
            if (ra5Prev && !ra5Now)
            {
                di();
                stepCount     = 0;
                edgeCount     = 0;
                edgeRemainder = 0;
                logIndex      = 0;
                ei();

                printf("\r\n--- RESET (RA5 pressed) ---\r\n");
            }
            ra5Prev = ra5Now;
        }

        // Number of logged step events
        uint8_t n;
        di();
        n = logIndex;
        ei();

        printf("\r\n----- NEW WINDOW (%u ms) -----\r\n", (unsigned)WINDOW_MS);

        // Print every logged step
        for (uint8_t i = 0; i < n; i++)
        {
            int32_t c;
            int8_t  d;

            di();
            c = stepLog[i].count;
            d = stepLog[i].dir;
            ei();

            const char* dirStr = (d > 0) ? "clockwise" : "counterclockwise";
            printf("count: %ld, direction: %s\r\n", (long)c, dirStr);
        }

        // If no movement occurred
        if (n == 0)
        {
            printf("no steps recorded\r\n");
        }
    }
}

```

## Download the Zip file of the project

The Zip file for the program can be found [*here.*](Rotary_Encoder_Ky040.zip)