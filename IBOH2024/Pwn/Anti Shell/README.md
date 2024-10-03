# Anti Shell

## File Analysis
![image](https://github.com/user-attachments/assets/5c9a4839-5a6c-4d3f-9613-27395e73f796)

## Steps to Solve
## Step 1:

Use **Ghidra** to decompile the code, and I see that the first input actually does nothing. But the second input is actually vulnerable because of the <b>"(*(code *)local_28)();"</b> This will actually just execute anything inside the local_28 buffer.

<details>
  <summary>Decompile Code</summary>
  
```C
void main(void)

{
  undefined local_28 [28];
  undefined4 local_c;
  
  setup();
  printf("How many characters is your input?\n>> ");
  __isoc99_scanf(&DAT_0040202f,&local_c);
  getchar();
  printf("Okay, now give me your payload\n>> ");
  setup_jail();
  local_c = 0xd;
  vuln(local_28,0xd);
  (*(code *)local_28)();
                    /* WARNING: Subroutine does not return */
  exit(-1);
}
```

</details>

The **vuln** function actually just a normal **fgets**.

```C
void vuln(char *param_1,int param_2)

{
  fgets(param_1,param_2,stdin);
  return;
}
```

## Step 2
But from what we see, the local_28 is only taking 13 bytes as input. It is very hard to write a shell code in 13 bytes. Therefore, I tried to use this 13 bytes to modify the fgets size input. So that I can have enough size to write my shellcode.
 
![image](https://github.com/user-attachments/assets/ee17c2a2-08a0-4dcb-beab-866fd30fb7dd)
So from **GDB**, we can see that [rbp-0x20] is actually the local_28, and the size specify for the input is [rbp-0x4] because 0xd is moving into it.







