# Chapter 07

1. xA7FE
---
2. x23FF
---
3. "AND" is a reserved word for LC-3 assembler, so it can't be used for any other purposes. If it used, assembler takes AND as a opcode, not label; so it will throw an error in the second pass.
---
4. Solution:
   | SYMBOL | ADDRESS |
   | ------ | ------- |
   | SAVE3 | x3029 |
   | SAVE2 | x302A |
   | TEST | x301F |
   | FINISH | x3027 |
---
5. It multiply x0004 with x0803. Result will be x200C.
---
6. Solution:
   1. A: 0010 0010 0000 1000
   2. B: 0001 0010 0111 1111
   3. D: 0000 0011 1111 1101
   4. It sums odd numbers between 0 to E.
---
7. Solution:
```
Most significant bit (MSB) method by Erkam (R3 is counter of 1s):
        .ORIG   x3000
                LD      R4, TOTALBITCOUNT
                AND     R3, R3, #0  ;(ONECOUNT)
                AND     R2, R2, #0  ;(TEMP)
                LD      R1, TESTNUMBER
                LD      R0, NUMBER
                BRn     NEGNUMLOOP
                BRzp    POSNUMLOOP
NEGNUMLOOP      ADD     R3 R3 #1
POSNUMLOOP      ADD     R0 R0 R0
                ADD     R4 R4 #-1
                BRz     DONE
                AND     R2 R1 R0
                BRzp    POSNUMLOOP
                ADD     R3, R3, #1
                BRnzp   POSNUMLOOP
DONE            HALT
NUMBER          .FILL   xC0F1
TOTALBITCOUNT   .FILL   x0010
TESTNUMBER      .FILL   x8000
        .END

My initial solution with mask method:
.ORIG x3000
        LD R0 NUMBER
        AND R2 R2 #0
        ADD R2 R2 #1        ; mask
        AND R1 R1 #0        ; 1's counter
        AND R4 R4 #0
        ADD R4 R4 #15       ; loop counter
LOOP    AND R3 R0 R2
        BRz NOT_0
        ADD R1 R1 #1
NOT_0   ADD R2 R2 R2
        ADD R4 R4 #-1
        BRzp LOOP
        HALT
NUMBER      .FILL xF001
.END
```
---
8. Solution:
```
R0: 0xA400
R1: 0x23FF
R2: 0xE1FF
R3: 0xA401
```
---
9. .END pseudo-op is just for indicator to tell the assembler program ends at that point but it doesn't actually terminates the program it even won't existing in runtime. But halt terminates the program.
---
10. Literal #30, exceeds range of literal of ADD operation. ADD operation literal can be -16 to 15. Assembler will detect incorrect use of literal, so it will be detect when assembling.
---
11. Solution:
```
.ORIG	x3000
        IN		        ; input the first char - either x or # 
        AND	R3, R3, #0
        ADD	R3, R3, #9  ; R3 = 9 if we are working
                        ; with a decimal or 16 if hex
        LD	R4, NASCIID     ;-x30 or -48
        LD	R5, NHEXDIF     ;-x11 or -17

        LD	R1, NCONSD      ;-x23 or -35
        ADD	R1, R1, R0
        BRz	GETNUMS
        LD	R1, NCONSX      ;-x78 or -120
        ADD	R1, R1, R0
        BRnp FAIL
        ADD	R3, R3, #6	; R3 = 15

GETNUMS IN
        ST	R0, CHAR1 
        IN
        ST	R0, CHAR2
        LEA	R6, CHAR1
        AND	R2, R2, #0
        ADD	R2, R2, #2	; Loop twice
                        ; Using R2, R3, R4, R5, R6 here
        AND	R0, R0, #0	; Result
LOOP	ADD	R1, R3, #0
        ADD	R7, R0, #0 	
LPCUR   ADD	R0, R0, R7
        ADD	R1, R1, #-1
        BRp	LPCUR

        LDR	R1, R6, #0
        ADD	R1, R1, R4
        ADD	R0, R0, R1
        ADD	R1, R1, R5
        BRn	DONECUR
        ADD	R0, R0, #-7	; for hex numbers
DONECUR ADD	R6, R6, #1
        ADD	R2, R2, #-1
        BRp	LOOP
        ; R0 has number at this point 
        AND	R2, R2, #0
        ADD	R2, R2, #8

        LEA	R3, RESEN_D
        LD	R4, ASCNUM
        AND	R5, R5, #0
        ADD	R5, R5, #1

STLP	AND	R1, R0, R5
        BRp	ONENUM
        ADD	R1, R4, #0
        BRnzp	STORCH 
ONENUM	ADD	R1, R4, #1 
STORCH	ADD	R5, R5, R5
        STR	R1, R3, #-1
        ADD	R3, R3, #-1
        ADD	R2, R2, #-1
        BRp	STLP
        LEA	R0, RES 
        PUTS
FAIL	HALT 
CHAR1	.FILL	x0 
CHAR2	.FILL	x0
ASCNUM	.FILL	x30	
NHEXDIF	.FILL	xFFEF	;	-x11   -17
NASCIID	.FILL	xFFD0	;	-x30   -48
NCONSX	.FILL	xFF88	;	-x78  -120
NCONSD	.FILL	xFFDD	;	-x23   -35

RES	    .BLKW 8 
RESEN_D	.FILL x0
 .END
```
---
12. The program compares the first and second 8-bit halves of the value at x4000. R5 is set to 1 if they match, 0 otherwise.
---
13. There is no SUM label and it will be detected at the assembly time because assembler wants to change this label with actual values. Also R1 is not initialized before being used. This will may occur wrong results and it will be debugable at the runtime. The labels 'ONE', 'TWO', and 'THREE' serve no purpose, so they should be converted into comments.
```
    .ORIG x3000
        LD  R0 A       ;ONE
        AND R1 R1 #0
        ADD R1 R1 R0
        LD  R0 B       ;TWO
        ADD R1 R1 R0
        LD  R0 C       ;THREE
        ADD R1 R1 R0
        ST  R1 SUM
        TRAP x25
A   .FILL x0001
B   .FILL x0002
C   .FILL x0003
SUM .FILL x0000
    .END
```
---
14. Solution:
- a.
	```
 	1011 0000 0000 0010
 	1111 0000 0010 0001
 	1111 0000 0010 0101
 	0000 0000 0010 0101
	```
