# Churchill - 30 points
 
## The idea
Exploitation of entering an input through an overly simple algorithm for correctness checking.

## The way

### Black box test:
Black box testing shows us that the program is in an infinite loop.<br />
And we are even provided with an example.<br />
Let's jump right into the code.

### Explore the code:

The program is simple, but too complicated to explain in words.<br />
Therefore, immediately after it will appear its "source code" that I wrote in c.

<img src="./21.1.png"></img>
<img src="./21.2.png"></img>

```c
#define MAX_SIZE 0x400

#define void func(void);

typedef (short)*(0x2420) dest; // dest is r11
typedef (char)*(0x2422) flag;  // flag is r9
typedef (char)*(0x2423) size;  // size is r10

void main()
{
    // allocate stack memory:
    char stackMemory[0x40]; // -0x40 == 0xffc0

    puts("Welcome to the secure program loader.");

start:
    puts("Please enter debug payload.");
    memset(0x2420, 0, MAX_SIZE);
    getsn(0x2420, MAX_SIZE - 1);

    if(0x8000 > dest || dest > 0xf000)
    {
        puts("Load address outside allowed range 0x8000 - 0xF000");
        goto start;
    }
    
    if(size < 6)
    {
        puts("Invalid payload length");
        goto start;
    }

    // move the digital signature into the top of the stack.
    memcpy(stackMemory, 0x2420 + size, 0x40);

    if(1 == flag)
    {
        // insert sha512 of the first size bytes of input into the top of the stack
        sha512(0x2420, size, stackMemory);
        
        /* compare between the sha512 and the signature from the input.
        for example:
         - input is [8000 01 06 3041
                    c26436953f8f3cadf1442fc218b18505
                    1ab6c20853a45f093fc32adf31529d05
                    a5ec3e96a9e41ed9ad1b14dcbdb98e50
                    e37a7ddc3d595b867807ed1605f2070e]
         
         - so compare between str1 and str2:
             - str1: sha512(800001063041)
             - str2: c26436953f8f3cadf1442fc218b18505
                     1ab6c20853a45f093fc32adf31529d05
                     a5ec3e96a9e41ed9ad1b14dcbdb98e50
                     e37a7ddc3d595b867807ed1605f2070e
        */
        if (memcmp(stackMemory, 0x2420 + size, 0x40) == 1)
            goto execute;
    }
    else if(0 == flag)
    {
        if(verify_ed25519(0x2400, 0x2420, size, stackMemory) == 1)
            goto execute;
    }
    else
    {
        puts("Invalid signature type.");
        goto start;
    }

    puts("Incorrect signature, continuing");
    goto start;
        
execute:
    puts("Signature valid, executing payload");
    memcpy(dest, 0x2424, size); // copy the code to the dest.
    ((func*)dest)();            // execute the code from the user

    goto start;
}
```

Therefore, it can be said that the input is divided as follows:
* Destination address (2 bytes)
* flag (1 byte)
* size of all without signature (1 byte)
* code to execute ((size - 4) bytes)
* Digital signature (0x40 bytes)

For example:

<img src="./21.3.png" width="80%"></img>

* code destination address: `8000`
* flag: `00`
* size: `06`
* code: `3041`
* digital signature:<br /> `c26436953f8f3cadf1442fc218b185051ab6c20853a45f093fc32adf31529d05a5ec3e96a9e41ed9ad1b14dbdb98e50e37a7ddc3d595b867807ed1605f2070e`

We couldn't find the private key for ed25519 in memory,<br />
so we can't generate a code that opens the door in a simple way like in the example.<br />
So, what can we do?

### How to exploit:

We will note that if the value of the flag is 1,<br />
all we need to open the door is that the string sha512(addr + flag + size + my code)<br />
be greater than the signature I enter.<br />
And if memcmp returns not just a value greater than 0, but an actual 1, we've won.

Let's start compiling our input:
* code destination address: `8000`
* flag: `01`
* size: `0c`
* code: `324000ff30401000`
    * same code from Lagos challange
    * you can check what it does by yourself.

To build a good "signature", let's look at sha512:
* sha512(`8000 01 0c 324000ff30401000`) == <br />
`09f072b7d3bfa6fce5ca0596a27818db0b054a0f0dd6bba7d3f9d3f810deae223b995ac0b07095a3460da22e46233a8c797ea9c305d6d05e0d1f4a2c1df3d3ff`

Now we'll want for the "signature" a sequence of bytes that is both lexically smaller than the sequence above and memcpy will return 1.

After a bit of trial and error, we found the sequence for which memcmp would return 1:<br />
`08000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000`

And that's all!

***Bonus:***<br />
From the trial and error I did on memcmp I understood how it works.<br />
Here is an implementation in c:
```c
char memcmp(char *s1, char *s2, unsigned short size)
{
    unsigned short i;

    for(i = 0; i < size; i++)
    {
        if (*(s1 + i) != *(s2 + i))
            return *(s1 + i) - *(s2 + i);
    }
    
    return 0;
}
```

## The cracking input (as bytes)
```
8000 01 0c 324000ff30401000 08000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
```