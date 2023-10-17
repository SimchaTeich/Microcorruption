# Addis Ababa - 50 points
 
## The idea
The weakness of printf

## The way
Black box testing tells us that the input to unlock is username and password separated by colons.

<img src="./11.1.png" width="80%"></img>

Let's look at function `main`:

<img src="./11.2.png"></img>

*Explain:*
1. Get input from user
    * up to 0x13 bytes
    * insert into memory at address 0x2400

2. Copy the user input from 0x2400 into the stack
    * top of the stack at 0x4212
    * the input inserts into 0x4214
    * using `strcpy`

3. Checking the correctness of the input
    * using `test_password_valid` that using interupt so we cant guess the password
    * the function will always return 0 for us
    * result move from _r15_ to 0x4212.
    * so, always at this point: *(0x4212) == 0x0

4. Printing the input from the user to the screen
    * using `printf`
    * so far in the other challenges we have used `puts`, So the use of `printf` raises suspicion...

5. Checking the result of `test_password_valid`
    * always jump to stage 7 (read step 3 again)

6. Unlock the door.

7. Prints that the input is incorrect.


Summary:
* Before printing the input from the user to the screen, the stack contains two things:
    * 2 bytes (address 0x4212) whose total value is 0 if the input is incorrect
    * the input from the user immediately after them (address 0x4214)
* After printing, these 2 bytes are checked
    * If they are 0, we lost
    * Otherwise, the door opens
* Conclusion: we will have to use `printf` to write a value other than 0 to those 2 bytes - at  address 0x4212




## The cracking input (as bytes)
```
4212 2578 2563
```