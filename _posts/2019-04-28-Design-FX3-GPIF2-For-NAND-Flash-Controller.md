---
layout: post
title: "Design FX3 GPIF2 for Nand Flash Controller"
categories: STM32
date: 2019-04-28
tags: [FX3, Vendor Command]
---

## Introduction
   Cypress FX3 USB superspeed controller has a built-in GPIF2 interface.  This project will design a protocol to drive the GPIF2 to control NAND flash, especially SamSung K9K8G08U0A.  

## NAND Flash Control Signals
   
   Name     | Pin No    | Description      |
   -----:   |--------:  |:-----------------|
   CE~      | 9         |
   R/B~     | 7         |
   RE~      | 8         |
   CLE      | 16        |
   ALE      | 17        |
   WE~      | 18        |
   WP~      | 19        | Connect to 1 |
   IO0-IO3  | 29-32     |
   IO4-IO7  | 41-44     |

## FX3 Hardware Design
   * There are 8-16 NAND flash attached.
   * Use 4 input DEMUX to choose active NAND flash.
   * Use 4 input MUX to select R/B~ of active NAND flash.
   * Other control signals and IO are connected to all NAND flash.  The control signal might need to bus driver IC for more fanout.  IO might need bi-directional bus driver.  The direction could be another control signal DIR.
   
## FX3 Nand Flash Interface
 1. ALE, CLE, and DIR use vendor command to control GPIO.
 2. RE~, WE~ use GPIF2 GPIO_DR to control signal.
 2. IO0~7 used in GPIF2 IN_DATA and OUT_DATA.
 3. R/B~ use vendor command to readGPIO pin.  
 4. CE~ and MUX, DEMUX selection signals might use vendor command.
 6. WP~ is wired to high in the module.  So no control signal is used.
 
## Vendor Commands
     
   Name       | Command   | Commands      |
   ----:      | ------    | :-------------|
   IO DIR     |  0        | Set IO direction, deassert ALE, CLE
   CE         |  1        | assert CE~
   CMD        |  2        | assert CLE, deassert ALE
   ADDR       |  3        | assert ALE, deassert CLE  
   READRB     |  F        | Read R/B~ pin
   
## BULK IN/OUT 
   * BULK IN will read data from IO0-7 and use GPIF2 to generate RD~ signal 
   * BULK OUT will write data to IO0-7 and use GPIF2 to gernate WR~ signal

## NAND Commmand
   Nand command are sequence of vendor commands and BULK IN/OUT to generate NAND control signals.

  
     


