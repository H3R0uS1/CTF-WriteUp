# ReverseMe

## File Analysis


# Step to Solve:
## Step 1:

Use **strings** command to see something suspicious that looks like the bytes of the file is being reversed.

## Step 2:

Write a python script to make a new file that reverse the file's bytes and save into a new file.

```python
print("hello")
```

## Step 3:

Use **strings** command again and make sure that the bytes have changed to thing that is easy to under. So we use **file** command again to look at the file and notice it becomes an **ELF** file now.

## Step 4:

Then, we use **IDA** to decompile the file. (Actually can use _Ghidra_ also, but when I tried with Ghidra I feel like the decompile is weird, so I used IDA.)

## Step 5:

We can notice it tokenize a hardcoded value and **XOR** it with **heehee**. So we just write a python script and to reverse the **XOR** and so on we get the flag.

> Flag: IBOH24{}
