---
title: Microcontroller Code
---

## Overview
This is the code for the Rotary Encoder Subsystem. The Code performs the main operations of the rotary encoder that includes the increment count, the decrement count and also the reset. 

## Code
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

## Download the Zip File

The Zip file for the program can be found [*here*](Rotary_Encoder_Ky040.zip)