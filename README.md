# ARM Notes


### DISCLAIMER
Information initially from [David Thomas](http://www.davespace.co.uk/arm/introduction-to-arm/).
If you are looking for an actual walkthrough, check there rather than here. These are literally just notes.

**Alternative Sources**:
Other sources used include the following (updated as I go):
- [ARM website's development tools manual](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0204j/Cihcdbca.html)
- [ARM Assembler User Guide on keil.com](http://www.keil.com/support/man/docs/armasm/armasm_dom1361289913099.htm)
- [ARM Community forums on Condition Codes](https://community.arm.com/processors/b/blog/posts/condition-codes-1-condition-flags-and-codes)

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

**Breakdown of Terms**:
    
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


### Movement

Command Structure: 
  
```
<OP>{cond}{S} Rd, Rn
```
  
**Instructions**:

--- 
**`MOV Rd, Rn` - move**
- move Rn into Rd
---  
**`MVN Rd, Rn` - Move NOT**
- Takes the bitwise not of Rn and puts the result into Rd
---
  
> **Examples**:
> 
> **`MOV r0, #42`**
> - A move literal, moves the constant value 42 into register r0
> --- 
> **`MOV r2, r3`**
> - A register move, moves the CONTENTS of register r3 into register r2
> ---
> **`MOV r0, r0`**
> - A NOP instruction as it puts the contents of register r0 into register r0
>   accomplishing nothing.  
> ---  
> **`MVN r1, r0`**
> - A NOT move on registers, moves the bitwise-not of the contents of register r0 into
>   register r1
>   - e.g. Above we have `MOV r0, #42` where 42 is loaded into r0.
>     - The bitwise-not (compliment), ~42, is -43
>     - 42 = `00101010`, ~42 = `11010101`
>       - Solve for two's compliment: convert 1s to 0s then add 1
>         - `11010101` -> `00101010` -> `00101011` = 43 (scalar) so -43
> 


### Arithmetic Operations

Command Structure:

```
<op>{cond}{S} Rd, Rn, Operand2
```

**Instructions**:

---
**`ADD Rd, Rn, Operand2` - Addition**
- Adds Operand2 to Rn and stores in Rd
- e.g. `ADD r1, r0, #45` -> `r1 := r0 + 45`
---
**`ADC Rd, Rn, Operand2` - Addition with carry**
- Same thing as ADD but with carry this time
- e.g. `ADC r1, r0, #45` -> `r1 := r0 + 45 + carry`
---
**`SUB Rd, Rn, Operand2` - Subtraction**
- Subtracts Operand2 from Rn and stores in Rd
- e.g. `SUB r0, r1, r4` -> `r0 := r1 - r4`
---
**`SBC Rd, Rn, Operand2` - Subtract with carry**
- e.g. `SBC r0, r1, r4` -> `r0 := r1 - r4 - NOT(carry)`
---
**`RSB Rd, Rn, Operand2` - Reverse subtract**
- literally SUB with order switched
- e.g. `RSB r0, r1, 45` -> `r0 := 45 - r1`
  - A typical SUB would have been `r0 := r1 - 45` instead
---
**`RSC Rd, Rn, Operand2` - Reverse subtract with carry**
- e.g. `RSC r0, r4, #0xFF` -> `r0 := 0xFF - r4 - NOT(carry)`
---

> **Additional Notes**:
> * No built-in division function
>   * Typically shifts or some compiler function will be used for these
> * Multiplication works differently, needs own section


### Logical Operations

Command Structure:

```
<op>{cond}{S} Rd, Rn, Operand2
```

**Instructions**:

---
**`AND Rd, Rn, Operand2` - Logical AND**
- Stores the bitwise AND of Rn and Operand2 in Rd
- e.g. `AND r0, r1, #1337` -> `r0 := r1 & 1337`
  - 1337 is 0x539 (hex)
  - Also note this is likely a bad example. In Thumb immediates (constants)
    only go up to 255.
---
**`EOR Rd, Rn, Operand2` - Exclusive OR (XOR)**
- Performs a bitwise XOR on Rn and Operand2 storing the result in Rd
- e.g. `EOR r1, #42, #0xFFFFFFFF` -> `r1 := 0xFFFFFFFF ^ 42`
  - `0xFFFFFFFF` represents -1 (2's compliment) so this is `42 ^ -1 = -43`
---
**`ORR Rd, Rn, Operand2` - Logical OR**
- Store the bitwise OR of Rn and Operand2 into Rd
- e.g. `ORR r8, r4, r6` -> `r8 := r4 || r6`
- e.g. 2: `ORR r7, #6, #88` -> ` r7 := 6 || 88 = 94`
---
**`BIC Rd, Rn, Operand2` - Bitwise Clear**
- Stores result of Rn AND NOT Operand2 into Rd
- e.g. `BIC r1, r4, 10` -> `r1 := r4 & (~10)`
---

> **NOTE:**    
> I am unsure how ARM aligns constants so its hard for me to tell how
> many of these logical operations will actually work. For example, if
> you ask Python 2 what ~10 is, it will tell you -11 because it aligns
> it to a full byte (e.g. `10` = `0000 1010`) so when inverting it gets
> `10 = 0000 1010, ~10 = 1111 0101` which in accounting for two's compliment
> is `- 0000 1010 + 01 = - 0000 1011 = - 11`. If on the other hand you do
> _not_ byte-align the value, the result is instead `5` (`1010` -> `0101`).


### Comparisons

Command Structure:

```
<op>{cond} Rn, Operand2
```

Comparison instructions perform operations but instead of storing the operation
result in a register, they set the flags based upon the operation. The specific

**Instructions**:

---
**`CMP Rn, Operand2` - Compare**
- Sets flags to result of Rn - Operand2
- Same as `SUBS` without storing the result
- e.g. `CMP r0, #42` stores result of `r0 - 42` (compares them)
  - if r0 is bigger, result will be positive, else 42 is bigger
> **Flags affected:** `N`,`V`,`C`,`Z`  
> **Flags not affected:** None  
---
**`CMN Rn, Operand2` - Compare Negative**
- Does the same thing as CMP but with the negative of Operand2
- Same as `ADDS` without storing the result
- e.g. `CMN r0, #42` -> `r0 - (-42) = r0 + 42`
  - Subtraction of a negative is addition
> **Flags affected:** `N`, `V`, `C`, `Z`  
> **Flags not affected:** None  
---
**`TST Rn, Operand2` - Bitwise test**
- Does a bitwise AND between Rn and Operand2 discarding the result but setting flags
- Same as `ANDS` without storing the result
- e.g. `TST r0, #250` -> Sets flags for `r0 & 250`
- Updates the N and Z flags according to the results 
- The C flag can update during calculation of Operand2
  - I'm not too sure what this means
- `TST` is useful for checking against bitmasks, this is what makes it different
  from `TEQ`. Using AND, TST determines if specific bits are set.
> **Flags affected:** `N`,`Z`, possibly `C`  
> **Flags not affected:** `V`  
---
**`TEQ Rn, Operand2` - Bitwise equivalence test**
- Sets flags for result of Rn EOR Operand2 (The xor of them)
- Basically checks that Rn == Operand2
- Same as `EORS` without storing the result
- e.g. `TEQ r0, r2` -> stores flags for `r0 XOR r2`
- result is zero if values are equal.
- does not affect the `V` or `C` flags (`CMP` does)
  - Updates the `N` and `Z` flags
> **Flags affected:** `N`, `Z`  
> **Flags not affected:** `V`, `C`  
---

## About the Barrel Shifter

The barrel shifter works on Operand2 and can provide up to 5 types of shifts
and rotations.

**Shifts Provided by Barrel Shifter**:

---
**`LSL` - Logical Shift Left**
- Equivalent to << in C and other high level languages
- Shifts the bits left
- e.g. Given `0000 0000 0000 0000 0000 0000 0000 1111 << 4` (LSL 4)
  - Result shifts everything to the left by 4 so we get:
    `0000 0000 0000 0000 0000 0000 1111 0000`
    - Note the group of ones shifted to the left by 4. The space is filled
      with zeros
- This is the same as an unsigned multiplication by a power of 2
  - In the above example the shift by 4 is actually multiplication by 2^4 (16)
---
**`LSR` - Logical Shift Right**
- Equivalent of >> in C and other high level languages
- Shifts the bits right
- e.g. Given `0000 0000 0000 0000 0000 0000 1111 0000 >> 4` (LSR 4)
  - Result shifts everything to the right by 4 (left filling with zeros)
    so we get `0000 0000 0000 0000 0000 0000 0000 1111`
- This is the same as unsigned division by a power of 2
  - The above example just divides the original value by 2^4 (16)
---
**`ASR` - Arithmetic Shift Right**
- This is similar to a Logical Shift Right except that it preserves the sign
  - `LSR` fills with `0`'s from the left, this keeps the sign
- e.g. Consider the value `1000 0000 0000 0000 0000 0000 0000 0000`
  shifted 3 with `ASR`
  - The result would be `1111 0000 0000 0000 0000 0000 0000 0000`
    - Notice the leftmost nibble changed from `1000` to `1111`
      - This is because the shift right fills with the same sign
        bit, 1's in this case.
- e.g. Using a positive byte as an example: `0100 0000` A>> 3
  would instead be `0000 1000`. 
  - It fills with 0s because it is positive.
- This is a signed division by a power of 2
---
**`ROR` - Rotate Right**
- Like a `LSR` but instead of the bits just "disappearing" off of the
  rightmost end, they rotate to the other end.
- e.g. Using just a byte, `0000 1010` rotated by 3 will become
  `0100 0001`
  - Note the `010` wrapped around to the leftmost bits.
---
**`RRX` - Rotate Right Extended**
- Rotate with wrap-around through carry bit
- Instead of strictly wrapping around it uses the previous carry bit
- e.g. `C 1000 0000 0000 0000 0000 0000 0000 00001` has an original carry bit C
  - When `RRX` is performed, the old C is placed onto the leftmost bit and then the
    rightmost bit that is "shifted off" is placed into the carry bit C.
  - Becomes `1 C100 0000 0000 0000 0000 0000 0000 0000` with `RRX`
    - Would be `1100 0000 0000 0000 0000 0000 0000 0000` if using `LSR`
    - C could be 1 or 0, unknown at start?
---

> **Further Notes:**  
> Right shifting of negative signed quantities is impementation defined behavior in C
> meaning the compiler chooses whether it will use `LSR` or `ASR`. Most typically the
> compiler will use `ASR` but it is important to note that this is **not guaranteed**.
>  
> Furthurmore, a handful of instructions such as `MUL`, `CLZ`, and `QADD` cannot use
> the barrelshifter. More on this probably where they are mentioned.


## Operand2

Operand2 can take three different forms:

- **immediate value**
  - 8-bit
  - rotated right by even number of places
- **register shifted by value**
  - 5-bit, unsigned integer shift
- **register shifted by register**
  - Uses the bottom 8-bits of a register

> **Examples:**  
>   
> **`MOV r0, #42`**  
> - Moves the value 42 into r0
> ---
> **`MOV r3, r2, LSR #1`**  
> - Shifts `r2` by one bit then stores in `r3`
> ---
> **`RSB r10, r5, r14, ASR #14`**  
> - Shift value of `r14` right by `14` then subtract value of `r5` from it
>   - Shift uses sign extension
>   - stores result in `r10`
> ---
> **`SUB r11, r1, r3, ROR r5`**
> - Rotate `r3` right by the value of `r5`
> - Subtract `r3` from `r1` then store result in `r11`


## Immediates

Immediates in ARM have restrictions on their formation and storage.
- 32-bit values can't fit into 32-bit instruction word
- ARM data processing instructions have 12 bits for values in instruction word
  - Comprised of 4-bit rotation value, and 8-bit immediate value
    - the 4-bit rotation value is multiplied by 2 so we can only shift by even
      values in the range 0:30 (0, 2, 4, 6, 8, ... 30)
    - Immediate value has range 0:255
  - Assembler converts big values to rotated form, impossible values cause error
  - Assembler sometimes use `MVN` to form bitwise compliment instead of using `MOV`
    - e.g. `MOV r0, #FFFFFFFF` == `MVN r0, #0`




