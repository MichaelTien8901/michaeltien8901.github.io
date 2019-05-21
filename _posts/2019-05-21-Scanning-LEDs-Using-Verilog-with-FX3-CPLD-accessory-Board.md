---
layout: post
title: "PWM Scanning LEDs Using Verilog with FX3 CPLD accessory Board"
date: 2019-05-21
---
## Hardware

   Cypress FX3 explorer Kit with CPLD accessory Board
   
## Settings
    
    J4(pmode) is closed(USBBoot).
    J5 open    
   
## LEDs using PWM for Shadow 
   Istead of only turn ON/OFF of LEDs, the scanning LEDs use PWM to create the shadow effect of surrounding LEDs.  
   
## Verilog Compoents
 * PWM 
 
    Create 4 PWM output 
    
 ```verilog
 //////////////////////////////////////////////////////////////////////////////////
// @brief
//    Generate 4 PWM output.  LED On when LOW, LED Off when High.
// @param PCLK input clock
// @param PWM_OUT 4 bit PWM output to drive LED.
//    PWM_OUT[0]: Duty cycle 1/16
//    PWM_OUT[1]: Duty cycle 2/16
//    PWM_OUT[2]: Duty cycle 6/16
//    PWM_OUT[3]: Duty cycle 15/16
//////////////////////////////////////////////////////////////////////////////////
module Pwm(
    input PCLK,
    output [3:0] PWM_OUT
    );

reg [3:0] pwm_counter;
reg [3:0] LED1;
assign PWM_OUT = LED1;

always @ (posedge PCLK) begin
   case (pwm_counter)
	   0:
		   begin
		      LED1 <= 4'h0;			
          end
	   1:
		   begin
		      LED1 <= 4'h1;			
         end		  
		2:
		  begin
		     LED1 <= 4'h3;
         end
		6:		
		  begin
		     LED1 <= 4'h7;
        end		  
		15:
		   begin
				LED1 <= 4'hF;
			end		   
   endcase		
   pwm_counter <= pwm_counter + 1;
end
endmodule

 ```

* MUX2
   4 to 1 multiplexer
```verilog
//////////////////////////////////////////////////////////////////////////////////
// @brief
//    4 inputs and multipluxer with 2 bit selection
//	 @param DIN 4 input
//	 @param SEL 2 bit selection
//  @param MUX)OUT one bit output    
///////////////////////////////////////////////////////////////////////////////////

module mux2(
    input [3:0] DIN,
    input [1:0] SEL,
    output reg MUX_OUT
    );

always @( DIN or SEL ) 
begin
   case (SEL)
	   2'h0: MUX_OUT = DIN[0];
		2'h1: MUX_OUT = DIN[1];
		2'h2: MUX_OUT = DIN[2];
		2'h3: MUX_OUT = DIN[3];
   endcase		
end
endmodule
```
* Top.v

   Top level design combine the 8 LED output and clock

```verilog
`timescale 1ns / 1ps

// First declare all the connections for the CPLD board
module top(
	inout [12:0] CTRL,
	inout [31:0] DQ,
	input PCLK,
	inout I2C_SCL,
	inout I2C_SDA,
	input GPIO45_n,
	input [7:0] Button,
	output [7:0] LED,
	inout [7:0] User,
	input SPI_SCK,
	input SPI_SSN,
	input RX_MOSI,
	output TX_MISO,
	output FlashCS_n,
	output INT,
	output TP_2
    );

// Need to assign inputs else they get optimized away
	assign TP_2 = RX_MOSI & SPI_SCK & SPI_SSN & I2C_SCL & I2C_SDA;

// Assign fixed outputs not used in this example
	assign FlashCS_n = 1'b1;
	assign INT = 1'b0;
	assign TX_MISO = 1'bZ;
	
   // pwm signals
   wire [3:0] pwm;
	// divider counter
   reg [23:0] Counter;	
	// LED position counter
	reg [2:0] pos_counter;
	// 8 LED pwm select signals
   reg [1:0] sel0, sel1, sel2, sel3, sel4, sel5, sel6, sel7;
	// pos_counter, up down flag.  1 is up
	reg flag;
	initial flag = 1;
	
	initial sel0 = 2'b00;
	initial sel1 = 2'b01;
	initial sel2 = 2'b10;
	initial sel3 = 2'b11;
	initial sel4 = 2'b00;
	initial sel5 = 2'b01;
	initial sel6 = 2'b10;
	initial sel7 = 2'b11;
	
