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
