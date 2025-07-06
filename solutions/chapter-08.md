# Chapter 08

1. It works with last-in first-out mechanism. So when you insert an element, it adds to the top of the list, and when you get an element from the list, it gives you the last element you added.
---
2.  
   - **No Data Shifting Overhead**  
Figure 8.8 (Hardware Registers):

Every PUSH/POP requires moving all existing stack entries up or down.

Example: Pushing 12 in (b) forces 18 to shift down. This is an O(n) operation for *n* elements.

Figure 8.9 (Memory):

Only the stack pointer (e.g., R6) moves; data stays fixed in memory.

PUSH/POP are O(1) operations (just update the pointer).

   - **Unlimited Depth**  
Hardware Registers: Limited by the number of physical registers (e.g., 5 in Figure 8.8).

Memory: Only limited by available memory (e.g., LC-3’s 65,536 addresses).

   - **Better for Recursion/Subroutines**  
Memory stacks handle nested calls gracefully (no register shuffling).

Critical for LC-3’s JSR/RET instructions, which rely on R6 (stack pointer).

   - **Energy Efficiency**  
Moving data in registers consumes more power than memory pointer updates.
---
3. Solution:
   1. PUSH  R1
   2. POP   R0
   3. PUSH  R3
   4. POP   R7
---
4. Solution:
   ``` 
   ST R1 S1
   ST R2 S2
   LD R1 EMPTY
   ADD R2 R6 R1
   BRz UNDERFLOW
   LDR R0 R6 #0
   RET
   ```  
   Overflow checks are unnecessary since peek doesn’t push to the stack.
---
5. Solution:

```assembly
;Subroutines for carrying out the PUSH and POP functions. This
;program works with a stack consisting of memory locations x3FFF
;through x3FFB. 
    .ORIG x4001
        LD R0 EIGHTEEN      ;This code just
        JSR PUSH            ;for checking 
        BRp QUIT            ;the stack code 
        LD R0 THIRTY_ONE    ;with the different
        JSR PUSH            ;sceneries.
        BRp QUIT
        AND R0 R0 #0
        ADD R0 R0 #5
        JSR PUSH
        BRp QUIT
        AND R0 R0 #0
        ADD R0 R0 #12
        JSR PUSH
        ADD R0 R0 #5
        JSR PUSH
        ADD R0 R0 #5
        JSR PUSH
        BRp QUIT
        JSR POP
        BRp QUIT
        JSR POP
        BRp QUIT
        JSR POP
        BRp QUIT
        JSR POP
        BRp QUIT
        JSR POP
        BRp QUIT

POP     ST  R2 Save1
        ST  R3 Save2
        ST  R4 Save3
        LD  R2 NUMBER_OF_LOCATIONS  ;R2 = number of occupied locations of the stack
        BRz fail_exit               ;is stack empty?
        LDI R0 TOP
        LD  R3 TOP
        ADD R2 R2 #-1               ;Store new number of
        ST  R2 NUMBER_OF_LOCATIONS  ;occupied locations.
        BRz DONE                    ;done if R2 was 1
LOOP    ADD R3 R3 #1                ;else rearrange the stack
        LDR R4 R3 #0
        STR R4 R3 #-1
        ADD R2 R2 #-1
        BRp LOOP
DONE    AND R4 R4 #0
        STR R4 R3 #0                ;clear last location
        BR  success_exit
        
PUSH    ST  R2 Save1
        ST  R3 Save2
        ST  R4 Save3
        ST  R6 Save4
        LD  R2 NUMBER_OF_LOCATIONS  ;R2 = number of occupied locations of the stack
        ADD R3 R2 #-5               ;stack consist of 5 locations
        BRz fail_exit2              ;is stack full?
        LD  R4 TOP
        ADD R2 R2 #0                
        BRz DONE2                   ;is stack empty?
LOOP2   ADD R3 R4 R2                ;else rearrange the stack
        LDR R6 R3 #-1
        STR R6 R3 #0
        ADD R2 R2 #-1
        BRp LOOP2
DONE2   STR R0 R4 #0                ;store new value at the top
        LD  R2 NUMBER_OF_LOCATIONS
        ADD R2 R2 #1                ;Store new number of
        ST  R2 NUMBER_OF_LOCATIONS  ;occupied locations.
        BR  success_exit2
        
success_exit    LD  R4, Save3
                LD  R3, Save2     ;Restore original
                LD  R2, Save1     ;register values.
                AND R5, R5, #0    ;R5 <--success
                RET
success_exit2   LD  R6, Save4
                LD  R4, Save3
                LD  R3, Save2     ;Restore original
                LD  R2, Save1     ;register values.
                AND R5, R5, #0    ;R5 <--success
                RET

fail_exit   LD  R4, Save3
            LD  R3, Save2     ;Restore original
            LD  R2, Save1     ;register values.
            AND R5, R5, #0
            ADD R5, R5, #1    ;R5 <--failure
            RET
fail_exit2  LD  R6, Save4
            LD  R4, Save3
            LD  R3, Save2     ;Restore original
            LD  R2, Save1     ;register values.
            AND R5, R5, #0
            ADD R5, R5, #1    ;R5 <--failure
            RET
QUIT    HALT

TOP     .FILL x3FFB
NUMBER_OF_LOCATIONS .FILL x0000 ;number of occupied locations of the stack

Save1   .FILL x0000
Save2   .FILL x0000
Save3   .FILL x0000
Save4   .FILL x0000

EIGHTEEN     .FILL x0012
THIRTY_ONE   .FILL x001F
    .END
```

