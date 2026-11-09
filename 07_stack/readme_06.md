# Assembly Stack Analysis: 06.asm

## 1. Stack Execution Walkthrough

The program is a 32-bit x86 (`BITS 32`) assembly routine that initializes two hexadecimal values into 32-bit registers, pushes them onto the stack, and pops them back in reverse order to restore their initial states before executing an `int 0x80` system exit call.

```nasm
BITS 32
section .text
    global _start

_start:
    mov eax, 0x11223344     ; EAX = 0x11223344
    mov ebx, 0x1234         ; EBX = 0x1234

    push eax                ; Push 32-bit EAX value onto stack
    push ebx                ; Push 32-bit extended EBX value onto stack
    nop                     ; Peak stack depth reached

    pop ebx                 ; Pop top value (0x1234) into EBX
    pop eax                 ; Pop next value (0x11223344) into EAX

    mov eax, 1              ; sys_exit syscall number for i386
    xor ebx, ebx            ; Exit status 0
    int 0x80                ; Invoke Linux kernel interrupt
```

---

## 2. Stack Mechanism & Pointer Adjustments

In the 32-bit x86 architecture, the stack grows downwards toward lower memory addresses. Every standard push or pop operation modifies the **ESP (Stack Pointer)** register by exactly **4 bytes** (32 bits).

*   **Initial State:** `ESP` starts at a baseline address (denoted as `V`). `EAX` and `EBX` contain their initialization data.
*   **`push eax`:** The CPU decrements `ESP` by 4 (`ESP = V - 4`) and writes the 4-byte value `0x11223344` into memory at that location.
*   **`push ebx`:** The CPU decrements `ESP` by another 4 (`ESP = V - 8`) and writes the 4-byte zero-padded value `0x00001234` to the top of the stack.

### Peak Stack State (At `nop`)
```text
Memory Address   | Stored Value       | Description
-----------------------------------------------------------------
V - 8   <-- ESP  | 0x00001234         | Top of Stack (From EBX)
V - 4            | 0x11223344         | Middle Element (From EAX)
V                | [OS Data]          | Initial Stack Base Frame
```

*   **`pop ebx`:** The CPU reads the 4-byte value at `V - 8`, copies `0x00001234` into `EBX`, and increments `ESP` by 4 (`ESP = V - 4`).
*   **`pop eax`:** The CPU reads the 4-byte value at `V - 4`, copies `0x11223344` into `EAX`, and increments `ESP` by 4 (`ESP = V`).
*   **Final State:** The values are successfully restored to their original registers because the `pop` order directly reversed the `push` order. The stack pointer `ESP` safely returns to its baseline frame `V`.

---

## 3. Program Observations

*   The program executed successfully and exited without error.
