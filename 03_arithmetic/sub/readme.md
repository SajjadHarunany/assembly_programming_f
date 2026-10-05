# Subtraction

The assembly programs were built as 32-bit Linux binaries using the NASM assembler and the ld linker. Both executed without errors and returned an exit code of `0`.

## sub1.asm

This program performs an 8-bit subtraction:

`50 - 80 = -30`

Two's complement: `0xE2`

### Flags after SUB

* **CF = 1 (Set):** Carry Flag in subtraction indicates an unsigned borrow. Since 50 is smaller than 80, the CPU must borrow when calculating `50 - 80`. Therefore, CF is set.

* **OF = 0 (Cleared):** Signed overflow would occur if the mathematical result could not be represented as a signed 8-bit value. The result is `-30`, which is within the range `-128` to `127`, so no signed overflow occurs.

* **SF = 1 (Set):** The most significant bit of `0xE2` is 1. In signed 8-bit two's complement representation, this indicates a negative result.

* **ZF = 0 (Cleared):** The result is `-30`, not zero, so the Zero Flag is cleared.

* **AF = 0 (Cleared):** The lower nibbles are `0x2` and `0x0`. Subtracting the lower nibble does not require a borrow from bit 4, so AF is cleared.

* **PF = 1 (Set):** The result byte `0xE2` is `11100010`, containing four 1-bits. Since four is even, the result has even parity and PF is set.

## sub2.asm

The program performs 16-bit subtraction:

`1000 - 2000 = -1000`

The result is represented in 16-bit two's complement as:

`0xFC18`

### Flags after SUB

* **CF = 1 (Set):** Since 1000 is smaller than 2000 when treated as unsigned values, the subtraction requires a borrow. Therefore, CF is set.

* **OF = 0 (Cleared):** The signed result is `-1000`, which is safely within the signed 16-bit range of `-32768` to `32767`. Therefore, signed overflow does not occur.

* **SF = 1 (Set):** Bit 15 of `0xFC18` is 1. This indicates that the two's complement result represents a negative number.

* **ZF = 0 (Cleared):** The result is `-1000`, not zero, so ZF is cleared.

* **AF = 0 (Cleared):** The lower nibbles are `0x0` and `0x0`, so the subtraction does not require a borrow from bit 4.

* **PF = 1 (Set):** The low byte of the result is `0x18`, which is `00011000` in binary. It contains two 1-bits, giving even parity, so PF is set.

## Conclusion

Both subtraction operations produce negative results and therefore have SF set. CF is also set because, when considered as unsigned arithmetic, the subtracted value is larger than the original value and a borrow is required. Neither operation produces signed overflow because both negative results are within their respective signed ranges.
