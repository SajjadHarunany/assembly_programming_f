# Assembly Stack Analysis: 04.asm

## 1. Stack Execution Walkthrough

The program is a 32-bit x86 (`BITS 32`) assembly routine that uses the stack to coordinate a function call (`work`) and establish a 32-bit stack frame. It allocates local variable slots on the stack using the 32-bit Base Pointer (`EBP`) and Stack Pointer (`ESP`) before executing an `int 0x80` system call.

```nasm
BITS 32
section .text
    global _start

_start:
    call work               ; Push 32-bit return address onto stack and jump to work

    ; Linux i386 exit syscall
    mov eax, 1              ; sys_exit syscall number
    xor ebx, ebx            ; Exit status 0
    int 0x80                ; Invoke Linux kernel interrupt

work:
    push ebp                ; Save old 32-bit Base Pointer
    mov ebp, esp            ; Set up new Base Pointer frame anchor
    sub esp, 8              ; Allocate 8 bytes for local variables

    mov dword [ebp - 4], 100  ; Store 100 in the first local stack slot
    mov dword [ebp - 8], 200  ; Store 200 in the second local stack slot

    mov eax, [ebp - 4]      ; Load 100 into EAX
    mov ebx, [ebp - 8]      ; Load 200 into EBX
    nop                     ; Peak function stack state configuration

    mov esp, ebp            ; Deallocate local variables (Collapse stack)
    pop ebp                 ; Restore original Base Pointer
    ret                     ; Pop return address into EIP and return
```

---

## 2. Stack Mechanism & Pointer Adjustments

In the 32-bit x86 architecture, the stack grows downwards toward lower memory addresses. Standard word operations shift the **ESP (Stack Pointer)** register by **4 bytes** (32 bits), while explicit subtraction modifies it by the requested immediate byte amount.

*   **Initial State:** `ESP` starts at a baseline address (denoted as `V`). `EBP` holds the caller's frame pointer.
*   **`call work`:** The CPU decrements `ESP` by 4 (`ESP = V - 4`) and pushes the 32-bit return address (pointing to `mov eax, 1`) into memory at `V - 4`.
*   **`push ebp`:** Inside `work`, `ESP` decrements by 4 (`ESP = V - 8`). The old value of `EBP` is stored at `V - 8`.
*   **`mov ebp, esp`:** `EBP` is updated to match `ESP` (`EBP = V - 8`), locking the base of this execution frame.
*   **`sub esp, 8`:** `ESP` is explicitly subtracted by 8 (`ESP = V - 16`), reserving two 4-byte slots for local stack data.

### Peak Function Stack State (At `nop`)
```text
Memory Address   | Stored Value       | Description
-----------------------------------------------------------------------
V - 16  <-- ESP  | 200 (32-bit dword) | Local Variable 2 [EBP - 8]
V - 12           | 100 (32-bit dword) | Local Variable 1 [EBP - 4]
V - 8   <-- EBP  | [Old EBP Value]    | Saved Frame Base Pointer
V - 4            | [Return Address]   | Code execution point for return
V                | [OS Data]          | Initial Stack Base Frame
```

*   **`mov esp, ebp`:** The stack pointer is restored directly to the frame anchor (`ESP = V - 8`), clearing the local variables instantly.
*   **`pop ebp`:** The CPU reads the old frame value from `V - 8`, loads it into `EBP`, and increments `ESP` by 4 (`ESP = V - 4`).
*   **`ret`:** The CPU pops the 32-bit instruction destination pointer from `V - 4` back into `EIP` and increments `ESP` by 4 (`ESP = V`).
*   **Final State:** Execution flows back to `_start` with `ESP` returning symmetrically to its clean baseline point `V`.

---

## 3. Program Observations

*   The program executed successfully and exited without error.
