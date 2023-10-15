# Reykjavik - 35 points
 
## The idea
First and simple use of the disassembler.

## The way
First, we will look at the program as a black box. That is, by running only.

<img src="./5.1.png" width="60%"></img>

When we look at `main`, we will notice that two functions may interest us. One named `enc`, and the other has no name.

<img src="./5.2.png" width="60%"></img>

Out of basic curiosity we will check the memory of the address 0x2400 and find that there is nothing there (before running the program).

<img src="./5.3.png"></img>

When we get to `main`, even before we run it, we will discover that the memory at address 0x4538 has been copied to the memory at 0x2400 by the `__do_copy_data` function.

<img src="./5.4.png" width="60%"></img>

<img src="./5.5.png" width="60%"></img>

<img src="./5.6.png" width="60%"></img>

Actually I don't know why exactly it happens like this.. but we will continue.
We'll notice that without seeing the contents of the `enc` function, when it finishes, the entire contents at address 0x2400 have changed.

<img src="./5.7.png" width="60%"></img>

<img src="./5.8.png" width="60%"></img>

First of all, to understand where the function that starts at 0x2400 ends, we will look for the ret command. In hexadecimal the command is 3041, which immediately gives us a division of the above code into two different functions.

So we will disassemble first the short function, and this is the code we will get:

<img src="./5.9.png" width="60%"></img>

This is exactly the code of `INT` from the previous challenges.

So, the long code is surely also responsible for printing the message to the screen, both receiving the input from the user and comparing it to the password.

You can also see it in the code itself in the next picture, and also deduce it ourselves. Because so far, for the user "nothing happened", and immediately after this function the program ends...

<img src="./5.10.png" width="60%"></img>

* Prints the sentence: "what's the password?"
    * interrupt 0x0 - print one char
    * "what's the password?" saved at address 0x4520.
    * print char by char in loop.
* Prints '\n'
    * interrupt 0x0
    * asscii of '\n' is 0xa.
* Gets input from user
    * interrup 0x2
    * size up to 0x1f bytes
    * While debugging it is easy to see that the input goes into address 0x43da.
* comparison operation
    * between the first 2 bytes of the user input to the values 0x74 and 0x68.
    * If they are equal it will open the door. Otherwise it will skip opening the door.

So, the password is any series of bytes (up to 0x1f bytes long) which starts with 0x74 and 0x68.

## The cracking input (as bytes)
```
7468
```


