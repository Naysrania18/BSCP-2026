# PortSwigger Lab Walkthrough: User role controlled by request parameter.

This guide outlines how to solve the [PortSwigger Academy "User role controlled by request parameter"](https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter) lab. 

## Vulnerability Overview

The application makes access control decisions using a client-side request parameter (a forgeable cookie named `Admin`). Because the server trusts this parameter without server-side validation, a regular user can escalate privileges by modifying the cookie value.

## Lab Goal

* Access the hidden admin panel at `/admin`.
* Delete the user `carlos`.

---

## Step-by-Step Solution

### 1. Map the Application
1. Open **Burp Suite** and ensure traffic passes through your proxy.
2. Navigate to the target web application lab URL using Burp's browser.
3. Log in to your account with the provided student credentials:
   * **Username**: `wiener`
   * **Password**: `peter`

### 2. Identify the Vulnerable Parameter
1. In Burp Suite, go to **Proxy** > **HTTP history**.
2. Find the `POST /login` request or the subsequent `GET /my-account` request.
3. Look at the request headers. Notice the application sets a cookie: `Admin=false`.

### 3. Exploit the Privilege Escalation
1. Right-click the `GET /my-account` request and select **Send to Repeater**.
2. Switch to the **Repeater** tab.
3. Modify the cookie value from:
   ```http
   Cookie: Admin=false
   ```
   to:
   ```http
   Cookie: Admin=true
   ```
4. Change the request path from `/my-account` to `/admin` and click **Send**.
5. Review the response body to confirm you now have access to the administrative page controls.

### 4. Perform the Admin Action
1. Locate the deletion URL for the user `carlos` in the HTML response (typically `/admin/delete?username=carlos`).
2. Update the target path in your Repeater request to that deletion endpoint.
3. Ensure the `Cookie: Admin=true` parameter remains intact.
4. Send the request. The server will process the request as an administrator and delete the user, solving the lab.
