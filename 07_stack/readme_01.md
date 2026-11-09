# Assembly Stack Analysis: 01.asm

## 1. Stack Execution Walkthrough

The program initializes two immediate values into 64-bit registers, utilizes a Last-In, First-Out (LIFO) stack sequence to swap their values into distinct destination registers, and terminates cleanly.

```nasm
section .text
    global _start

_start:
    mov rax, 100            ; RAX = 100
    mov rbx, 200            ; RBX = 200

    push rax                ; Push 100 onto stack
    push rbx                ; Push 200 onto stack
    nop                     ; Peak stack depth reached

    pop rcx                 ; Pop top value (200) into RCX
    pop rdx                 ; Pop next value (100) into RDX
    nop                     ; Stack cleared

    mov rax, 60             ; sys_exit syscall number
    xor rdi, rdi            ; Exit status 0
    syscall
```

---

## 2. Stack Mechanism & Pointer Adjustments

In the x86_64 architecture, the stack grows downwards toward lower memory addresses. Every operation modifies the **RSP (Stack Pointer)** register by exactly **8 bytes**.

*   **Initial State:** `RSP` starts at a baseline address (denoted as `V`). `RAX = 100` and `RBX = 200`.
*   **`push rax`:** The CPU decrements `RSP` by 8 (`RSP = V - 8`) and writes the value `100` to that memory address.
*   **`push rbx`:** The CPU decrements `RSP` by another 8 (`RSP = V - 16`) and writes the value `200` to the new top of the stack.

### Peak Stack State (At first `nop`)
```text
Memory Address   | Stored Value | Description
-----------------------------------------------------------------
V - 16  <-- RSP  | 200          | Top of Stack (From RBX)
V - 8            | 100          | Middle Element (From RAX)
V                | [OS Data]    | Initial Stack Base Frame
```

*   **`pop rcx`:** The CPU reads the value at the top of the stack (`V - 16`), copies `200` into `RCX`, and increments `RSP` by 8 (`RSP = V - 8`).
*   **`pop rdx`:** The CPU reads the value at the new top of the stack (`V - 8`), copies `100` into `RDX`, and increments `RSP` by 8 (`RSP = V`).
*   **Final State:** `RCX` holds `200` and `RDX` holds `100`. The stack pointer `RSP` cleanly returns to its baseline frame `V`, maintaining proper stack symmetry before exiting.

---

## 3. Program Observations

*   The program executed successfully and exited without error.
