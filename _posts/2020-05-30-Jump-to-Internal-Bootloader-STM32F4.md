---
layout: post
title: "Jump to Internal Bootloader STM32F4"
tags: [STM32F4, Bootloader]
date: 2020-05-30
---

This project use STM32F407 Disco to implement "jump to internal bootloader" as the article described at [http://stm32f4-discovery.net/2017/04/tutorial-jump-system-memory-software-stm32/](http://stm32f4-discovery.net/2017/04/tutorial-jump-system-memory-software-stm32/). 

## Briefing

STM32 feature of embedded bootloader for software download to flash.  The bootloader is in the system memory and accessiable with BOOT configuraion (like BOOT0 pin or BOOT1 option bytes).

Normally, if you want to jump to system memory, you have to setup pin/OB and reset device.  But it is also possible to jump to the system memory from current program in STM32.

## Software and Hardware Needed

* STM32 Cube Programmer
   This should install with necessary DFU USB driver for STM32 Bootloader.

* Hardware STM32F407 Discovery

We will use simple blinky as demo program.  Use the "User" button press to activate jump to bootloader.

## Test Steps

   Blue LED is blinking initially.
1. Press B1 button, LED turn off.  If you plug USER USB, we can find a "STM32 Bootloader" user driver.

2. Run STM32 Cube Programmer, in the top right menu, select USB and press connect button to start programming new code.

## Some Changes

There are two changes from the original source code.  

1 Add HAL_DeInit function call to deinitialize HAL devices.

2 Delete  __disable_irq functino call.  This could depend on the projects.  

```c
void JumpToBootloader(void) {
    void (*SysMemBootJump)(void);

    /**
     * Step: Set system memory address.
     *
     *       For STM32F429, system memory is on 0x1FFF 0000
     *       For other families, check AN2606 document table 110 with descriptions of memory addresses
     */
    volatile uint32_t addr = 0x1FFF0000;

    /**
     * Step: Disable RCC, set it to default (after reset) settings
     *       Internal clock, no PLL, etc.
     */
#if defined(USE_HAL_DRIVER)
    HAL_RCC_DeInit();
    HAL_DeInit(); // add by ctien
#endif /* defined(USE_HAL_DRIVER) */
#if defined(USE_STDPERIPH_DRIVER)
    RCC_DeInit();
#endif /* defined(USE_STDPERIPH_DRIVER) */

    /**
     * Step: Disable systick timer and reset it to default values
     */
    SysTick->CTRL = 0;
    SysTick->LOAD = 0;
    SysTick->VAL = 0;

    /**
     * Step: Disable all interrupts
     */
//    __disable_irq(); // changed by ctien

    /**
     * Step: Remap system memory to address 0x0000 0000 in address space
     *       For each family registers may be different.
     *       Check reference manual for each family.
     *
     *       For STM32F4xx, MEMRMP register in SYSCFG is used (bits[1:0])
     *       For STM32F0xx, CFGR1 register in SYSCFG is used (bits[1:0])
     *       For others, check family reference manual
     */
    //Remap by hand... {
#if defined(STM32F4)
    SYSCFG->MEMRMP = 0x01;
#endif
#if defined(STM32F0)
    SYSCFG->CFGR1 = 0x01;
#endif
    //} ...or if you use HAL drivers
    //__HAL_SYSCFG_REMAPMEMORY_SYSTEMFLASH();    //Call HAL macro to do this for you

    /**
     * Step: Set jump memory location for system memory
     *       Use address with 4 bytes offset which specifies jump location where program starts
     */
    SysMemBootJump = (void (*)(void)) (*((uint32_t *)(addr + 4)));

    /**
     * Step: Set main stack pointer.
     *       This step must be done last otherwise local variables in this function
     *       don't have proper value since stack pointer is located on different position
     *
     *       Set direct address location which specifies stack pointer in SRAM location
     */
    __set_MSP(*(uint32_t *)addr);

    /**
     * Step: Actually call our function to jump to set location
     *       This will start system memory execution
     */
    SysMemBootJump();

    /**
     * Step: Connect USB<->UART converter to dedicated USART pins and test
     *       and test with bootloader works with STM32 STM32 Cube Programmer
     */
}
```

## Project links

[https://github.com/MichaelTien8901/JmpBldrDemo](https://github.com/MichaelTien8901/JmpBldrDemo)

