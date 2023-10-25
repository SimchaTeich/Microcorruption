# Halifax - 20 points
 
## The idea
Exploitation of the ability to extract the hash of any part I want,<br />
and in any size I want, from a memory that is not accessible.

## The way

### Black box test:
Let's look at the challenge instructions:

<img src="./24.1.png" width="80%"></img>
* we can't unlock the door by 0x7f interrupt anymore
* ther is a mistake here. it's 0x42.
* payload is very clearly. we can divide it to 3 parts:
    * dest - `8000`
    * size of code - `02`
    * code - `3041`

Let's also look at how it looks in the user window:

<img src="./24.2.png" width="80%"></img>
* red - it's a mistake. 0x42, not 0x41.

Let's explore the code.

### Explore the code:

Below is the main function of the program.<br />
Immediately afterwards a reconstruction of this code will appear in c.<br />
I did this to make it easier to explain, so read it carefully as well.

<img src="./24.3.png"></img>
<img src="./24.4.png"></img>

```c
#define MAX_SIZE 0x400
#define void func(void);

typedef (short)*(0x2400) dest;
typedef (char)*(0x2402) size;

void main()
{
    char stack_memory[0x20];

    puts("Welcome to the test program loader.");

    puts("Enablibg hardened mode");
    INT(0x40); // interrupt for harden mode.

    puts("Verifying 0x7f interrupt disable");
    INT(0x7f); // unlock-the-door interrupt doesn't work anymore.
    puts("0x7f interrupt disabled, key stored in internal SRAM");

    // there is a mistake here. the interrupt number is 0x42, not 0x41.
    puts("unlock by providing the 16 byte key to 0x42 interrupt");

    // extract result of sha256(SRAM) into the top of the stack.
    memset(stack_memory, 0, 0x20);
    puts("Internul SRAM Hash:");
    sha256_internal(0, 0x1000, stack_memory);

    /*
        Some loop that prints the result of the abouve function.
        From the stack, to the screen.
    */

    while(1)
    {
        puts("Please enter debug payload.");
        memset(0x2400, 0, MAX_SIZE);
        getsn(0x2400, MAX_SIZE - 1);

        if(size < 2)
        {
            puts("Invalid payload length");
            continue;
        }

        puts("Executing debug payload");
        memcpy(dest, 0x2403, size);  // copy user program into dest
        ((func*)dest)();             // execute the user program
    }
}
```

So we can run any code we want, but we won't be able to open the door with the 0x7f interface like we did in all the previous challenges.<br />

Let's pay attention to the `sha256_internal` function:
* param1 - offset from the beginning of the SRAM.
* param2 - size of bytes from the offset to take
* param3 - Destination address for the memory accessible to us, to which the result of the function will be written.
    * The result is the sha256 of the bytes we selected from the SRAM.

So the password is somewhere in a memory area whose size is 0x1000 bytes and which is not accessible to us.<br />

what can we do?

### How to exploit:
Let's pay attention again to the `sha256_internal` function.<br />
It does not limit the amount of bytes it hashes.<br />
That means you can also extract each byte there separately.<br />
Therefore, as a start only, we will try to extract all the SRAM and hash it.<br />
The following code extracts 0x400 bytes, where each byte becomes a 16-byte hash and is stored in memory.

```asm
; save registers
push	r8
push	r9
push	r10
push	r11
push	r13
push	r14
push	r15

; init values.
mov	#0x400, r11 ; i = 0x400
mov	#0x6000, r8 ; init destination
mov	#0x1, r9    ; init size
clr	r10         ; init offset from start of SRAM.

; while loop
mov	r8, r13
mov	r9, r14
mov	r10, r15
call	#0x45b6 ; sha256(offset, size, dest)
add	#0x20, r8   ; the next dest address, so as not to overrun the previous result.
inc	r10         ; offset++
dec	r11         ; i--
jnz	$-0x14      ; while(i>0) 

; restoring registers
pop	r15
pop	r14
pop	r13
pop	r11
pop	r10
pop	r9
pop	r8
ret
```



## The cracking input (as bytes)
```

```