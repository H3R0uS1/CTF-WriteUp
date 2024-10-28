# Crack the Hex

Flag Format: IBOH24{password}

## Initial Analysis

For this challenge, the file provided is an **'.exe'** so that, I know this a **Windows Executable**. Therefore, I'm going to use **FlareVM** to analysis and solve this challenge with **IDA** as my disassembler. 

## Step 1

Then when we try to run the **'.exe'**, it will prompt "Are you sure you want to delete 1973866 files?", but after I tried when clicking the "YES" or "NO" it will also just mentioned "Just Kidding". But other than that, the binary didn't show anything already.

![image](https://github.com/user-attachments/assets/d013faa3-792c-4cb1-bb10-2d6b1efa3fb4) ![image](https://github.com/user-attachments/assets/63674222-e25e-4b4a-91cf-f91092c7ab73)

## Step 2

Then, I just use IDA to disassemble the binary and see what is it doing. By browsing the assembly, I suspicious that the sub_140001522 will be the main function because it is the only one which does most of things.

![image](https://github.com/user-attachments/assets/645757df-b6e9-4381-bbc2-c409e523c56f)

## Step 3

By browsing the assembly code, I saw some suspicious value is moving into **var_7B**, therefore I just use **cyberchef** and see what is it.

![image](https://github.com/user-attachments/assets/139542bd-2efb-473f-a532-f33fb104ae54)

If you look at the input in the cyberchef carefully, you will see that I only take '3432686F6269h' instead of '343268686F6269h'. The reason for this is because the second mov is at the offset 'var_7B + 3', therefore, it will overwrite the 4th byte in the first mov. After that, I'll rename the var_7B to **'iboh'**

![image](https://github.com/user-attachments/assets/f9451b86-f2f9-42c6-ab2e-56ad31582908)

> The reason to use reverse is because of endianess, which mean the value will be pushed from **LSB** to **MSB**.

## Step 4

Next, I just browse the assembly and found one loops, but at first I still don't know what is it doing. I just know that keep **XOR** some value. Then I quickly identify that **var_14 is a counter** because it will keep looping until it value is not below the **edx**.

![image](https://github.com/user-attachments/assets/c8ce4029-151c-4b68-a4f7-64086e778a07)

But, at this point I don't know what is the value of the **var_74**, therefore I **set a breakpoint at cmp** then just see the value of the eax. 

![image](https://github.com/user-attachments/assets/140a559d-fa34-4ca3-a7ab-044f4f6184a5)

So now I know that the **eax** value will be "5000h", therefore I know that it will loops for "5000h" times. But let's see what the value is doing the **XOR**.

## Step 5

After that, I step by step debug the disassembly and find out that the **XOR** will take the **IBOH24** and keep **XOR** with the value located starting from 00007FF606E7B0A8 for 5000h times.

![image](https://github.com/user-attachments/assets/db6c55de-8dbf-42ed-b787-63f1ce9625f7)

Then what I have to do is find out the address 00007FF606E7B0A8, and take the value from it for 5000h times.

![image](https://github.com/user-attachments/assets/4a51f91d-e1d6-4772-8f3b-15bfa6203369)

## Step 6

Then I use **HxD** and search the value "24 38 FF 68 31 34 69 62".

![image](https://github.com/user-attachments/assets/6160b470-9e47-4d7e-9cd3-3044c0e4fcff)

After I found it, then I just **Select Block** starting from the **offset 4CA8**, and the length is **5000 hex**

![image](https://github.com/user-attachments/assets/4a3003f8-9643-439f-9446-3ca56b36dc65)

Then I just copy all the selected blocks and save it into a new file.

## Step 7

After having the bytes that repeat for 5000h times, then also have the key that XOR with. So now I take the **bytes and the key** and throw into **cyberchef**.

![image](https://github.com/user-attachments/assets/d657640d-3dc8-423e-8297-1dc8493de53e)

The Magic Output shows me that, after XOR the bytes and the keys, a **Windows Portable Executable File detected**. So I just download the file, and run it again. But then when I run the new file, it also shows the same thing, which is only delete files and just kidding.

## Step 8

Since the file is still doing the same thing, then I try to disassemble the new file and see what is it doing.

![image](https://github.com/user-attachments/assets/cd46c4db-b635-4d48-9875-725205c29982)

By browsing this new disassemble code, the structure is quite similar to the first file. But just that now this file can see the strings like "Confirm File Delete". So I just do the same thing, and browse the disassemble also see it doing the XOR thingy. But this time with different bytes and value.

![image](https://github.com/user-attachments/assets/6fd5832f-c3e7-4774-b1df-0fd47e9c98e7)

Then from here, I also quickly identify that **var_18 is the counter** and **var_20 value** can be found at the above image which is **12Ch**

## Step 9

Then now I know it loops for 12Ch times, but now I didn't know the value that are performing the XOR. So I do the same thing, I set breakpoint at the cmp and slowly step by step debug and see the register.

![image](https://github.com/user-attachments/assets/2b25d759-f845-408c-9773-daf73bcd7fdd)

From this image, I know that the ecx value will be located at address 00007FF6A8733020, and it will loop for 12Ch times. Now have to find the key, which is the eax.

![image](https://github.com/user-attachments/assets/b1ce0fac-d55a-40af-9a56-fca6da023b32)

And from this image, I know that the eax value will be located at address 00007FF6A8733000, and it will doing **modulus with 0Ch which is 12 in decimal** which means keep looping the bytes every 12. (0Ch is the var_28 and can be found on the above image at step 8). 

![image](https://github.com/user-attachments/assets/2152813f-b58c-40fb-858a-b26b8e1c0cf4)

## Step 10

![image](https://github.com/user-attachments/assets/4fda8705-7f34-4afb-8cd9-a4e97df8ecba)

So now I just do the same thing, I get some of the **hex dump value** of the **ecx** and search it in **HxD**

![image](https://github.com/user-attachments/assets/8a4a3865-8ccc-497d-9cae-2ce24c652b4d)

Next, go to **Select Block** and choose the starting offset and the length, which is **12Ch**

![image](https://github.com/user-attachments/assets/7c38c9df-8606-4b5f-b4ae-6e4998a9947a)

Then I just do the same thing, copy this value and save it in a new file. 

## Step 11

Next, I go to Cyberchef and input the file and do the XOR with the 12 bytes that got earlier.

![image](https://github.com/user-attachments/assets/5d467d7b-c9b3-4aa3-9424-35427d34f65f)

## Step 12

After XOR with the value, there is something suspicious at the output, which is **'net user flag wh0s_d4t?!?1!'**. Then by doing some research what is **net user**, it will shows: 

![image](https://github.com/user-attachments/assets/a8052f7b-075f-4616-98ce-b87b2c9d700d)

Reference: [Net user](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771865(v=ws.11))

Lastly, since the flag format for this challenge is **IBOH24{password}**. So that's mean I got the flag for this challenge, which is **IBOH24{wh0s_d4t?!?1!}**.






