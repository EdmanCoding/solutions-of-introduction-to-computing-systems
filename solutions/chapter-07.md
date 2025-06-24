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
21. It counts negative numbers between x4000 and x4009 and stores count at the x5000.
---
22. Solution:
```
; LC-3 program to convert mnemonic to 4-bit opcode
        .ORIG x3000
        ; Prompt user for mnemonic
            LEA   R0, PROMPT      ; R0 = address of "Enter mnemonic: "
            TRAP  x22             ; PUTS: print prompt
            AND   R3, R3, #0      ; reset counter for buffer
            ADD   R3, R3, #5      ; input shuldn't be more than 5 chars
        ; Read characters into buffer until Enter (ASCII 10) is pressed
            LEA   R1, BUFFER      ; R1 = base address of input buffer

READ_LOOP   TRAP  x23             ; IN: read and echo one char into R0
            LD    R2, CHAR_LF     ; Load ASCII code for line feed (0x0A)
            NOT   R2, R2
            ADD   R2, R2, #1      
            ADD   R2, R2, R0      ; Compare R0 to 0x0A
            BRz   DONE_READ       ; If equal, end input
            STR   R0, R1, #0      ; Store character in buffer
            ADD   R1, R1, #1      ; Increment buffer pointer
            ADD   R3, R3, #-1     ; decrement counter 
            BRz      INVALID      ; invalid input if more than 5 chars entered
            BRnzp    READ_LOOP

            ; Null-terminate the input string
DONE_READ   AND   R0, R0, #0      ; R0 = 0
            STR   R0, R1, #0      ; Store null terminator
            LEA   R4, BUFFER      ; R4 = pointer to buffer start

            ; Compare input to each mnemonic
            LDR   R0, R4, #0      ; Load first character

            ; Check first letter 'A' (ADD, AND)
            LD    R2, CHAR_A
            NOT   R2, R2
            ADD   R2, R2, #1
            ADD   R2, R2, R0
            BRz   IS_A          ; If R0=='A', go handle ADD/AND

            ; Check first letter 'B' (BR)
            LD    R2, CHAR_B
            NOT   R2, R2
            ADD   R2, R2, #1
            ADD   R2, R2, R0
            BRz   IS_B           

            ; Check first letter 'J' (JMP, JSR)
            LD    R2, CHAR_J
            NOT   R2, R2
            ADD   R2, R2, #1
            ADD   R2, R2, R0
            BRz   IS_J          ; If R0=='J', go handle JMP/JSR

            ; Check first letter 'L' (LD, LDI, LDR, LEA)
            LD    R2, CHAR_L
            NOT   R2, R2
            ADD   R2, R2, #1
            ADD   R2, R2, R0
            BRz   IS_L          ; If R0=='L', go handle LD/LDI/LDR/LEA

            ; Check first letter 'S' (ST, STI, STR)
            LD    R2, CHAR_S
            NOT   R2, R2
            ADD   R2, R2, #1
            ADD   R2, R2, R0
            BRz   IS_S          ; If R0=='S', go handle ST/STI/STR

            ; Check first letter 'T' (TRAP)
            LD    R2, CHAR_T
            NOT   R2, R2
            ADD   R2, R2, #1
            ADD   R2, R2, R0
            BRz   IS_T
            
            ; Check first letter 'N' (NOT)
            LD    R2, CHAR_N
            NOT   R2, R2
            ADD   R2, R2, #1
            ADD   R2, R2, R0
            BRz   IS_N

            ; If none matched:
INVALID     LEA   R0, ERRMSG      ; R0 = address of "Invalid mnemonic!"
            TRAP  x22             ; PUTS: print error message
            HALT
            
        ; Handle 'N' -> NOT
IS_N    LDR   R1, R4, #1      ; second char (must be 'O')
        LD    R2, CHAR_O
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   IS_NO
        BRnzp    ERROR
IS_NO   LDR   R1, R4, #2      ; third char (must be 'T')
        LD    R2, CHAR_T
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   OUTPUT_NOT
        BRnzp    ERROR  

; Handle first letter 'A' -> could be ADD or AND
IS_A    LDR   R1, R4, #1      ; second character
        LD    R2, CHAR_D
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   IS_AD           ; if 'D', maybe ADD
        LD    R2, CHAR_N
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   IS_AN           ; if 'N', maybe AND
        BRnzp    ERROR

IS_AD   LDR   R1, R4, #2      ; third character for ADD
        LD    R2, CHAR_D
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   OUTPUT_ADD
        BRnzp    ERROR

IS_AN   LDR   R1, R4, #2      ; third character for AND
        LD    R2, CHAR_D
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   OUTPUT_AND
        BRnzp    ERROR

; Handle 'B' -> BR (branch)
IS_B    LDR   R1, R4, #1      ; second character for BR
        LD    R2, CHAR_R
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   B_CHECK_NULL ; If R1=='R', proceed
        BRnzp    ERROR
                ; Also check no extra char (BR is 2 letters)
B_CHECK_NULL    LDR   R1, R4, #2
                BRz   OUTPUT_BR
                BRnzp    ERROR

; Handle 'J' -> JMP or JSR
IS_J    LDR   R1, R4, #1      ; second character
        LD    R2, CHAR_M
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   J_ATM           ; if 'M', it's JMP
        LD    R2, CHAR_S
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   J_ATS           ; if 'S', it's JSR
        BRnzp    ERROR

J_ATM   LDR   R1, R4, #2      ; third character for JMP
        LD    R2, CHAR_P
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   OUTPUT_JMP
        BRnzp    ERROR

J_ATS   LDR   R1, R4, #2      ; third character for JSR
        LD    R2, CHAR_R
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   OUTPUT_JSR
        BRnzp    ERROR

; Handle 'L' -> LD, LDI, LDR, LEA
IS_L    LDR   R1, R4, #1      ; second character
        LD    R2, CHAR_D
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   L_ATD           ; if 'D', could be LD/LDI/LDR
        LD    R2, CHAR_E
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   L_ATE           ; if 'E', it's LEA
        BRnzp    ERROR

L_ATD   LDR   R1, R4, #2      ; third character
        LD    R2, CHAR_I
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   OUTPUT_LDI      ; LDI
        LD    R2, CHAR_R
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   OUTPUT_LDR      ; LDR
        ; If no third char, it's LD
        LDR   R2, R4, #2
        BRz   OUTPUT_LD
        BRnzp    ERROR

L_ATE   LDR   R1, R4, #2      ; third character for LEA
        LD    R2, CHAR_A
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   OUTPUT_LEA
        BRnzp    ERROR
; Handle 'S' -> ST, STI, STR
IS_S    LDR   R1, R4, #1      ; second character (must be 'T')
        LD    R2, CHAR_T
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   S_ATT
        BRnzp    ERROR

S_ATT   LDR   R1, R4, #2      ; third character
        LD    R2, CHAR_I
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   S_ATI           ; STI
        LD    R2, CHAR_R
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   S_ATR           ; STR
        ; If no third char, it's ST
        LDR   R2, R4, #2
        BRz   OUTPUT_ST
        BRnzp    ERROR
        ; matched 'STI'
S_ATI   BRnzp    OUTPUT_STI
        ; matched 'STR'
S_ATR   BRnzp    OUTPUT_STR
; Handle 'T' -> TRAP
IS_T    LDR   R1, R4, #1      ; second char (must be 'R')
        LD    R2, CHAR_R
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   T_ATR
        BRnzp    ERROR

T_ATR   LDR   R1, R4, #2      ; third char (must be 'A')
        LD    R2, CHAR_A
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   T_ATRA
        BRnzp    ERROR

T_ATRA  LDR   R1, R4, #3      ; fourth char (must be 'P')
        LD    R2, CHAR_P
        NOT   R2, R2
        ADD   R2, R2, #1
        ADD   R2, R2, R1
        BRz   OUTPUT_TRAP
        BRnzp    ERROR
        
        ; Print error message if no match
ERROR   LEA   R0, ERRMSG
        TRAP  x22
        BRnzp    HALT_
        ; Input buffer (max 4 chars + null)
BUFFER      .BLKW 5
; Character constants
CHAR_A      .FILL x0041  ; ASCII 'A'
CHAR_B      .FILL x0042  ; 'B'
CHAR_D      .FILL x0044  ; 'D'
CHAR_E      .FILL x0045  ; 'E'
CHAR_I      .FILL x0049  ; 'I'
CHAR_J      .FILL x004A  ; 'J'
CHAR_L      .FILL x004C  ; 'L'
CHAR_M      .FILL x004D  ; 'M'
CHAR_N      .FILL x004E  ; 'N'
CHAR_O      .FILL x004F  ; 'O'
CHAR_P      .FILL x0050  ; 'P'
CHAR_R      .FILL x0052  ; 'R'
CHAR_S      .FILL x0053  ; 'S'
CHAR_T      .FILL x0054  ; 'T'
CHAR_LF     .FILL x000A  ; Line Feed
CHAR_NULL   .FILL x0000  ; Null terminator
PROMPT      .STRINGZ "Enter mnemonic: "  ; Prompt string
ERRMSG      .STRINGZ "Invalid mnemonic!" ; Error message
; Output routines for each mnemonic
OUTPUT_ADD  JSR   NULL_CHECK
            BRnp    ERROR 
            LEA   R0, ADDSTR
            TRAP  x22
            BRnzp    HALT_
OUTPUT_AND  JSR   NULL_CHECK
            BRnp    ERROR 
            LEA   R0, ANDSTR
            TRAP  x22
            BRnzp    HALT_
OUTPUT_NOT  JSR   NULL_CHECK
            BRnp    ERROR 
            LEA   R0, NOTSTR
            TRAP  x22
            BRnzp    HALT_
OUTPUT_BR   LEA   R0, BRSTR
            TRAP  x22
            BRnzp    HALT_
OUTPUT_JMP  JSR   NULL_CHECK
            BRnp    ERROR 
            LEA   R0, JMPSTR
            TRAP  x22
            BRnzp    HALT_
OUTPUT_JSR  JSR   NULL_CHECK
            BRnp    ERROR 
            LEA   R0, JSRSTR
            TRAP  x22
            BRnzp    HALT_
OUTPUT_LD   LEA   R0, LDSTR
            TRAP  x22
            BRnzp    HALT_
OUTPUT_LDI  JSR   NULL_CHECK
            BRnp    ERROR 
            LEA   R0, LDISTR
            TRAP  x22
            BRnzp    HALT_
OUTPUT_LDR  JSR   NULL_CHECK
            BRnp    ERROR 
            LEA   R0, LDRSTR
            TRAP  x22
            BRnzp    HALT_
OUTPUT_LEA  JSR   NULL_CHECK
            BRnp    ERROR 
            LEA   R0, LEASTR
            TRAP  x22
            BRnzp    HALT_
OUTPUT_ST   LEA   R0, STSTR
            TRAP  x22
            BRnzp    HALT_
OUTPUT_STI  JSR   NULL_CHECK
            BRnp    ERROR 
            LEA   R0, STISTR
            TRAP  x22
            BRnzp    HALT_
OUTPUT_STR  JSR   NULL_CHECK
            BRnp    ERROR 
            LEA   R0, STRSTR
            TRAP  x22
            BRnzp    HALT_
OUTPUT_TRAP
            LEA   R0, TRAPSTR
            TRAP  x22
HALT_       TRAP  x25            ; Halt execution
NULL_CHECK  LDR   R1, R4, #3     ; Check: is fourth character 0?
            RET
; --- Data Section ---
; Opcode strings (4-bit codes as characters)
ADDSTR      .STRINGZ "0001"
ANDSTR      .STRINGZ "0101"
NOTSTR      .STRINGZ "1001"
BRSTR       .STRINGZ "0000"
JMPSTR      .STRINGZ "1100"
JSRSTR      .STRINGZ "0100"
LDSTR       .STRINGZ "0010"
LDISTR      .STRINGZ "1010"
LDRSTR      .STRINGZ "0110"
LEASTR      .STRINGZ "1110"
STSTR       .STRINGZ "0011"
STISTR      .STRINGZ "1011"
STRSTR      .STRINGZ "0111"
TRAPSTR     .STRINGZ "1111"
        .END
```
---
23. Solution :
    1. ADD R1 R1 #-1
    2. LDR R4, R1 #0
    3. ADD R0, R0, #1
    4. ADD R1, R1, #-1
    5. BRnzp LOOP
---
24. The first line of LOOP doesn't check the R2 which is for tracking shift count. Lines of ADD R2, R2, #-1 and ADD R3, R3, R3 should change their positions.
```
    .ORIG x3000 
        AND R2 R2 #0
        ADD R2 R2 #4
LOOP    ADD R3 R3 R3
        ADD R2 R2 #-1
        BRz DONE
        BR LOOP          
DONE    HALT
    .END
```
---
25. It'll give assembly error because it exceed LC-3's memory address capacity.
---
26. The two resulting machine language programs are the same. Names of the labels don't effect the machine code. 
---
27. Solution:
    1. The program logical right-shifts the number in R0 by the number in R1 and puts it in RESULT.
    2. R0 holds the input number to right-shift. Range = [x0000 to xFFFF]
    3. R1 holds the amount to right-shift. Range = [1 to 15]
    4. R6 holds the right-shifted output. Range = [x0000 to x7FFF]
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
