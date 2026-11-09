# Assembly Stack Analysis: 05.asm

## 1. Stack Execution Walkthrough

The program initializes a full 64-bit immediate hexadecimal value into `RAX` and a smaller 16-bit immediate value into `RBX`. It pushes both 64-bit padded configurations onto the stack and restores them using Last-In, First-Out (LIFO) sorting parameters before a clean exit.

```nasm
section .text
    global _start

_start:
    mov rax, 0x1122334455667788 ; RAX = 0x1122334455667788
    mov rbx, 0x1234             ; RBX = 0x1234

    push rax                    ; Push 64-bit RAX value onto stack
    push rbx                    ; Push 64-bit extended RBX value onto stack
    nop                         ; Peak stack depth reached

    pop rbx                     ; Pop top value (0x1234) back into RBX
    pop rax                     ; Pop next value (0x1122334455667788) back into RAX

    mov rax, 60                 ; sys_exit syscall number
    xor rdi, rdi                ; Exit status 0
    syscall
```

---

## 2. Stack Mechanism & Pointer Adjustments

In the x86_64 architecture, the stack grows downwards toward lower memory addresses. Even though `RBX` holds a smaller 16-bit numeric value (`0x1234`), a standard 64-bit push expands and zero-pads the data to occupy a full **8-byte** word slot on the stack.

*   **Initial State:** `RSP` starts at a baseline address (denoted as `V`). `RAX` and `RBX` contain their initialization data.
*   **`push rax`:** The CPU decrements `RSP` by 8 (`RSP = V - 8`) and copies the 8 bytes of `0x1122334455667788` into memory at that location.
*   **`push rbx`:** The CPU decrements `RSP` by another 8 (`RSP = V - 16`) and copies the 8-byte padded representation of `0x0000000000001234` to the top of the stack.

### Peak Stack State (At `nop`)
```text
Memory Address   | Stored Value           | Description
-----------------------------------------------------------------------
V - 16  <-- RSP  | 0x0000000000001234     | Top of Stack (From RBX)
V - 8            | 0x1122334455667788     | Middle Element (From RAX)
V                | [OS Data]              | Initial Stack Base Frame
```

*   **`pop rbx`:** The CPU retrieves the 8-byte value at `V - 16` (`0x1234`), copies it into `RBX`, and increments `RSP` by 8 (`RSP = V - 8`).
*   **`pop rax`:** The CPU retrieves the 8-byte value at `V - 8` (`0x1122334455667788`), copies it into `RAX`, and increments `RSP` by 8 (`RSP = V`).
*   **Final State:** The data is safely restored to its original register locations because the pops matched the exact reverse order of the pushes. `RSP` cleanly returns to its baseline frame `V`.

---

## 3. Program Observations

*   The program executed successfully and exited without error.
