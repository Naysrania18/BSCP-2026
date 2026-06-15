
# PortSwigger Web Security Academy: User ID Controlled by Request Parameter with Password Disclosure

This repository contains a detailed walkthrough and solution for the **"User ID controlled by request parameter with password disclosure"** lab found in the Broken Access Control module of the PortSwigger Web Security Academy.

## 📝 Lab Description
* **Vulnerability Class:** Broken Access Control / Insecure Direct Object Reference (IDOR)
* **Objective:** Extract the administrator's password, log in as `administrator`, and delete the user `carlos`.
* **Your Credentials:** `wiener:peter`

---

## 🛠️ Exploit Walkthrough

### Step 1: Analyze the Target Application
1. Access the lab link using **Burp's built-in browser** or an intercepted proxy connection.
2. Navigate to the **My Account** page and log in using the provided credentials:
   * **Username:** `wiener`
   * **Password:** `peter`
3. Notice that after a successful login, the application redirects you to a URL structured as follows:
   ```http
   https://<LAB-ID>.web-security-academy.net/my-account?id=wiener
   ```
4. Observe that your password field is pre-filled on this page. This means the server returns the user's password in the raw HTTP response.

### Step 2: Test for Parameter Tampering (IDOR)
The application relies on the client-controlled `id` parameter to fetch user data instead of checking the session token server-side.

1. Modify the `id` parameter directly in the URL query string:
   * Change `id=wiener` to `id=administrator`
2. Send the request. The modified URL should look like this:
   ```http
   https://<LAB-ID>.web-security-academy.net/my-account?id=administrator
   ```
3. The server successfully processes the request and loads the administrator's account page dashboard.

### Step 3: Extract the Administrative Password
Because the password input field uses `type="password"`, the characters are masked on the screen. To retrieve the plaintext value:

1. Right-click the masked password input box on the page and click **Inspect** to open the browser Developer Tools.
2. View the highlighted HTML element in the DOM tree. It will resemble the following:
   ```html
   <input type="password" name="password" value="REDACTED_PASSWORD_STRING">
   ```
3. Copy the string value inside the `value="..."` attribute. *(Alternatively, double-click `type="password"` and change it to `type="text"` to reveal it directly on screen).*

### Step 4: Complete the Objective
1. Click **Log out** to clear your current session.
2. Return to the log in portal and authenticate using the stolen administrative credentials:
   * **Username:** `administrator`
   * **Password:** `<The copied password string>`
3. Once logged in, click on the **Admin panel** link located in the top navigation header.
4. Click the **Delete** button next to the user **carlos**.

---

## 🛡️ Remediation Guidance

### 1. Robust Access Control (Object-Level Validation)
Do not trust the identifier sent via client-controlled request parameters (`GET`, `POST`, or `JSON` body) to authorize data access. 
* Implement server-side verification that matches the user's active session state (e.g., cryptographic session tokens or JWTs) directly against the requested object record.

### 2. Data Minimization
* **Do not mirror passwords:** Applications should never transmit plaintext passwords back to the client interface. 
* For password change mechanisms, provide blank input fields requiring verification of the old password rather than exposing the current one in the HTML DOM structure.
