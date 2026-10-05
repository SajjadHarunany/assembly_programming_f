# Division

The assembly programs were built as 32-bit Linux binaries using the NASM assembler and the ld linker. Both executed without errors and returned an exit code of `0`.

Crucially, unlike ADD, SUB and MUL, the x86 DIV instruction leaves the arithmetic status flags undefined. Consequently, CF, OF, SF, ZF, AF and PF cannot be relied upon after a division operation.

Any flag values displayed in a debugger like GDB are merely remnants of previous instructions and do not reflect the outcome of the division.

## div1.asm

The program performs unsigned 8-bit division.

The operation uses the AX register for the dividend and the BL register for the divisor:

`100 ÷ 7 = 14 remainder 2`

After the division:

* `AL = 14` — quotient
* `AH = 2` — remainder

### Flags after DIV

* **CF = Undefined**
* **OF = Undefined**
* **SF = Undefined**
* **ZF = Undefined**
* **AF = Undefined**
* **PF = Undefined**

Because the DIV instruction does not alter these flags in a meaningful way, their states cannot be used to determine if the quotient is zero, positive, negative, or even/odd in parity.

## div2.asm

The program performs unsigned 16-bit division.

The dividend is split across the DX:AX register pair, while the divisor is stored in BX:

`50000 ÷ 300 = 166 remainder 200`

After the division:

* `AX = 166` — quotient
* `DX = 200` — remainder

### Flags after DIV

* **CF = Undefined**
* **OF = Undefined**
* **SF = Undefined**
* **ZF = Undefined**
* **AF = Undefined**
* **PF = Undefined**

As with the 8-bit version, these arithmetic flags are not defined by the hardware during division. Any values seen in GDB are leftover states from preceding code.

## Conclusion

Both division programs produce the expected quotient and remainder. `div1` produces a quotient of 14 and remainder of 2, while `div2` produces a quotient of 166 and remainder of 200. However, none of the six arithmetic status flags can be reliably classified as set or cleared after `DIV`, because the x86 instruction leaves their results undefined.
