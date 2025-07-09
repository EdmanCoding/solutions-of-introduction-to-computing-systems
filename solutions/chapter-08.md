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
;Subroutines for carrying out the PUSH and POP functions. This
;program works with a stack consisting of memory locations x3FFF
;through x3FF6 (5 elements, 2 memory locations each). R6 is the stack pointer.
;input: R0 - second memory location, R3 - first
POP AND R5, R5, #0  ; R5 <--success
    ST  R1, Save1   ; Save registers that
    ST  R2, Save2   ; are needed by POP
    LD  R1, EMPTY   ; EMPTY contains -x4000
    ADD R2, R6, R1  ; Compare stack pointer to x4000
    BRz fail_exit   ; Branch if stack is empty

    LDR R3, R6, #0  ; The actual"pop"
    ADD R6, R6, #1  ; Adjust stack pointer
    LDR R0, R6, #0  ; The actual"pop"
    ADD R6, R6, #1  ; Adjust stack pointer
    BRnzp success_exit
    
PUSH    AND R5, R5, #0
        ST  R1, Save1   ; Save registers that
        ST  R2, Save2   ; are needed by PUSH
        LD  R1, FULL    ; FULL contains -x3FF6
        ADD R2, R6, R1  ; Compare stack pointer to x3FF6
        BRz fail_exit   ; Branch if stack is full

                ADD R6, R6, #-1 ; Adjust stack pointer
                STR R0, R6, #0  ; The actual "push"
                ADD R6, R6, #-1 ; Adjust stack pointer
                STR R3, R6, #0  ; The actual "push"
success_exit    LD  R2, Save2   ; Restore original
                LD  R1, Save1   ; register values
                RET
                
fail_exit   LD  R2, Save2   ; Restore original
            LD  R1, Save1   ; register values
            ADD R5, R5, #1  ; R5 <--failure
            RET

EMPTY   .FILL xC000 ; EMPTY contains -x4000
FULL    .FILL xC009 ; FULL contains -x3FF6
Save1   .FILL x0000
Save2   .FILL x0000
```
---
7. Solution:

```assembly
.ORIG x3000
;Subroutines for carrying out the PUSH and POP functions. This
;program works with a stack consisting of 5 elements of arbitrary sizes.
;The stack starts at x3FFF and grows downward (to x3FFE, x3FFD, etc.).
;input: R1 - number of locations (from 1 to 4); R0, R2, R3, R6 - values
    ADD R1 R1 #4        ; this block is for testing
    ADD R0 R0 #13
    ADD R2 R2 #9
    ADD R3 R3 #5
    ADD R6 R6 #10
    JSR PUSH
    ADD R1 R1 #-2
    ADD R0 R0 #3
    ADD R2 R2 #6
    JSR PUSH
    JSR POP
    JSR POP
    JSR POP
    HALT

POP ST  R4 Save1
    LD  R5 NUMBER_OF_ELEMENTS
    BRz fail_exit2                  ; Branch if stack is empty
    ADD R5 R5 #-1           
    ST  R5 NUMBER_OF_ELEMENTS       ; save new number of elements after current pop
    LEA R4 NUMBER_OF_LOC_IN_EL_1    
    ADD R4 R4 R5                    ; select top element
    LDR R5 R4 #0                    ; load number of values at the top element to R5
   
    ADD R4 R5 #-4                   ; choose appropriate size by matching the number of values
    BRz SIZE_4
    ADD R4 R5 #-3
    BRz SIZE_3
    ADD R4 R5 #-2
    BRz SIZE_2
    ADD R4 R5 #-1
    BRz SIZE_1
    BR  fail_exit2 

SIZE_4  LD R5 POINTER       ; Load output registers,
        LDR R6 R5 #0        ; set pointer to the new top 
        LDR R3 R5 #1
        LDR R2 R5 #2
        LDR R0 R5 #3
        ADD R5 R5 #4
        BR  RETURN
SIZE_3  LD R5 POINTER
        LDR R3 R5 #0
        LDR R2 R5 #1
        LDR R0 R5 #2
        ADD R5 R5 #3
        BR  RETURN
SIZE_2  LD R5 POINTER
        LDR R2 R5 #0
        LDR R0 R5 #1
        ADD R5 R5 #2
        BR  RETURN
SIZE_1  LD R5 POINTER
        LDR R0 R5 #0
        ADD R5 R5 #1
        BR  RETURN
