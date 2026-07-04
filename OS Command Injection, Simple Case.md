# LAB: OS Command Injection, Simple Case.

The target application takes user input via a product stock check feature and executes an arbitrary operating system command using that input. Because the input is not validated or sanitized, an attacker can append their own commands. 

To solve this lab, you must execute the `whoami` command to determine the name of the current user.

---

## 🛠️ Step-by-Step Walkthrough

### 1. Identify the Vulnerable Feature
1. Open the lab homepage.
2. Select any product to view its details page.
3. Scroll down to the **Check Stock** section.
4. Open your browser's Developer Tools (`F12`) or use an intercepting proxy like PortSwigger Burp Suite.
5. Click the **Check Stock** button.

### 2. Analyze the Request
The application sends a `POST` request to the `/product/stock` endpoint with the following parameters in the body:
```http
productId=1&storeId=1
```
The server passes these values directly into a shell command background process, likely looking similar to this:
```bash
stock_check_script.sh 1 1
```

### 3. Inject the Command
You can use a command separator like `&`, `&&`, or `|` to chain a malicious command onto the end of the legitimate argument.

1. Intercept the stock check request.
2. Modify the `storeId` parameter to append the command payload. 

#### Example Payload:
```http
productId=1&storeId=1|whoami
```
*(Alternatively, you can try `1; whoami` or `1 && whoami` depending on the operating system environment).*

### 4. View the Result
Submit the modified request. The application will execute the original script, hit the separator, and immediately run `whoami`. 

The web server will return the raw command output directly inside the HTTP response:

```http
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8

peter-XXXXXX
```

The output (e.g., `peter-XXXXXX` or `www-data`) will appear on your screen, successfully completing the lab.

---

## 🛡️ Remediation Guidance

To secure this endpoint and prevent OS command injection:
* **Avoid direct shell execution:** Never pass raw user inputs directly to OS shells (like `system()`, `exec()`, or `popen()`).
* **Use built-in APIs:** Use the programming language's internal file system or database API handlers instead of calling command-line binaries.
* **Implement strict allowlisting:** If system commands are unavoidable, validate user input against a strict allowlist of expected safe alphanumeric characters or integers.
