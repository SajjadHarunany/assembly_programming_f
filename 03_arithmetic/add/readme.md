# Addition

These assembly programs were built as 32-bit Linux binaries using the NASM assembler and the ld linker. Both executed without errors and returned an exit code of `0`.

## add1.asm

The program performs an 8-bit unsigned addition:

Decimal: `120 + 10 = 130`

Hexadecimal: `0x78 + 0x0A = 0x82`

The result is stored in `AL`.

### Flags after ADD

* **CF = 0 (Cleared):** Carry Flag checks for an unsigned carry out of the most significant bit. The largest unsigned 8-bit value is 255, and 130 fits within this range. Therefore, there is no carry beyond bit 7.

* **OF = 1 (Set):** Overflow Flag is concerned with signed arithmetic. Both operands, 120 and 10, are positive signed 8-bit values, but their mathematical result, 130, is greater than the maximum signed 8-bit value of 127. The bit pattern `0x82` therefore represents `-126` as a signed 8-bit value, indicating signed overflow.

* **SF = 1 (Set):** The Sign Flag copies the most significant bit of the result. `0x82` is `10000010` in binary, so bit 7 is 1. The result is therefore interpreted as negative in signed representation.

* **ZF = 0 (Cleared):** Zero Flag is set only when the result is exactly zero. Since the result is 130, ZF remains cleared.

* **AF = 1 (Set):** Auxiliary Carry detects a carry from bit 3 to bit 4. The lower nibbles are `0x8` and `0xA`. Their sum is `0x12`, which requires a carry into the upper nibble, so AF is set.

* **PF = 1 (Set):** Parity Flag considers the low byte of the result. `0x82` is `10000010`, which contains two 1-bits. Since the number of 1-bits is even, PF is set.

## add2.asm

The program adds two 16-bit values:

`32000 + 500 = 32500`

In hexadecimal:

`0x7D00 + 0x01F4 = 0x7EF4`

The result is stored in `AX`.

### Flags after ADD

* **CF = 0 (Cleared):** The unsigned result, 32500, is below the maximum 16-bit unsigned value of 65535. Therefore, the addition does not produce a carry beyond bit 15.

* **OF = 0 (Cleared):** Both operands are positive and their signed result, 32500, is still within the 16-bit signed range of `-32768` to `32767`. Therefore, signed overflow does not occur.

* **SF = 0 (Cleared):** The most significant bit of `0x7EF4` is 0. Therefore, the result is positive when interpreted as a signed 16-bit value.

* **ZF = 0 (Cleared):** The result is 32500, not zero, so ZF is cleared.

* **AF = 0 (Cleared):** The low nibbles are `0x0` and `0x4`. Their sum does not exceed `0xF`, so there is no carry from bit 3 to bit 4.

* **PF = 0 (Cleared):** PF checks the low byte, which is `0xF4` (`11110100`). It contains five 1-bits, an odd number, so the parity flag is cleared.

## Conclusion

The two additions demonstrate the difference between unsigned and signed overflow. `add1` has no unsigned carry because 130 fits in 8 bits, but it has signed overflow because 130 cannot be represented as a positive signed 8-bit value. `add2` fits in both unsigned and signed 16-bit ranges, so neither CF nor OF is set.
