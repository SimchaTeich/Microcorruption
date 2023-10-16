# Jakarta - 40 points
 
## The idea
Bypassing the input length verification mechanism using a game of byte addition sums.

## The way
In the black box test, it seems that a username and password must be entered whose total length does not exceed 0x20 bytes.

<img src="./10.1.png" width="85%"></img>
<img src="./10.2.png" width="85%"></img>

In function `main` there is not too much except a call to `login`. So we will examine `login`.

<img src="./10.3.png"></img>
<img src="./10.4.png"></img>

*Explain:*
1. Get username from user
    * up to 0xff bytes
    * enters memory at address 0x2402

2. Calculation of username length
    * _r11_ contains the username length (without 0x0 at the end)

3. Copying the username to the top of the stack
    * using `strcpy`

4. Verification that the length of the username does not exceed 0x1f
    * 0x20 - 0x1 = 0x1f. username length without 0x0 at the end..
    * if the test failed, we will continue to 5.
    * otherwise, we will skip 5.

5. Printing to the screen that the password is too long, and the end of the entire program
    * this is not a mistake, indeed the length of the username is checked and an output is printed that talks about the password.

6. Get passwrord from user
    * up to 0x1f - len(username) bytes ⚠️
    * enters memory at address 0x2402.

7. Copying the password to the top of the stack after the username
    * using `strcpy`

8. Calculation of the total length of the username and password
    * _r15_ contains the len(username + password)

9. Verification that the length of the username and the password together does not exceed 0x1f
    * 0x20 - 0x1 = 0x1f. username+password length without 0x0 at the end of the password.
    * if the test failed, we will continue to 10.
    * otherwise, we will skip 10.

10. Printing to the screen that the password is too long, and the end of the entire program

11. Checking that the username and password are correct
    * We have no way of knowing the login details so we will always fail and skip opening the door.


Summary:
* Receiving a username and checking its length
     * if it is too long, the program will end
* Receiving a password and checking the total length of the username and password
     * if it is too long, the program will end
* Checking correctness of username and password
     * always fail, we will skip opening the door and reach the `login` ret command.




```python
username = '11' * 0x20
```

```python
password = '22222222' + ' 4c44 ' + '22' * (0xe0 - 0x6)
```


## The cracking input (as bytes)
```
1111111111111111111111111111111111111111111111111111111111111111
```
```
22222222 4c44 2222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222
```
