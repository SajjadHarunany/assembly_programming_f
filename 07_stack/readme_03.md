# Assembly Stack Analysis: 03.asm

## 1. Stack Execution Walkthrough

The program loads an initial value into the `RAX` register and calls a subroutine (`calculate`). It utilizes the stack exclusively to manage the control flow of the function call and return before concluding with a clean process exit.

```nasm
section .text
    global _start

_start:
    mov rax, 10             ; RAX = 10
    call calculate          ; Push the address of 'after_call' onto stack and jump

after_call:
    ; RAX should now be 15
    nop                     ; Post-subroutine synchronization boundary

    mov rax, 60             ; sys_exit syscall number
    xor rdi, rdi            ; Exit status 0
    syscall

calculate:
    add rax, 5              ; Modify RAX (10 + 5 = 15)
    ret                     ; Pop return address from stack into RIP
```

---

## 2. Stack Mechanism & Pointer Adjustments

In the x86_64 architecture, the stack pointer register **RSP** tracks the top of the stack and decrements automatically when data is pushed or functions are called.

*   **Initial State:** `RSP` sits at the initial operating system execution boundary `V`. `RAX = 10`.
*   **`call calculate`:** The CPU automatically decrements `RSP` by 8 bytes (`RSP = V - 8`). It then copies the 64-bit address of the next sequential instruction (`after_call`) directly into memory at the location pointed to by `RSP`.

### Peak Stack State (Inside `calculate`)
```text
Memory Address   | Stored Value       | Description
-----------------------------------------------------------------
V - 8   <-- RSP  | [after_call Addr]  | Saved Instruction Pointer Link
V                | [OS Data]          | Initial Stack Base Frame
```

*   **Subroutine Operation:** While inside `calculate`, the program modifies the `RAX` register arithmetic state directly, without interacting with or pushing further items onto the stack.
*   **`ret`:** The CPU reads the 64-bit return address from the memory location currently tracked by `RSP` (`V - 8`) and loads it into the Instruction Pointer (`RIP`). Simultaneously, it increments `RSP` by 8 bytes (`RSP = V`).
*   **Final State:** Program execution successfully resumes at `after_call`. The stack pointer `RSP` perfectly resets back to its initial baseline value `V`, maintaining frame safety.

---

## 3. Program Observations

*   The program executed successfully and exited without error.