- b. Instead of use STI it should use LD. Because TRAP instruction uses register to display. STI uses for storing data from register to memory not to load data from memory to register to use for TRAP.
- c. STI will attempt to store the value from R0 at memory location x0025, which is reserved for system routines. This is why executing 'STI R0, LABEL' will cause an access violation.
---
15. It checks the MSB of the numbers and if they are positive, it doubles and store backs to their original memory address.
---
16. The program counts the number of even and odd integers in the sequence stored at x4000, storing the results in registers R3 and R4 respectively.
```
.ORIG x3000
        AND     R4, R4, #0	; counter for ODD numbers
        AND     R3, R3, #0	; counter for EVEN numbers
        LD      R0, NUMBERS	; Initialize pointer to x4000
LOOP    LDR     R1, R0, #0	; Load current integer into R1
        NOT     R2, R1		; R2 = NOT R1 (invert bits)
        BRz     DONE		; If R2=0 (i.e., R1 was xFFFF/-1), end program
        AND     R2, R1, #1	; Test LSB: R2 = R1 & 1 (0=even, 1=odd)
        BRz     L1		; If R2=0 (even), jump to L1
        ADD     R4, R4, #1	; ODD: Increment R4 (odd counter)
        BRnzp   NEXT		; Skip even increment
L1      ADD     R3, R3, #1	; EVEN: Increment R3 (even counter)
NEXT    ADD     R0, R0, #1	; Move pointer to next integer
        BRnzp   LOOP		; Repeat loop
DONE    TRAP    x25
NUMBERS .FILL   x4000
 .END
```
---
17. It won't be problem because each module has own symbol table and assembler only checks external modules if an address labeled as EXTERNAL.
---
18. Solution:
```
    .ORIG x3000
        LD  R1 FIRST
        LD  R2 SECOND
        AND R0 R0 #0
LOOP    LDR R3 R1 #0    ; (a)
        LDR R4 R2 #0    
        BRz NEXT
        ADD R1 R1 #1
        ADD R2 R2 #1
        NOT R4 R4       ; (b)
        ADD R4 R4 #1    ; (c)
        ADD R3 R3 R4	; if characters the same - result is zero
        BRz LOOP
        AND R5 R5 #0	; strings aren't the same
        BRnzp DONE	; stop the program
NEXT    AND R5 R5 #0
        ADD R5 R5 #1
DONE    TRAP x25
FIRST   .FILL x4000
SECOND  .FILL x4100
    .END
```
---
19. The DATA labeled address holds #11, and the loop cycle decrements by 3 in every loop, so it will be executed only 4 times.
---
20. First one dynamically initialize value in runtime, but the second one initialize value at the assembly time.
---
21. It counts negative numbers between x4000 and x400A and stores count at the x5000.
---
22. Not try to solve. It'll take too much time.
---
23. Solution :
    1. ADD R1 R1 #-1
    2. LDR R4, R1 #0
    3. ADD R0, R0, #1
    4. ADD R1, R1, #-1
    5. BRnzp LOOP
---
24. Because first line of LOOP doesn't check the R2 which is for tracking shift count. Lines of ADD R2, R2, #-1 and ADD R3, R3, R3 should change their positions.
---
25. It'll give assembly error because it exceed LC-3's memory address capacity.
---
26. They'll give same output. Only difference is label naming and this doesn't affect the execution of program.
---
27. Solution:
    1. Can't figure out
    2. Can't figure out.
    3. R1 shift count
    4. R6 holds shifted value/result.
---
28. Can't solve.
---
29. Solution:
    1. a: x1801, d: x1802, f: x1800, h: x1803, I: x0ffd
---
30. Can't solve.
---
31. It counts odd integers between x5000 and x6000 memory addresses.
---
32. Solution:
    1. x8006: 0010001000000011
    2. x8007: 0000010000000001
    3. x8008: 0011000000000010
    4. Line 10 sets #5 at assembly time, but line 7 sets #5 dynamically at runtime.
---
33. Solution:
    1. ADD R2, R2, #1
    2. ADD R0, R1, R0
    3. It is trying to divide a number to another number and find quotient.
    4. It shouldn't do BRnzp AGAIN, it should do BRp AGAIN and add BRn DONE and subtract quotient (result) by 1 at that branch for correct result.
---
34. Solution:
    1. NOT R2, R0
    2. ADD R2, R2, #1
    3. BRz DONE
    4. ADD R0, R0, #1
---
35. Solution:
	![Solution](_attachments/Pasted%20image%2020250110232306.png)
---
36. Solution:
    1. R0 = A, R1 = B, R2 = -B
    2. At the line 10, instead of ST it should be STI.
---
37. Solution:
    ![Solution](_attachments/Pasted%20image%2020250110234939.png)