RETURN  ST  R5 POINTER      ; save pointer to the new top
        LD  R4 Save1        
        AND R5 R5 #0        ; R5 <--success
        RET
    
    
PUSH    ST  R0 Save1
        ST  R4 Save2
        ST  R0 LOCATION1            ; Store R0 right away since R0 is the first value 
                                    ; and quantity of pushed values can't be less than 1.
        LD  R0 NUMBER_OF_ELEMENTS   ; How much elements in the stack alredy exist?
        ADD R0 R0 #-5               ; It shouldn't be more than 5, since 5 is a full stack.
        BRz fail_exit               ; If stack alredy full - return from the subroutine .
        ADD R0 R1 #-4               ; Number of values we push shouldn't be more than 4.
        BRp fail_exit               ; If more than 4 - return from the subroutine.
        ST  R1 NUMBER_OF_LOCATIONS  ; Save namber of values we want to push.
        
        AND R0 R0 #0                ; This block finds appropriate routine for specified number of values 
        ADD R0 R0 #-1               
        ADD R5 R0 R1
        BRz SIZE_ONE
        ADD R0 R0 #-1
        ADD R5 R0 R1
        BRz SIZE_TWO
        ADD R0 R0 #-1
        ADD R5 R0 R1
        BRz SIZE_THREE
        ADD R0 R0 #-1
        ADD R5 R0 R1
        BRz SIZE_FOUR
        BR fail_exit
SIZE_ONE    LD  R5 POINTER      ; routine for 1 value (R0 alredy stored)
            LEA R0 LOCATION1
            BR  PUSHING
SIZE_TWO    ST  R2 LOCATION2    ; routine for 2 values
            LD  R5 POINTER
            LEA R0 LOCATION1
            BR  PUSHING
SIZE_THREE  ST  R2 LOCATION2    ; routine for 3 values
            ST  R3 LOCATION3
            LD  R5 POINTER
            LEA R0 LOCATION1
            BR  PUSHING
SIZE_FOUR   ST  R2 LOCATION2    ; routine for 4 values
            ST  R3 LOCATION3
            ST  R6 LOCATION4
            LEA R0 LOCATION1
            LD  R5 POINTER
PUSHING ADD R5 R5 #-1           
        LDR R4 R0 #0
        STR R4 R5 #0            ; push approptiate quantity of values to the stack
        ADD R0 R0 #1
        ADD R1 R1 #-1
        BRp PUSHING
        ST  R5 POINTER                  ; save new top
        LD  R0 NUMBER_OF_ELEMENTS
        LD  R1 NUMBER_OF_LOCATIONS
        LEA R5 NUMBER_OF_LOC_IN_EL_1    
        ADD R5 R5 R0
        STR R1 R5 #0                    ; save the number of values in the current element
        ADD R0 R0 #1
        ST  R0 NUMBER_OF_ELEMENTS       ; save new number of elements
        
success_exit    LD  R4 Save2
                LD  R0 Save1   
                AND R5 R5 #0    ; R5 <--success
                RET
                
fail_exit   LD  R4 Save2
            LD  R0 Save1   
            ADD R5 R5 #1  ; R5 <--failure
            RET
fail_exit2  LD  R4 Save1
            ADD R5 R5 #1  ; R5 <--failure
            RET            

NUMBER_OF_LOCATIONS .FILL x0000     ; number of locations we want to PUSH
NUMBER_OF_ELEMENTS  .FILL x0000     ; current number of elements in the stack (5 is full)
LOCATION1           .FILL x0000     ; this block is needed for PUSH
LOCATION2           .FILL x0000
LOCATION3           .FILL x0000
LOCATION4           .FILL x0000
POINTER             .FILL x4000
NUMBER_OF_LOC_IN_EL_1     .FILL x0000   ; this block counts number of values in each stack element
NUMBER_OF_LOC_IN_EL_2     .FILL x0000
NUMBER_OF_LOC_IN_EL_3     .FILL x0000
NUMBER_OF_LOC_IN_EL_4     .FILL x0000
NUMBER_OF_LOC_IN_EL_5     .FILL x0000
Save1   .FILL x0000
Save2   .FILL x0000
.END
```

---
8. Solution:
   1. F and A.
   2. At point when the stack consist of A, C, D.
   3. A, F, and M.
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
