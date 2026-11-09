# Assembly Stack Analysis: 02.asm

## 1. Stack Execution Walkthrough

The program uses the stack to coordinate a function call (`work`) and establish a standard stack frame (activation record) to allocate space for local variables before executing a clean program exit.

```nasm
section .text
    global _start

_start:
    call work               ; Push return address onto stack and jump to work
    mov rax, 60             ; sys_exit syscall number
    xor rdi, rdi            ; Exit status 0
    syscall

work:
    push rbp                ; Save old Base Pointer
    mov rbp, rsp            ; Set up new Base Pointer frame
    sub rsp, 16             ; Allocate 16 bytes for local variables

    mov qword [rbp - 8], 100  ; Store 100 in first local slot
    mov qword [rbp - 16], 200 ; Store 200 in second local slot

    mov rax, [rbp - 8]      ; Load 100 into RAX
    mov rbx, [rbp - 16]     ; Load 200 into RBX
    nop                     ; Peak function stack configuration

    mov rsp, rbp            ; Deallocate local variables (Collapse stack)
    pop rbp                 ; Restore original Base Pointer
    ret                     ; Pop return address into RIP and return
```

---

## 2. Stack Mechanism & Pointer Adjustments

In x86_64, the stack grows downwards toward lower memory addresses. Standard word operations shift the **RSP (Stack Pointer)** register by **8 bytes**, while explicit subtraction modifies it by the requested immediate value.

*   **Initial State:** `RSP` starts at baseline address `V`. `RBP` holds original caller base pointer data.
*   **`call work`:** The CPU decrements `RSP` by 8 (`RSP = V - 8`) and pushes the 64-bit return address (pointing to `mov rax, 60`) into memory at `V - 8`.
*   **`push rbp`:** Inside `work`, `RSP` decrements by 8 (`RSP = V - 16`). The original value of `RBP` is stored at `V - 16`.
*   **`mov rbp, rsp`:** `RBP` is updated to match `RSP` (`RBP = V - 16`). This anchors the base of the current stack frame.
*   **`sub rsp, 16`:** `RSP` explicitly drops by 16 bytes (`RSP = V - 32`), carving out local variable space below `RBP`.

### Peak Function Stack State (At `nop`)
```text
Memory Address   | Stored Value       | Description
-----------------------------------------------------------------------
V - 32  <-- RSP  | 200 (64-bit qword) | Local Variable 2 [RBP - 16]
V - 24           | 100 (64-bit qword) | Local Variable 1 [RBP - 8]
V - 16  <-- RBP  | [Old RBP Value]    | Saved Frame Base Pointer
V - 8            | [Return Address]   | Code execution point for return
V                | [OS Data]          | Initial Stack Base Frame
```

*   **`mov rsp, rbp`:** The stack pointer is reset directly back to the base anchor (`RSP = V - 16`), automatically discarding the local stack variables.
*   **`pop rbp`:** The CPU reads the old pointer data from `V - 16`, restores it into `RBP`, and increments `RSP` by 8 (`RSP = V - 8`).
*   **`ret`:** The CPU pops the saved execution pointer address from `V - 8` back into the instruction register (`RIP`) and increments `RSP` by 8 (`RSP = V`).
*   **Final State:** The execution returns cleanly to `_start`, with `RSP` restored symmetrically back to its initial baseline value `V`.

---

## 3. Program Observations

*   The program executed successfully and exited without error.
