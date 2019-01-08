---
layout: post
title: "Another Undocument Feature: STM32F0 SPI Interface MOSI Idle State"
categories: STM32
date: 2019-01-06
---

## MOSI State When SPI is Idle

   SPI is simple protocol with 4 signals(SS, SCLK, MOSI, and MISO).  With STM32 CubeMX utility, I can easily manipulate the timing of SPI 
   timing.  We can use SPI for generate control signals for some hardware, like LED with WS2812B([https://bitbucket.org/mtien888/stm32-for-ws2812b-using-spi/src/](https://bitbucket.org/mtien888/stm32-for-ws2812b-using-spi/src/).

## Why Do We Care MOSI Idle State? 

   It is not defined in SPI how the MOSI idle state should be.  It doesn't matter for SPI device when select signal(SS) is not enabled.  
   If you google around, some people say MOSI is high(maybe with pull-up resistor) when SPI is idle.  Others say it is the last bit sent by SPI.  
   I made a series tests to find out MOSI state when SPI is idle with STM32F072.

## WARNING: THE RESULT ONLY FOR STM32F072   

   For different brand and series MCU(F1, F3, F4 for example), the result maybe be different. 
   
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

| Test Data       | MOSI IDLE       |
|-----------------|-----------------|
| 00              | 0               |
| FF              | 1               |
| 01              | 0               |
| 80              | 1               |

   Looks like the MOSI Idle State is the first bit(MSB) send to SPI.  

 * Second Test (2 bytes)

| Test Data       | MOSI IDLE       |
|-----------------|-----------------|
| 00 00           | 0               |
| FF FF           | 1               |
| 01 01           | 0               |
| 80 00           | 1               |

   Looks like the MOSI Idle State is the first bit(MSB) send to SPI, just like previous test.

 * 3 Bytes Tests

| Test Data       | MOSI IDLE       |
|-----------------|-----------------|
| XX XX 00        | 0               |
| XX XX FF        | 1               |
| XX XX 01        | 0               |
| XX XX 80        | 1               |

   This is definitively not following previous conclusion.  The Idle state is the MSB of last byte.

 * 4 Bytes Tests

| Test Data       | MOSI IDLE       |
|-----------------|-----------------|
| 00 XX XX XX     | 0               |
| FF XX XX XX     | 1               |
| 01 XX XX XX     | 0               |
| 81 XX XX XX     | 1               |

  This is getting interesting.  The MOSI idle state is the MSB of first byte.

 * 5 Bytes Test

| Test Data       | MOSI IDLE       |
|-----------------|-----------------|
| XX 00 XX XX XX  | 0               |
| XX FF XX XX XX  | 1               |
| XX 01 XX XX XX  | 0               |
| XX 81 XX XX XX  | 1               |

 The MOSI idle state is the MSB of second byte.

 * 6 Bytes Test

| Test Data              | MOSI IDLE       |
|------------------------|-----------------|
| XX XX 00 XX XX XX      | 0               |
| XX XX FF XX XX XX      | 1               |
| XX XX 01 XX XX XX      | 0               |
| XX XX 81 XX XX XX      | 1               |

 * 7 Bytes Test

| Test Data                 | MOSI IDLE       |
|---------------------------|-----------------|
| XX XX XX 00 XX XX XX      | 0               |
| XX XX XX FF XX XX XX      | 1               |
| XX XX XX 01 XX XX XX      | 0               |
| XX XX XX 81 XX XX XX      | 1               |

 * 8~16 Bytes Test

| Test Data                 | MOSI IDLE       |
|---------------------------|-----------------|
| XX ~ XX XX 00 XX XX XX    | 0               |
| XX ~ XX XX FF XX XX XX    | 1               |
| XX ~ XX XX 01 XX XX XX    | 0               |
| XX ~ XX XX 81 XX XX XX    | 1               |

We finally see a pattern.  If we send N bytes(N>=4), MOSI IDLE is MSB of (N-3)th byte.

## Possible Explanation

Why the MSB of last 4 byes?  We can guess how the hardware works.  

* 32 bytes Shift Register

  SPI hardware has a 32 bits shift register to send MOSI.  SPI send data (to MOSI) from MSB to LSB.  After last byte sent, the hardware somehow load the MSB to MOSI. 

* One and Two Byte Transmission Exception

  The one and two byte transmit one load the data to MSB of shift register.  And MOSI load the MSB of shift register when transmission finsihed.

* 3 bytes transmission Exception
  The exception for 3 byte transmit might be done by first load two bytes and then one byte to transmit.  That might explain MOSI IDLE is the MSB of last byte.


## How to manipulation MOSI Idle state

According to previous finding, to setup MOSI IDLE is pretty easy.  At most send 4 more bytes, I can setup the MOSI IDLE to any state I want.  For some situation, I only need to tweak data a little bit without sending more bytes.


## **Waring Warning Warning**

This is just hack hardware for current project.  It might be not useful for SPI of different MCU.  But I am happy I can use it without cost me additional hardware. 