// generate PWM signals
Pwm pwm1 (
	.PCLK(PCLK),
	.PWM_OUT(pwm)
);
// Connect 8 LED output to MUX output
mux2 mux_led0( 
   .DIN(pwm),
	.SEL(sel0),
	.MUX_OUT(LED[0])
);	
mux2 mux_led1( 
   .DIN(pwm),
	.SEL(sel1),
	.MUX_OUT(LED[1])
);	
mux2 mux_led2( 
   .DIN(pwm),
	.SEL(sel2),
	.MUX_OUT(LED[2])
);	
mux2 mux_led3( 
   .DIN(pwm),
	.SEL(sel3),
	.MUX_OUT(LED[3])
);	
mux2 mux_led4( 
   .DIN(pwm),
	.SEL(sel4),
	.MUX_OUT(LED[4])
);	
mux2 mux_led5( 
   .DIN(pwm),
	.SEL(sel5),
	.MUX_OUT(LED[5])
);	
mux2 mux_led6( 
   .DIN(pwm),
	.SEL(sel6),
	.MUX_OUT(LED[6])
);	
mux2 mux_led7( 
   .DIN(pwm),
	.SEL(sel7),
	.MUX_OUT(LED[7])
);	
// divider counter
always @ (posedge PCLK) 
begin
   Counter <= Counter + 1;
end

// increment or decrement LED position counter (0-8)
// use position counter value mapping to LED PWM selection
always @( posedge Counter[20])
begin
	case (pos_counter)
	3'b000:
	   begin
		   sel0 <= 3;
			sel1 <= 2;
			sel2 <= 1;
			sel3 <= 0;
			sel4 <= 0;
			sel5 <= 0;
			sel6 <= 0;
			sel7 <= 0;
			flag = 1;
		end
	3'b001:
	   begin
		   sel0 <= 2;
			sel1 <= 3;
			sel2 <= 2;
			sel3 <= 1;
			sel4 <= 0;
			sel5 <= 0;
			sel6 <= 0;
			sel7 <= 0;
		end
	
	3'b010:
	   begin
		   sel0 <= 1;
			sel1 <= 2;
			sel2 <= 3;
			sel3 <= 2;
			sel4 <= 1;
			sel5 <= 0;
			sel6 <= 0;
			sel7 <= 0;
	   end
	3'b011:
	   begin
		   sel0 <= 0;
			sel1 <= 1;
			sel2 <= 2;
			sel3 <= 3;
			sel4 <= 2;
			sel5 <= 1;
			sel6 <= 0;
			sel7 <= 0;
	   end
	3'b100:
	   begin
		   sel0 <= 0;
			sel1 <= 0;
			sel2 <= 1;
			sel3 <= 2;
			sel4 <= 3;
			sel5 <= 2;
			sel6 <= 1;
			sel7 <= 0;
	   end
	3'b101:
	   begin
		   sel0 <= 0;
			sel1 <= 0;
			sel2 <= 0;
			sel3 <= 1;
			sel4 <= 2;
			sel5 <= 3;
			sel6 <= 2;
			sel7 <= 1;
	   end
	3'b110:
	   begin
		   sel0 <= 0;
			sel1 <= 0;
			sel2 <= 0;
			sel3 <= 0;
			sel4 <= 1;
			sel5 <= 2;
			sel6 <= 3;
			sel7 <= 2;
	   end
	3'b111:
	   begin
		   sel0 <= 0;
			sel1 <= 0;
			sel2 <= 0;
			sel3 <= 0;
			sel4 <= 0;
			sel5 <= 1;
			sel6 <= 2;
			sel7 <= 3;
			flag = 0;
	   end	
	endcase
	// up or down counter
	if ( flag ) 
	   pos_counter <= pos_counter + 1;
	else
	   pos_counter <= pos_counter - 1;
end
endmodule

```

## Write CPLD using FX3

1. Right click top.v and "implment Top Module"
2. Tools->iMPACT...
3. In the ISE iMPACT, click XC2C128 (and turned green) to select the top.jed.
4. Output->XSVF->Create XSVF File... Use the name 'pwm.xsvf' for the file.
5. At the "iMPACT Processes" panel, click functions at the following order
   Erase, Program, Verify
6. Output->XSVF-> Stop Writing to XSVF File
7. Connect the FX3(with CPLD accessory Board plugged in) with USB3. (Reset FX3 board if already plugged in).
8. Drag the file 'pwm.xsvf' to ProgramCPLD.exe in Windows File explorer.  The Program will start to write the CPLD.
9. In order to show the result, we need to enable the clock output of FX3 to drive the CPLD.  Use USB Control Center with 
the following step,
   * Click to select the FX3 Bootloader Device
   * Program->FX3->RAM, and then select GPIF_Example1.img
   

   
