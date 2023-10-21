# Vancouver - 10 points
 
## The idea
Naive exploitation of a debug payload as input.

## The way

### Black box test:
This is the first challenge where the user is asked to enter an input that is a program,<br />
and not a username and password as we are used to.

<img src="./19.1.png" width="80%"></img>

In the explanation screen of the challenge, an example of input appears:

<img src="./19.2.png" width="80%"></img>

So the background story is that you can only insert programs whose role is to "map bugs",<br />
and the program `8000023041` is provided, which we still don't know how it is composed and what it does.<br />
Time to explore the code!

### Explore the code:



### How to exploit:


## The cracking input (as bytes)
```
8000 08 324000ff30401000
```