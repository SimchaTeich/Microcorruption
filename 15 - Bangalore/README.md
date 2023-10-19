# Bangalore - 100 points
 
## The idea
Bypassing the mechanism of protected memory.

## The way

### Black box test:
The program asks the user for an input between 0x8 and 0x10 bytes long:

<img src="./15.1.png" width="80%"></img>

Is it really possible to insert only 0x10 bytes in total? From our experience so far, probably not. Let's see what happens here..

### Let's explore the code:

***Explain the `main`:***

As usual, the `main` calls `login`. And so we will probably have to overwrite a return value.

<img src="./15.2.png"></img>

But unlike previous challenges, this time before `login` another function is activated, called `set_up_protection`.
For now we will skip it directly to login. If necessary, we will return to investigate it later.

***Explain the `login`:***

At first glance, `login` looks suspiciously too easy:

<img src="./15.3.png"></img>

1. Allocate memory on the stack
    * up to 0x10 bytes

2. Get the user input
    * up to 0x30 bytes
    * insert it to the top of the stack

3. prints that password is inccorrect

***First attempt:***

It is very clear here that the return value can and should be overridden.
But, there is no function in the code segment that can open the door. Not even the well-known `INT`.
So we will have to inject a code that opens the door and jump to it.

If we investigate in another challenge what the `INT` function does with the parameter 0x7f, we find that the actual code that opens the door looks like this:

```arm
mov #0xff00, sr
call #0x10

; the bytes of the code after the assembler: 324000ffb0121000 
```

Return value at a distance of 0x10 from the top of the stack.
Therefore, the input will consist of 3 components:
* 0x10 random bytes
* New return address
    * since the code comes immediately after the return value, then the return value will be the address of the next 2 bytes in the stack from where it itself is located.
* The code above.

This is what the Stack memory looks like just before the input:

<img src="./15.4.png"></img>

* `4044` - 0x4440 is the return valur back to main.

And, this is the Stack memory just after the next input `00000000000000000000000000000000 0040 324000ffb0121000`:

<img src="./15.5.png"></img>

* `0040` is return value direct to the code `324000ffb0121000`

But unfortunately, this is what happens when trying to run the code:

<img src="./15.6.png" width="80%">

So the attempt failed because we tried to run code where we wrote it.
Remember the call to `set_up_protection` that occurred in the `main` function before `login`?
It's time to explore what it does.

***Explain the `set_up_protection`:***


### How to exploit:


## The cracking input (as bytes)
```
00000000000000000000000000000000 ba44 4000 0000 0640 324000ff30401000
```