---
6. Solution:

```assembly
PUSH    ADD R6, R6, #-2
        STR R0, R6, #0
        STR R1, R6, #0

POP     LDR R0, R6, #0
        LDR R1, R6, #0
        ADD R6, R6, #2
```

---
7. Solution:

```assembly
POP             AND     R5,R5,#0 ; R5 <-- success
                ST      R1,Save1 ; Save registers that
                ST      R2,Save2 ; are needed by POP
                LD      R1, COUNT    ; R1 <-- Current element count
                LD      R2, CAPACITY ; R2 <-- Negative stack capacity
                ADD     R3, R1, R2
                NOT	    R3,	R3
                ADD     R3, R3, #1
                ADD	    R3,	R3,	R2  ; If count + (-capacity) + capacity = 0,
                                    ; it means stack is empty.
                BRz     fail_exit
                ;
                LDR     R0,R6,#0 ; The actual "pop"
                ADD     R6,R6,#1 ; Adjust stack pointer
                BRnzp   success_exit
                ;
PUSH            AND     R5,R5,#0
                ST      R1,Save1 ; Save registers that
                ST      R2,Save2 ; are needed by PUSH
                LD      R1, COUNT    ; R1 <-- Current element count
                LD      R2, CAPACITY ; R2 <-- Negative stack capacity
                ADD	    R3,	R1,	R2  ; If count + (-capacity) = 0,
                                    ; it means stack is full.
                ;
                ADD     R6,R6,#-1 ; Adjust stack pointer
                STR     R0,R6,#0 ; The actual "push"
success_exit    LD      R2,Save2 ; Restore original
                LD      R1,Save1 ; register values
                LD      R3,Save3
                RET
                ;
fail_exit       LD      R2,Save2 ; Restore original
                LD      R1,Save1 ; register values
                LD      R3,Save3
                ADD     R5,R5,#1 ; R5 <-- failure
                RET
                ;
BASE            .FILL   xC006   ; Start address of stack
COUNT           .FILL	xC007   ; Element count of stack at the moment
CAPACITY        .FILL   xC008   ; Negative stack capacity
Save1           .FILL   x0000
Save2           .FILL   x0000
Save3           .FILL   x0000
```

---
8. Solution:
   1. F and A.
   2. After PUSH J and PUSH K.
   3. Just M.
---
9. Solution:
   1. BDECJKIHLG
   2. PUSH Z, PUSH Y, POP, PUSH X, POP, PUSH W, PUSH V, POP, PUSH U, POP, POP, POP, PUSH T, PUSH S, POP, PUSH R, POP, POP
   3. Number of valid output permutations is determined by the Catalan number for the given input length. The Catalan number 5 is 14.
---
10. I couldn't understand how this question works.
---
11. Solution:
    1. 16.
    2. x400F
    3. Average of 4 consecutive values in memory.
---
12. I couldn't understand how this question works.
---
13. Solution:

```assembly
FACT    ST  R1,SAVE_R1
        ADD R1,R0,#0
        BRz ZERO
        ADD R0,R0, #-1
        BRz DONE
        BRnzp AGAIN
ZERO    ADD R0, R0, #1
        BRnzp DONE
AGAIN   MUL R1,R1,R0
        ADD R0,R0,#-1 ; R0 gets next integer for MUL
        BRnp AGAIN
DONE    ADD R0,R1,#0 ; Move n! to R0
        LD  R1,SAVE_R1
        RET
SAVE_R1 .BLKW 1
```

---
14. Partial Solution:
    1.  JSR X
    2.  LDR R1, R3, #1
    3.  LDR R2, R4, #1
    4.  ADD R1, R1, R0
    5.  STR R0, R5, #1
    6.  -
    7.  -
    8.  -
---
15. This problem would not belong to Chapter 08.
