# Chapter 10

1. Pops the first value from the stack and checks if the pop operation was successful and stores it in a register and pops the second value from the stack and checks if the pop operation was successful and stores it in a register. Checks whether the flag is positive or negative and sets the flag. Holds a register that holds one of the operands of the operation, summing it with itself by the number of other operands of the operation. Sets the sign of the result and then checks the result in the range -999 to 999 and returns the result if successful.
---
2. Solution:
```assembly
.Orig x3000
        TRAP x23
        ADD R1 R0 #0
        AND R1 R1 x000F     ;convert first input to decimal
        TRAP x23
        AND R0 R0 x000F     ;convert second input to decimal
        ADD R0 R1 R0
        LD  R1 ASCIIoffset  
        ADD R0 R1 R0        ;convert sum to ASCII
        TRAP x21
        TRAP x25
        
ASCIIoffset .FILL x0030
.END
```
---
3. Solution:
```assembly
.Orig x3000
        TRAP x23
        LD R2 MINUS_30
        LD R3 IS_HEX
        LD R5 HEX
        LD R6 NEG_HEX
        ADD R1 R0 R2
        ADD R4 R1 R3
        BRnz NEXT
        ADD R1 R1 R6
        
NEXT    TRAP x23
        ADD R0 R0 R2
        ADD R4 R0 R3
        BRnz NEXT2
        ADD R0 R0 R6
        
NEXT2   ADD R0 R1 R0
        ADD R4 R0 R3
        BRnz NEXT3
        ADD R0 R0 R5

NEXT3   LD R2 ASCIIoffset
        ADD R0 R0 R2
        TRAP x21
        TRAP x25

MINUS_30    .FILL xFFD0 ; - x0030        
ASCIIoffset .FILL x0030
IS_HEX      .FILL xFFF7 ; -9
HEX         .FILL #7
NEG_HEX     .FILL #-7
.END
```
