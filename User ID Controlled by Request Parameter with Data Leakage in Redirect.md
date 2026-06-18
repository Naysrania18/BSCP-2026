# User ID Controlled by Request Parameter with Data Leakage in Redirect.

## Lab Goal

Obtain the API key of the user `carlos` and submit it to solve the lab.

## Steps

### 1. Log in to the Application

Use the provided credentials:

```text
Username: wiener
Password: peter
```

### 2. Browse to "My Account"

After logging in, navigate to the account page.

Observe that the URL contains a user identifier parameter, for example:

```http
GET /my-account?id=wiener HTTP/1.1
```

### 3. Test for Horizontal Access Control

Modify the `id` parameter to another username:

```http
GET /my-account?id=carlos HTTP/1.1
```

Send the request.

### 4. Observe the Redirect

The application does not directly display Carlos's account page.

Instead, it responds with a redirect such as:

```http
HTTP/1.1 302 Found
Location: /login
```

### 5. Examine the Redirect Carefully

Look closely at the `Location` header.

The application leaks sensitive information in the redirect URL. You may see something similar to:

```http
Location: /login?message=Access+denied&apikey=xxxxxxxxxxxxxxxx
```

or another parameter containing Carlos's API key.

### 6. Copy Carlos's API Key

Extract the API key value from the redirect response.

Example:

```text
carlos_api_key = 5d8e4a7b9c1234567890abcdef123456
```

### 7. Submit the API Key

Use the **Submit solution** button in the lab and enter Carlos's API key.

### 8. Lab Solved

After submitting the correct API key, the lab will be marked as solved.

---

## Key Vulnerability

The application performs access control checks but leaks sensitive data through a redirect response. Although direct access to Carlos's account is blocked, confidential information is exposed in the redirect URL, allowing an attacker to retrieve the API key.
