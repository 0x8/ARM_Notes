# ARM Notes


### DISCLAIMER
Information initially from [David Thomas](http://www.davespace.co.uk/arm/introduction-to-arm/).
If you are looking for an actual walkthrough, check there rather than here. These are literally just notes.


### Basics:

* ARM has 16 registers
  * each 32 bits wide
    * R0-R12 are general purpose
    * R13 - Stack Pointer register
    * R14 - Link Register
      * Holds return address 
    * R15 - Program Counter
      * Holds address of next instruction to exec (EIP/RIP analog)

* ARM also features special register: CSPR (Current Program Status Register)
  * Holds flags from math and logic ops

* ARM is a "load-store" arch
  * values MUST be loaded into registers before they can be used
    * No direct memory value operations


### Multi-Mode Arch:
* Two main modes: ARM and Thumb

* ARM Mode:
  * Instructions are 32 bits wide
  * Instructions MUST be word aligned
    * "Word" is 4-bytes
      * 32 bits
    * "Half-word" is 2-bytes
      * 16 bits
  * PC is in bits 2-31 where bits 0-1 are undefined
    *  e.g XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXZZ
      * Where X is a PC bit but Z is undefined

* Thumb Mode:
  * Instructions are 16 bits wide
  * Instructions halfword aligned
    * 2-byte aligned
  * PC is in bits 1-31 with bit 0 undefined
    * e.g. XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXZ
      * Where X is a PC bit but Z is undefined


### Instruction Syntax for ARM:
    
```
<operation>{cond}{flags} Rd,Rn,Operand2
```

*Breakdown of Terms*:
    
* `<operation>`:
  * A three-letter mnemonic (MOV, ADD, etc.)
* `{cond}`:
  * An optional two-letter condition code (EQ, CS, etc.)
* `{flags}`:
  * Optional added flag(s) (e.g. S)
* `Rd`:
  * Destination register
* `Rn`:
  * The FIRST source register
* `Operand2`:
  * Flexible second operand

> Notes:
> * Above is general format for logic and arithmetic instrs.
> * Curly braces are optional parts
> * leftmost register is the destination
> * Instrs. are single-cycle with exceptions of write to PC and 
>   register controlled shift (?)
> * Also worth looking into recent assembly form UAL
>   * "Unified Assembler Language"
>   * conditions can go after the flags
>     * e.g. `<op>{flags}{con} Rd,Rn,Operand2`

   
### Organization:

* In a three register instruction:
  * `<op> Rd,Rn,Rm`
  * e.g. ADD R0,R1,R2
    * R0 and R1 are used directly
    * R1 is "Operand1" (op1)
    * R2 put through "barrel shifter" and becomes "Operand2" (op2)
      *  barrel shifter that can rotate and shift values
 

```
     -----------                                    ----------
    | Register  |Rm >--------------->--------------|  Barrel  |
    | Bank      |                                  |  Shifter |
    |           |Rn >------V                        ----------
     -----------           |                        |  ^    |
      Rd                   |        V----------<----<  |    v
      ^                    |        |                 --------
      |                    V        V                |  CPSR  |
      |                   OP1      OP2                --------
      |                -------    -------              |   ^
      |                \      \  /      /<------<------<   |
      ^                 \      \/      /                   |
      |                  \     ALU    />-------->----------^
      |                   ------------
      |                       Result
      |                          |
      ^------------<-------------<

```

> NOTE:
> CPSR is a register but is shown seperate to illustrate usability
> by both Barrel Shifter and ALU when setting flags
