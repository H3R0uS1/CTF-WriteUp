# Lekir Framework -- Json Web Token
![image](https://github.com/user-attachments/assets/a17138b4-a122-4740-bd3a-b4418f79b504)

First of all, let's review the source code provided.

```php
<?php
-- get-data.php
require_once 'vendor/autoload.php';

use Firebase\JWT\JWT;

// Your secret key for signing the token
$secretKey = 'password';

// User information or any other data you want to include in the token
$userData = [
    'jwtrole' => 'user'
];

// Create a token
$token = JWT::encode($userData, $secretKey, 'HS256');


-- ./api/process-token.php
//verify the token
if (isset($_POST['token'])) {
$token = $_POST['token'];

try {
    $decoded = JWT::decode($token, new Key($secretKey, 'HS256'));

    $role = $decoded->jwtrole;

    session_start();
    $_SESSION['jwtrole'] = $role;

    header('Location: ../jwt.php');
    exit();
}

-- jwt.php
//logic
if(isset($_SESSION['jwtrole'])){
  if($_SESSION['jwtrole'] === 'admin'){

    $data = "FLAG = FLG_H@k1M3TaWP@n!";

  } elseif($_SESSION['jwtrole'] === 'user'){

    $data = "Oppppsss! No permission!";

  } else {

    $data = "Come get your flag!";
  }
}
?>
```
From the source code provided, we know that we need to change the **jwtrole** to **admin**, and the **secret key** for the token is **password**. Then start to exploit.

## Step 1
![image](https://github.com/user-attachments/assets/821789eb-ff7c-413a-920d-bfe42fb02976)

Click **"Show me the Flag"** and it shows **No Permission**, then we go to burpsuite and check the requests.

## Step 2
![image](https://github.com/user-attachments/assets/d0be8a01-7c19-4887-b0b1-08af8352cd7a)

By browsing those 3 requests, we see that the **POST** Request, got a token looks like jwt, and since the challenge already mentioned about Json Web Token, so can tell that is a jwt.

## Step 3
![image](https://github.com/user-attachments/assets/d45f4f80-4c0c-445a-b82b-2028bb1ef816)

Copy the jwt and browse [jwt.io](https://jwt.io/) then paste the jwt in the column. Then we can see the **_Header_, _Payload_, _Verify Signature_**

## Step 4
![image](https://github.com/user-attachments/assets/b901fdd0-8c48-4d5a-84a2-abb93d8ff919)

We can see that the in the Payload column, **"jwtrole" = "user"**. Then we can change it to **"jwtrole" = "admin"**. Besides, have to change the secret key to **password**.

## Step 5
![image](https://github.com/user-attachments/assets/8e6a9003-b535-4013-986a-33fd34fb6743)

After that, copy the new jwt and send back the **POST** request with the new jwt. Then, we can follow the redirection and show the response in browser.

## Step 6
![image](https://github.com/user-attachments/assets/64a7adfe-0efa-401d-a6cb-9876300d8b51)

Just like that, and we got the flag.
