# Sydney - 15 points
 
## The idea
A challenge very similar to New Orleans that illustrates how bad it is to keep the password in the source code.

## The way

Function main is very similar to New Orleans but
without calling to `unlook_door`. Instead, the operation of unlooking the door is raise interrupt with 0xf7 as parameter.

<img src="./2.1.png" width="75%"></img>

## The cracking input (as bytes)
```
5f26452663642e3f
```


