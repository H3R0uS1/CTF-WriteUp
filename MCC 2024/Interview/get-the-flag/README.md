# Get-The-Flag
![image](https://github.com/user-attachments/assets/9b3499af-2fdf-47bb-8d9f-5519bae31226)

After unzip the file, we can set that a get-the-flag file, I do the basic file analysis and notice that the get-the-flag is a executable file. So I change the mode of the file to executable.

Then by running the executable, we see that is a snake game. Besides, that is a reminder with **You need to get exactly 16525 points for the flag**. Therefore, I know the goal is to set the score to 16525 points.

To do this runtime modification, we can use **_scanmem (commandline)_** or **_GameConqueror (Linux GUI)_** or **_Cheat Engine (Windows GUI)_**

## Step 1:
![image](https://github.com/user-attachments/assets/969ff140-d1e4-4293-a94b-10bcdb5c4ab1)

Run the "get-the-flag" executable.

## Step 2:
![image](https://github.com/user-attachments/assets/24d2df5a-bfc1-4058-bb99-2b2a13f4dbb5)

Without closing the executable, then we use **ps aux | grep "get-the-flag"** to find the pid for the executable, this will be used for the **scanmem**.

## Step 3:
![image](https://github.com/user-attachments/assets/d23794c8-ca6a-483b-a369-0448cb97fe7c)

Next, we run the **scanmem** and attach the pid of the "get-the-flag".

## Step 4:
![image](https://github.com/user-attachments/assets/619cb89e-c60e-40fb-b3ba-276c1285b891)

Then we search for the score value which is 0. We can see there are a lot of addresses with value of 0. 

## Step 5:
![image](https://github.com/user-attachments/assets/38c17edc-9256-4710-8bdc-6b048824c0cc)

Next, we will play the game and make the high-score and the current score is different.

## Step 6:
![image](https://github.com/user-attachments/assets/d6009629-cff5-48da-8130-9483ba4091ea)

Then we go back **scanmem** and search for the current score value. We can see there is only 1 address match the value, therefore we just modify that value to 16525.

## Step 7:
![image](https://github.com/user-attachments/assets/e5859491-b866-425e-895b-c6dfb97a40e3)

We just go back the executable and run it and then we can get our flag.
