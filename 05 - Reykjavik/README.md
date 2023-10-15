# Reykjavik - 35 points
 
## The idea
First and simple use of the disassembler.

## The way
First, we will look at the program as a black box. That is, by running only.

<img src="./5.1.png" width="70%"></img>

When we look at `main`, we will notice that two functions may interest us. One named `enc`, and the other has no name.

<img src="./5.2.png" width="70%"></img>

Out of basic curiosity we will check the memory of the address 0x2400 and find that there is nothing there (before running the program).

<img src="./5.3.png"></img>

When we get to `main`, even before we run it, we will discover that the memory at address 0x4538 has been copied to the memory at 0x2400 by the `__do_copy_data` function.

<img src="./5.4.png" width="70%"></img>

<img src="./5.5.png" width="70%"></img>

<img src="./5.6.png" width="70%"></img>

Actually I don't know why exactly it happens like this.. but we will continue.
We'll notice that without seeing the contents of the enc function, when it finishes, the entire contents at address 0x2400 have changed.

<img src="./5.7.png" width="70%"></img>

<img src="./5.8.png" width="70%"></img>



## The cracking input (as bytes)
```
7468
```


