---
layout: post
title: "Use Interrupt Priority to Enable Nested Interrupt Handling for H8/300H"
date: 2023-03-23
---

## Problem Description

* SCI0 (UART) lost receiving data frequently, with baudrate 19200.  

## Analysis

* System have more than one timer interrupt handling that might prevent SCI0 interrupt handling in time.  

## Interrupt Background of H8/300H

* CCR: Condition Code Register
  * Bit 7Interrupt Mask Bit (I): Masks interrupts other than NMI when set to 1. NMI is
accepted regardless of the I bit setting. The I bit is set to 1 at the start of an exception-handling
sequence.

  * Bit 6User Bit or Interrupt Mask Bit (UI): Can be written and read by software using the
LDC, STC, ANDC, ORC, and XORC instructions. This bit can also be used as an interrupt
mask bit. For details see section 5, Interrupt Controller.

* Table 5.4 UE, I, and UI Bit Settings and Interrupt Handling

| UE(SYSCR) | I(CCR) | UI(CCR) | Description                                                                        |
|-----------|--------|---------|------------------------------------------------------------------------------------|
| 1         | 0      | -       | All interrupts are accepted. Interrupts with priority level 1 have higher priority.|
| 1         | 1      | -       | No interrupts are accepted except NMI.                                             |
| 0         | 0      | -       | All interrupts are accepted. Interrupts with priority level 1 have higher priority.|
| 0         | 1      | 0       | NMI and interrupts with priority level 1 are accepted.                             |
| 0         | 1      | 1       | No interrupts are accepted except NMI.                                             |

* Interrupt Handling with UE = 0

![H8 Interupt Handling](assets/2023-03-23/H8_Interrupt_handling.png)

* Nested Interrupt Handling
  * Setup: SYSCR.UE = 1(Enable UE), IRPB.B3 = 1(SCI0 at high priority).  Timer interrupt use the interrupt priority(low)

  * Timer handler: CCR.UI = 0 at first time.  This will enable nested interrupt for high priority interrupt.
  
## Solutions

### Settings

* Set SCI0 at high priority

```c
   // priority SCI0 at high
   IPRB |= 0x08;
```

* Enable UE, so that UI of CCR function as interrupt mask

```c
// SYSCR clear UE, BIT 3, UI in CCR is interrupt mask
   SYSCR &= ~0x8;
```

### Lower Priority Handlers

* At timer handlers, add the following instruction at first line.  In this example, Timer0 handler is defined as

```c
void DoTimer0Handler(void)
{
    __asm("ANDC.B   #191,CCR"); // UI(CCR bit 6) = 0

...
}
```

or (depending on available compiler options)

```c
void DoTimer0Handler(void)
{
    asm("ANDC.B   #191,CCR"); // UI(CCR bit 6) = 0
...
}
```

If no `asm` or `__asm` available, define it as a public function with assembly language.  And call it at the first time of timer handler

```s20
    public clearUI
clearUI:
    ANDC.B #191,CCR
    RTS
```

```c
extern clearUI(void);
void DoTimer0Handler(void)
{
    clearUI();
    ...


}    
```
