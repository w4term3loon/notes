```ascii
/------------------\  lower
|                  |  memory
|       Text       |  addresses
|                  |
|------------------|
|   (Initialized)  |
|        Data      |
|  (Uninitialized) |
|------------------|
|                  |
|       Stack      |  higher
|                  |  memory
\------------------/  addresses
```

```c
void function(int a, int b, int c) {
   char buffer1[5];
   char buffer2[10];
}

void main(void) {
  function(1,2,3);
}
```

To understand what the program does to call `function()` we compile it with `gcc` using the `-S` switch to generate assembly code output:

```asm
pushl $3
pushl $2
pushl $1
call function
```

This pushes the 3 arguments to function backwards into the stack, and calls `function()`.  The instruction `call` will push the instruction pointer (IP) onto the stack.  We'll call the saved IP the return address `RET`.  The first thing done in function is the procedure prolog:

```asm
pushl %ebp
movl %esp,%ebp
subl $20,%esp
```

This pushes `EBP`, the frame pointer, onto the stack.  It then copies the current `SP` onto `EBP`, making it the new `FP` pointer.  We'll call the saved `EBP` pointer `SFP` (saved frame pointer).  It then allocates space for the local variables by subtracting their size from `SP`.

We must remember that memory can only be addressed in multiples of the word size.  A word in our case is **4 bytes**, or **32 bits**.  So our **5 byte** buffer is really going to take **8 bytes** (2 words) of memory, and our **10 byte** buffer is going to take **12 bytes** (3 words) of memory.  That is why `SP` is being subtracted by **20**.  With that in mind our stack looks like this when
`function()` is called (each space represents a byte):

```ascii
bottom of                                                            top of
memory                                                               memory
 	         buffer2       buffer1   SFP   RET   a     b     c
 <------ SP [            ][        ][    ][    ][    ][    ][    ]
	   
top of                                                            bottom of
stack                                                                 stack
```

**Note:** If the total size of local variables is **less than 16 bytes**, the compiler pads the stack allocation to **16 bytes** because the **x86-64 System V ABI** mandates that the stack pointer `RSP` must be a aligned to a multiple of 16 before making a function call.