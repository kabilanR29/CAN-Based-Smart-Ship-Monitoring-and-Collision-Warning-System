# CAN-Based-Smart-Ship-Monitoring-and-Collision-Warning-System:-

A multi-ECU embedded system built on the **LPC2129 (ARM7TDMI)** microcontroller that simulates a ship's onboard monitoring network using the **CAN bus protocol**. Three independent ECUs communicate over a shared CAN network to provide real-time radar-based obstacle detection, engine-room safety monitoring, and centralized status display for the captain — mirroring how modern marine and automotive systems use distributed ECUs instead of a single monolithic controller.

## Overview

Ships rely on multiple independent sub-systems (navigation, safety, control) that must communicate reliably even if one module fails or is added/removed from the network. This project demonstrates that architecture at a hardware level using three separate LPC2129 boards networked over CAN, each responsible for a distinct function.

## System Architecture

| ECU | Role | Sensors / Peripherals |
|---|---|---|
| **ECU-1** | Marine Radar | HC-SR04 ultrasonic distance sensor, servo motor (sweep), MCP2551 CAN transceiver |
| **ECU-2** | Ship Safety Monitoring | LM35 temperature sensor (ADC), gas sensor, flame sensor, MCP2551 CAN transceiver |
| **ECU-3** | Captain Control / Display | CAN receiver, 20×4 LCD, UART-to-PC (MAX232), MCP2551 CAN transceiver |

All three ECUs communicate over a single CAN bus (125 Kbps @ PCLK = 60 MHz), with each node identified by unique CAN message IDs.

## CAN Message IDs

| CAN ID | Source | Data |

| `0x102` | ECU-1 | Obstacle distance (cm) |
| `0x103` | ECU-1 | Collision status (0 = Safe, 1 = Warning, 2 = Danger) |
| `0x201` | ECU-2 | Engine room temperature (°C) |
| `0x203` | ECU-2 | Gas leak detection (0 = Normal, 1 = Detected) |
| `0x204` | ECU-2 | Flame detection (0 = Normal, 1 = Detected) |

## How It Works

1. **ECU-1** continuously measures distance to nearby objects using the HC-SR04 sensor while a servo motor sweeps the sensor across a fixed range. Each reading is classified into a Safe / Warning / Danger zone based on distance thresholds and transmitted over CAN.
2. **ECU-2** monitors engine-room conditions — reading temperature via the onboard ADC and checking digital gas/flame sensor outputs — and transmits each value independently over CAN.
3. **ECU-3** listens to all CAN traffic from both ECUs, decodes each message by ID, and displays live readings on a 20×4 LCD (distance/status on the top half, temperature/gas/flame on the bottom half) while also streaming the same data to a PC over UART for logging or visualization.

## Collision / Safety Thresholds

| Zone | Condition

| Safe | Distance > 150 cm |
| Warning | 50 cm – 150 cm |
| Danger | < 50 cm |

## Hardware Used

- 3 × LPC2129 (ARM7TDMI) development boards
- HC-SR04 ultrasonic sensor
- SG90 / MG995 servo motor
- LM35 temperature sensor
- MQ-series gas sensor (digital output)
- IR flame sensor (digital output)
- MCP2551 CAN transceivers (×3)
- 20×4 character LCD
- MAX232 (RS232 level converter for PC logging)
- 120Ω termination resistors at both physical ends of the CAN bus

## Firmware Details

- Written in **register-level embedded C** (no HAL/RTOS) using Keil µVision
- Timer0/Timer1-based delay and pulse-width measurement routines
- CAN2 peripheral configured for 125 Kbps operation with `VPBDIV = 1` (PCLK = 60 MHz), a fixed clock constraint shared across all timing-dependent modules to keep sensor timing, PWM, and CAN bit-rate consistent
- Each ECU runs as a single standalone `main.c`, making it a self-contained flashable image per board

## Repository Structure

```
ECU1_main.c         - Radar ECU (HC-SR04 + servo + CAN Tx)
ECU2_main.c          - Safety ECU (LM35 + gas + flame + CAN Tx)
CAN_RXmain.c          - Captain Control ECU (CAN Rx + LCD + UART)
CANTrans_driver.c    - CAN2 transmit driver (shared by ECU-1 & ECU-2)
CAN_RXdriver.c        - CAN2 receive driver (ECU-3)
CAN_RXUARTdriver.c   - UART0 driver for PC logging (ECU-3)
CAN_header.h          - Shared CAN message struct & type definitions
lcdheader.h           - 20x4 LCD driver (4-bit mode)
delay_header.h        - Timer-based delay utilities
```

## Future Scope

- GSM module integration on ECU-3 for emergency SMS alerts
- Buzzer-based audible alarm for Danger-level events
- PC-side GUI for real-time radar visualization
