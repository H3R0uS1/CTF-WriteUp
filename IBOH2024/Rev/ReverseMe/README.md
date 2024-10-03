# ReverseMe

## File Analysis
We can see that file is only just a **DATA**, so at first I actually don't know what to do with it. I just continue to analyze the file.

![image](https://github.com/user-attachments/assets/e4947797-b145-4256-aabe-c32cd9486474)

# Step to Solve:
## Step 1:
Use **strings** command to see something suspicious that looks like the bytes of the file is being reversed. (We can see that _**'tnemmoc.'**_ reverse will be _**.comment**_, _**'ssb.'**_ reverse will be **_.bss_**)

![image](https://github.com/user-attachments/assets/62396830-c6b0-4dc7-8b07-e2094a8ff978)

## Step 2:

Write a python script to make a new file that reverse the file's bytes and save into a new file.


```python
with open("./ReverseMe", "rb") as file:
	content = file.read()

with open("./NewReverseMe", "wb") as file:
	file.write(content[::-1])
```

## Step 3:

Use **strings** command again and make sure that the bytes have changed to thing that is easy to under. So we use **file** command again to look at the file and notice it becomes an **ELF** file now.

![image](https://github.com/user-attachments/assets/b62cc50b-74bc-44d3-baae-dc5d683a004a)

With this, we can see many familiar things already like **"libc"**, even the bottom there we can also see like **".comment"**. Then we use analyze the file again using **file** command.

![image](https://github.com/user-attachments/assets/78a2b94b-7732-401d-bef8-868d02150c54)

We can see that now the **NewReverseMe** file become **ELF**. That's mean now we can run this file already. (remember to chmod of the file)

![image](https://github.com/user-attachments/assets/eea95906-c903-4ff6-b020-46fa4b7c4376)

The output given a bunch of numbers but we don't know what is that doing.

## Step 4:

Then, we use **IDA** to decompile the file. (Actually can use _Ghidra_ also, but when I tried with Ghidra I feel like the decompile is weird for this binary, so I used IDA.)


<details>
  <summary>IDA DECOMPILE CODE</summary>

```C
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int index; // ebx
  _DWORD array[21]; // [rsp+9h] [rbp-C7h] BYREF
  char s[72]; // [rsp+60h] [rbp-70h] BYREF
  int key_length; // [rsp+A8h] [rbp-28h]
  int j; // [rsp+ACh] [rbp-24h]
  int i; // [rsp+B0h] [rbp-20h]
  int counter; // [rsp+B4h] [rbp-1Ch]
  char *nptr; // [rsp+B8h] [rbp-18h]

  strcpy(s, "33 39 42 32 87 81 19 11 12 43 0 58 31 81 55 5 58 16 56 24");
  nptr = strtok(s, " ");
  counter = 0;
  while ( nptr )
  {
    index = counter++;
    *(_DWORD *)((char *)&array[index + 1] + 3) = atoi(nptr);
    nptr = strtok(0LL, " ");
  }
  for ( i = 0; i < counter; ++i )
    printf("%d ", *(_DWORD *)((char *)&array[i + 1] + 3));
  putchar(10);
  strcpy((char *)array, "heehee");
  key_length = 6;
  for ( j = 0; j < counter; ++j )
    *(_DWORD *)((char *)&array[j + 1] + 3) ^= *((char *)array + j % key_length);
  return 0;
}

```
</details>

## Step 5:

We can notice it tokenize a hardcoded value and **XOR** it with **heehee**. So we just write a python script and to reverse the **XOR** and so on we get the flag.

<details>
  <summary>Solve Script</summary>
  
```python

string = "33 39 42 32 87 81 19 11 12 43 0 58 31 81 55 5 58 16 56 24"
key = "heehee"

flag = ''.join([chr(int(string.split()[i]) ^ ord(key[i % 6])) for i in range(len(string.split()))])

print(flag)

```
</details>

> Flag: IBOH24{niCe_w4Rm_uP}
