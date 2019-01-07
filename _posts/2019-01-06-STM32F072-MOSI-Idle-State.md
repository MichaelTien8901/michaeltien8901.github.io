---
layout: post
title: "Another Undocument Feature: STM32F0 SPI Interface MOSI Idle State"
date: 2019-01-06
---

# SPI Interface MOSI Idle State

   SPI is simple protocol with 4 signals(SS, SCLK, MOSI, and MISO).  With STM32 CubeMX utility, I can easily manipulate the timing of SPI 
   timing.  One day, I think maybe we can use SPI for generate control signals for some hardware, like LED with WS2812B.

## Why Do We Care MOSI Idle State? 

   It is not defined in SPI how the MOSI idle state should be.  It doesn't matter for SPI device when select signal is not enabled.  
   If you google around, some people it is high(maybe with pull-up resistor).  Others say it is the last bit sent.  
   For me to use MOSI to generate control signal, the undefined state just unacceptable.  Then I make a series tests to find out 
   MOSI for STM32F072.  

## WARNING: THE RESULT ONLY FOR STM32F072   

   The test result is only for STM32F072.  For different brand and series MCU(F1, F3, F4 for example), the result maybe be different. 
   
## Tests
  * STM32F072 SPI Settings
   
   The following is the setting used for SPI settings

| Settings            | Value                |
|---------------------|----------------------|
| Mode                | Transmit Only Master |
| Hardware NSS Signal | Disable              |
| Frame Format        | MOTOROLA             |
| Data Size           | 8 bits               |
| First Bit           | MSB First            |
| CPOL                | High                 |
| CPHA                | 1 Edge               |
| CRC Calculation     | Disabled             |
| NSSP Mode           | Dislabled            |
| NSS Signal Type     | Software             |

  
  * First Test (1 byte)

| Test Data         | MOSI Idle State |
|-------------------|-----------------|
| 00                | 0               |
| FF                | 1               |
| 01                | 0               |
| 80                | 1               |

   Looks like the MOSI Idle State is the first bit send to SPI.  

 * Second Test (2 bytes)

| Test Data       | MOSI Idle State |
|-----------------|-----------------|
| 00 00           | 0               |
| FF FF           | 1               |
| 01 01           | 0               |
| 80 00           | 1               |

   Looks like the MOSI Idle State is the first bit send to SPI, just like previous test.

 * 3 Bytes Tests

| Test Data           | MOSI Idle State |
|---------------------|-----------------|
| XX XX 00            | 0               |
| XX XX FF            | 1               |
| XX XX 01            | 0               |
| XX XX 80            | 1               |

   This is definitively not following previous conclusion.  The Idle state is the last byte's first bit(MSB).

 * 4 Bytes Tests

| Test Data           | MOSI Idle State |
|---------------------|-----------------|
| 00 XX XX XX         | 0               |
| FF XX XX XX         | 1               |
| 01 XX XX XX         | 0               |
| 81 XX XX XX         | 1               |

  This is getting interesting.  The MOSI idle state is the first byte's first bit(MSB).

 * 5 Bytes Test

| Test Data           | MOSI Idle State |
|---------------------|-----------------|
| XX 00 XX XX XX      | 0               |
| XX FF XX XX XX      | 1               |
| XX 01 XX XX XX      | 0               |
| XX 81 XX XX XX      | 1               |

 The MOSI idle state is the last 4th byte's first bit(MSB).

 * 6 Bytes Test

| Test Data              | MOSI Idle State |
|------------------------|-----------------|
| XX XX 00 XX XX XX      | 0               |
| XX XX FF XX XX XX      | 1               |
| XX XX 01 XX XX XX      | 0               |
| XX XX 81 XX XX XX      | 1               |

 * 7 Bytes Test

| Test Data                 | MOSI Idle State |
|---------------------------|-----------------|
| XX XX XX 00 XX XX XX      | 0               |
| XX XX XX FF XX XX XX      | 1               |
| XX XX XX 01 XX XX XX      | 0               |
| XX XX XX 81 XX XX XX      | 1               |

 * 8~16 Bytes Test

| Test Data                 | MOSI Idle State |
|---------------------------|-----------------|
| XX ~ XX XX 00 XX XX XX    | 0               |
| XX ~ XX XX FF XX XX XX    | 1               |
| XX ~ XX XX 01 XX XX XX    | 0               |
| XX ~ XX XX 81 XX XX XX    | 1               |

We finally see a pattern.  MOSI idle state is last 4th byte's MSB.

## Possible Explanation

Why the last 4th bytes MSB?  We can guess how the hardware works.  

### 32 bytes Shift Register

SPI hardware has a 32 bits shift register to send MOSI.  SPI send data (to MOSI) from MSB to LSB.  After last byte sent, the hardware somehow load the MSB to MOSI. 

### One and Two Byte Transmit Exception

The one and two byte transmit one load the data to MSB of shift register.  And MOSI load the MSB of shift register when transmit finsihed.

### 3 bytes transmit Exception
   The exception for 3 byte transmit might be done by first load two bytes and then one byte to transmit.  That explain MOSI idle state is
   the last byte's MSB.


## How to manipulation MOSI Idle state

According to previous finding, to setup MOSI idle state is pretty easy.  At most send 4 more bytes, I can setup the MOSI idle state to any 
state I want.  Sometimes, I can only tweak data a little bit without sending more data.  


## Waring Warning Warning

This is only hack the current hardware I used for current project.  It might be not useful for SPI of different MCU.  But I am happy I can 
use it for my current project without cost me another hardware.  



