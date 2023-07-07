## Reconstruction of the Rampant demo

This is the code (or part of it) for a demo called ‘Rampant’ which I originally wrote back around 1991.  This demo was also itself part of a bigger collection of demos (mega demo) called the Brain dead Mega demo for the Archimedes computer (A3000 was the machine I coded it on). The demo was written in arm2 assembly.

To run this on arculator emulator:-

Copy the GitHub files to your PC to a folder called ‘Rampant’ inside the aculator ‘hostfs’ folder.  
Then when running arculator, click on the ‘Rampant’ folder and then double click on the ‘rampant/txt’ file, which assembles the demo and then runs it.

## Motivation
The original assembly source code for this demo has long been lost.  As it has been such a long time, I was interested to remember how I actually coded this.  Luckily binaries of this demo still exists today, so I simply managed to ‘decompress’ the rampant demo binary, then uploaded it to an online arm disassembler.    From looking at the output I understood (remembered) enough to then try to reconstruct the original assembly source code adding more meaningful variable names and comments.

The reconstruction is only looking at the colour wiggly pipes effect part of the rampant demo

 ![App Screenshot]( https://github.com/stuart73/Rampant-reconstructed/blob/main/demooutput2.jpg)

## Notes on how I coded this (for myself mostly)
As with all demos of that time, you simply could not code it as you would expect something like that needed to be coded.  This would be toooo slow!

For the Archimedes a3000,  To get 320x256 screen with 256 colours running at 50 frames a second, then the only thing that can do that is a blit sprite type of copy to the screen using LDMIA and STMIA assembly instructions.  These instructions simply load multiple data into a range of 32 bit registers and then stores multiple them to somewhere else (typically screen memory).  So the vast majority of the cpu time spent drawing to the sceen has to be done using these (fast) instructions.

The core drawing routine for this demo comes down to the below code.  This code draws (renders) one horizontal line at a time.  It is then repeated in a loop to draw the whole screen. 

.draw_tube_line_loop

ldr r11, start_tube_buffer_address

ldr r0,[r14],#4

add r11,r11,r0	; offset from value in sin table 1

ldr r0,[r9],#8

add r11,r11,r0 ; offset from value in sin table 2


;find which buffer copy to use

movs r1,r11,asr#1

addcs r11, r11, #640

movs r1,r1,asr#1

addcs r11, r11, #1280


;draw whole line

ldmia r11!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

stmia r12!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

ldmia r11!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

stmia r12!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

ldmia r11!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

stmia r12!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

ldmia r11!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

stmia r12!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

ldmia r11!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

stmia r12!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

ldmia r11!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

stmia r12!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

ldmia r11!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

stmia r12!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

ldmia r11!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

stmia r12!, {r0, r1, r2, r3, r4, r5, r6, r7, r8}

ldmia r11!, {r0, r1, r2, r3, r4, r5, r6, r7}

stmia r12!, {r0, r1, r2, r3, r4, r5, r6, r7}

subs r10, r10, #1

bne draw_tube_line_loop



## How the demo was coded

The demo has 4 layers of colour pipes.  Blue background, then red then yellow and finally green.  Obviously if we draw the whole screen first with blue, then draw the next layer we will simply be drawing over much of the blue layer.  This technique would be far too slow.  So i’ve listed here how to achieve this effect as efficiently as possible on the Archimedes.

## step 1
Only draw a 1 pixel height version of all the 4 layers of the pipes on top of each other.  This 1 pixel height version gets drawn properly (slower), with all the pipes drawn at the right place each frame.  For this, each pipe x position is based on some sine table I pre-calculated before the demo is ran.  For rampant, each coloured layer follows a different sine table giving the effect of the colour pipes moving at different rates. 

The 1 pixel height version (which I will call the offscreen buffer) is a buffer with twice the width of the screen (I.e. 2 x 320  = 640 pixels).  This will become clear later.

The pipes are drawn in a way such that they will screen wrap around.  This means the 640 width buffer is actually two identical 320 pixel buffers. So because of this we only need to draw the first 320 pixels properly (but slowly) and then simply copy the first 320 pixels to the next 320 pixels (640 in total).

## step2
I then make 4 copies of this buffer.  Each copy is 1 pixel (right) offset from the last.  This is because the Archimedes allows you to store a 32 bit value at a 32 bit aligned address.   I.e. We can write 4 pixels at a time (32 bit as 1 pixel represents 8 bit) with one store instruction - great! BUT it has to be on a 32 bit (or 4 pixel) aligned boundary.  So a big trick (to use these fast instructions)  used on the Archimedes is to have 4 copies of a sprite with each copy one pixel shifted right.  Then when we come to draw the sprite we can work out which copy of the sprite to use based on the 32 bit alignment of the storing address (screen memory).  

In the above draw code, the buffer version to use is calculated based on the buffer + offset value in R11’s two least significant bits.   This allows us to draw the offscreen buffer to the screen at a 1 pixel resolution.

; Work out which version of the buffer to use (stored in r11) 

add r11,r11,r0

movs r1,r11,asr#1

addcs r11, r11, #640

movs r1,r1,asr#1

addcs r11, r11, #1280



## step3
The code then repeats the line drawing code from top to bottom, which achieves something like this.

 ![App Screenshot]( https://github.com/stuart73/Rampant-reconstructed/blob/main/demooutput1.jpg)

To make the wiggly effect, we simply adjust the x offset value of the buffer each time we draw a line.  This is done by using two different pre-calculated sin tables.  We also iterate through the first sin table at TWICE the rate as the first.  This then gives us a more moving effect, rather than a scrolling type effect.

ldr r0, tube_table_to_use_index1

add r0,r0,#8

str r0, tube_table_to_use_index1


ldr r3, tube_table_to_use_index2

add r3, r3, #4

str r3, tube_table_to_use_index2


This will result in the full effect.

The x offset values in the two sine tables (when added together) are in the range of 0-320 pixels.   This is why our buffer needs to be 640 pixels wide. I.e. because we need to render 320 pixels reading from the  start of buffer+offset value.

Hope this was fun read!

