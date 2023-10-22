# Cold Lake - 20 points

## The idea
The digital signature ed25519.

## The way

### Black box test:
In the instructions of the challenge it becomes clear that the user has to enter an address,<br />
program and signature of the program.

<img src="./20.1.png" width="80%"></img>

In contrast to the Vancouver challenge, here it is necessary to sign the plan as a security measure.<br />
It can be concluded that we cannot execute our own program the easy way.

So, let's jump the code.

### Explore the code:

<img src="./20.2.png"></img>
<img src="./20.3.png"></img>

### How to exploit:


## The cracking input (as bytes)
```
0880
```
```
324000ff30401000
```
```
00
```
```
0090
```
```
3540088000450545054505450545054505450f433041
```
```
8605e027f42368ea6bba9de66409f6a8ddedcd49614a4648281c47a7b4ad252f5639069b17ba8ff104d371e2d8a625b038f0750667364087e7987e40ea81510f
```