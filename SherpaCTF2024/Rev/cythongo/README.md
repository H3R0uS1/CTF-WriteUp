# CPythonGO

For this challenge, I didn't manage to solve it when the CTF is ongoing. I only solved it after 1 week of the CTF end, but I still like to include this in the writeup, because I actually think this is a good challenge.

I also need to thanks to one of my teammate, **s0laris** who assist me to find the flag together. Without him, I will use even more longer time to solve this challenge.

## Initial Analysis

For this file given, we use ```file``` command to analysis the basic information of the file.

![image](https://github.com/user-attachments/assets/9fc400a8-3206-44ce-b16f-035dd2dc2f3c)

From here, we can't actually see anything interesting, just that thing need to highlight is **not stripped**, so that's mean the binary have the debug symbol for us to debug.

## Step 1

![image](https://github.com/user-attachments/assets/84ccaf48-0dff-41da-88d5-0a5d38c5df0d)

So when I first, test run the file, it shows that **"Enter Password:"**, so I just randomly enter a password, and it will return with **"Incorrect password !! No flag for you.."**

So after this, I know the main goal of this challenge is need to find a **correct password** for it. So quickly, I go to decompile the binary and see what can I found in the decompmile code.

## Step 2

![image](https://github.com/user-attachments/assets/870a341c-c403-4ea9-afbc-1d52779e65f6)

But when I go inside the **Ghidra** and trying to look for "Enter Password" string, I didn't maange to found it. So because of the title of the challenge and also I did some challenge like this before, which can be found in the [IBOH2024 Crack the Hex]. Then I assume that this is actually a **GO** language file hide in a **Python** file and the **Python** file inside this **C** file. That's why the reason I cannot find the string **""Enter Password:"**. 

Then after knowing what to do, then I just try to find out where is the suspicious function that doing this process.

## Step 3

![image](https://github.com/user-attachments/assets/28955cb7-bf84-4d02-b122-d1ef5f854a4b)

So from the decompile code of the file, I see one very suspicious function which is **"decrypt"**, then I just dive into the **decrypt** function and see what is it doing.

![image](https://github.com/user-attachments/assets/b891897e-6797-4f63-a2e0-e3f3d7db9b21)

After looking at the **decrypt** function, I can see that is it actually doing a **for loop** of **XOR**. But the difficulty right now is, I didn't know what is the **ctx** actually, so I just go to **GEF** and find for this function and reverse it step by step.

## Step 4

![image](https://github.com/user-attachments/assets/e65c4dfe-cba2-47d6-8bc7-229b574298bd)

After go into the **decrypt** function, we see the assembly that it compare the **QWORD PTR [rbp-0x40], rax**, if **QWORD PTR [rbp-0x40]** is above rax, then it will jump to the **<decrypt+34>** which is the for loop, so that from here I can actually see how many times it do the for loops. Then when I inspect the **rbp-0x40**, it shows that is 0x900000, so that I know is 0x900000 times for loops, and the rax now is 0 is because this is the start of the decrypt function, it haven't go through any decrypt yet.

![image](https://github.com/user-attachments/assets/6481f9b7-960f-41e2-ad3f-6d06e7218a6d)

Then at the assembly here, I can see where is the bytes that start to be taken to perform the xor, the start of the address now is **0x00007ffff74a7010**, then it will keep xor with the key for 0x900000 times. So now I have to extract out the range of the bytes, and save it into a new file first. Then I can use it for my decrypt function.

![image](https://github.com/user-attachments/assets/1d06c3a6-c5ab-44a0-b410-12da169daffd)

Now I have the bytes and the range already, so I just have to find the **"flag"** now. As we saw the **strlen** function is taking the flag as arguement, so I can actually go to the strlen function and inspect the rdi. And then I can get the flag.

![image](https://github.com/user-attachments/assets/7dbfd160-5728-4b3a-8d20-09a65929e805)

## Step 5

After I have all the information needed in the decrypt function, then I can start to write a python script to decrypt it.

```python
def reverse_decrypt(ctx, flag):
    flag_len = len(flag)  # Length of the flag string (should be 0x34, i.e., 52 bytes)
    
    # Convert ctx to a mutable list of integers if it's not already
    ctx = list(ctx)  # If ctx is a bytes object, convert it to a list of byte values
    
    for i in range(len(ctx)):
        # Reverse the XOR operation to decrypt the data
        ctx[i] = ctx[i] ^ ord(flag[i % flag_len]) ^ i
    
    # Ensure all values are in the valid byte range (0-255) before converting back to bytes
    ctx = [val % 256 for val in ctx]  # Apply modulo 256 to ensure valid byte range
    
    # Return the decrypted data as a byte array
    return bytes(ctx)

def read_ctx_from_file(file_path):
    # Open the file in binary read mode and read the data into ctx
    with open(file_path, 'rb') as f:
        ctx = f.read()  # Read the entire file into a bytes object
    return ctx

# Example usage:
# Read the encrypted data (ctx) from the file
ctx = read_ctx_from_file('./memory_dump.bin')  # Replace with the actual file path

# The flag should match the expected flag length (52 bytes)
flag = 'Here is the flag! https://r.mtdv.me/here-is-the-flag'  # Replace with the correct flag

# Call the reverse_decrypt function to decrypt the data
decrypted_data = reverse_decrypt(ctx, flag)

with open('cpythongo2', 'wb') as f:
    f.write(decrypted_data)


# Print the decrypted data (as a string if it's readable text)
# print("Decrypted data:", decrypted_data.decode(errors='ignore'))  # Decode bytes to string and handle errors
```

![image](https://github.com/user-attachments/assets/f7ecb692-c7c7-4226-a5d6-563bfbf370e9)

Then after with the decrypt, we can see the output **cpythongo2**, and use the ```file``` command to analysis again can see that, there is another ELF file with stripped.

## Step 6

Since this is a **stripped** file, so I can't actually find a debugging symbol for this. But then I try to use the backtrace command in gdb ```bt``` to find out what function actually have done when running the file.

![image](https://github.com/user-attachments/assets/0ac631f2-7938-46b8-bc71-9146e2e5e1e7)

By searching those function in the ghidra, I saw something like **"pyi-bootloader-ignore-signals"**. Because of the **"pyi"** keyword, then I actually just think that is the python file. So I use ```pyinstxtractor``` to get the **.pyc** file and then use ```pycdc``` to revert the file into python code. Because I already have the pyinstxtractor and the pycdc tools in my FlareVM, so that I just use my FlareVM to done this step.

![image](https://github.com/user-attachments/assets/a703315c-9cae-4411-b24c-1161c06941bb)

After using ```pyinstxtractor```, looking at the .pyc file, there is one file very suspicious which is the **"test.pyc"**. Then I just ```pycdc``` to convert this test.pyc into the python code.

![image](https://github.com/user-attachments/assets/10e3e52d-51a4-404f-82e1-f2f301f78625)

## Step 7

After having the python file, then we can start to analysis the file inside the python file.

```python
# Source Generated with Decompyle++
# File: test.pyc (Python 3.8)

import ctypes
import os
import base64
import time
import zlib
libc = ctypes.CDLL(None)
syscall = libc.syscall
fexecve = libc.fexecve
x = zlib.decompress(base64.b64decode(b'A SUPER LONG BYTES CAN\'\\t IN THIS GITHUB '))
key = b'CythonGo!'
content = bytearray(len(x))
for i in range(len(x)):
    content[i] ^= x[i] ^ key[i % 9]
    if i != len(x) - 1:
        content[i + 1] ^= content[i]
        continue
        fd = syscall(319, '', 1)
        os.write(fd, content)
        t = int(time.time()).to_bytes(4, 'big', **('byteorder',))
        key = bytearray('SherpaSecIstheBEST', 'utf-8', **('encoding',))
        for i in range(len(key)):
            key[i] = t[i % 4] ^ key[i]
        argv = ctypes.pointer(ctypes.c_char_p * 2(b'SGVyZSBpcyB0aGUgZmxhZyEgaHR0cHM6Ly95b3V0dS5iZS9kUXc0dzlXZ1hjUT90PTQy', bytes(key)))
        fexecve(fd, argv, argv)
        return None

```
