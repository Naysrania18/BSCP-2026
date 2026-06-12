# Lab: User ID Controlled by Request Parameter, with Unpredictable User IDs

## Objective

Obtain Carlos's API key and submit it to solve the lab.

---

## Steps to Solve

### 1. Log in to the Application

Use the provided credentials:

```text
Username: wiener
Password: peter
```

After logging in, navigate to **My Account**.

---

### 2. Observe the Account URL

While viewing your account page, notice that the URL contains a user identifier similar to:

```text
/my-account?id=8d2b6c7a-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

The application uses a user ID parameter to determine which user's account information is displayed.

---

### 3. Find Carlos's User ID

Since the user IDs are unpredictable, you cannot simply guess Carlos's ID.

Navigate to the blog section of the website and open any post authored by **Carlos**.

Click on the **Carlos** author link.

You should be redirected to a URL similar to:

```text
/blogs?userId=4a1f9e2c-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

or

```text
/users/4a1f9e2c-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Copy Carlos's unique user ID from the URL.

---

### 4. Access Carlos's Account

Replace your own user ID in the account URL with Carlos's ID.

Example:

```text
/my-account?id=<Carlos-ID>
```

For example:

```text
/my-account?id=4a1f9e2c-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Load the page.

---

### 5. Retrieve Carlos's API Key

Because the application does not properly enforce access controls, Carlos's account page will be displayed.

Locate Carlos's API key on the page and copy it.

---

### 6. Submit the API Key

Click **Submit Solution** in the lab interface.

Paste Carlos's API key into the submission field and submit it.

---

## Why This Vulnerability Exists

The application relies on a user-controlled request parameter (`id`) to determine which account data to display. Although the user IDs are unpredictable, they are exposed elsewhere in the application (for example, through author profile links).

Because the server fails to verify that the authenticated user is authorized to access the requested account, an attacker can replace their own user ID with another user's ID and gain unauthorized access.

This is an example of an **IDOR (Insecure Direct Object Reference)** or **Broken Access Control** vulnerability.

---

## Result

✅ Carlos's API key obtained
✅ Lab solved
