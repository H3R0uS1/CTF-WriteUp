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
