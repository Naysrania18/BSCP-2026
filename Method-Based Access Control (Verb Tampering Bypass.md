# Lab Solution: Method-Based Access Control (Verb Tampering Bypass

## Objective
Bypass insecure method-based access controls to gain unauthorized access to a restricted administrative functionality (e.g., deleting a user account or accessing an admin panel).

---

## Vulnerability Explanation
Method-Based Access Control vulnerabilities occur when a web application or application server configuration restricts access to specific HTTP methods (like `GET` or `POST`) but fails to secure others. 

If a rule explicitly denies or allows access only to `GET` and `POST`, an attacker can swap the HTTP method to an alternative verb (such as `HEAD`, `OPTIONS`, `PUT`, or arbitrary strings like `JEFF`) to bypass the filter while the backend still executes the underlying logic.

---

## Step-by-Step Walkthrough

### Step 1: Intercept the Target Request
1. Open your browser and route your traffic through a web proxy (e.g., **Burp Suite**).
2. Navigate to the restricted administrative page or trigger the administrative action (e.g., clicking a "Delete User" button).
3. If access is blocked, you will receive an error like `403 Forbidden` or `401 Unauthorized`.
4. Locate this blocked request in your proxy's HTTP history.

### Step 2: Send Request to Repeater
1. Right-click the blocked request in Burp Suite.
2. Select **Send to Repeater**.
3. Move to the **Repeater** tab to manipulate the request manually.

### Step 3: Test Alternative HTTP Methods
Modify the HTTP verb in the very first line of the raw request. Test the following techniques:

#### Technique A: HTTP Verb Swapping
Change the method from `POST` or `GET` to alternative standard methods:
* Change `POST /admin/delete?username=victim HTTP/1.1` to:
  ```http
  GET /admin/delete?username=victim HTTP/1.1
  ```
* Alternatively, try the `HEAD` method:
  ```http
  HEAD /admin/delete?username=victim HTTP/1.1
  ```

#### Technique B: Arbitrary/Custom Verbs
Some server configurations fail securely when encountering non-standard verbs but still pass the URI parameters to the application layer.
* Change the method to a random string:
  ```http
  TRACK /admin/delete?username=victim HTTP/1.1
  ```
  *(or use `OPTIONS`, `PUT`, `DELETE`)*

#### Technique C: URL Parameter Conversion
If changing `POST` to `GET` causes a missing parameters error, ensure your form parameters are moved from the request body into the URL query string:
* **Original POST:**
  ```http
  POST /admin/delete HTTP/1.1
  Host: vulnerable-website.com
  ...

  username=victim
  ```
* **Bypassed GET:**
  ```http
  GET /admin/delete?username=victim HTTP/1.1
  Host: vulnerable-website.com
  ...
  ```

### Step 4: Verify Success
1. Click **Send** in Burp Repeater.
2. Analyze the response code and content:
   * **Success:** `200 OK`, `302 Found` (redirecting to a success page), or `204 No Content`.
   * **Failure:** `403 Forbidden`, `405 Method Not Allowed`, or `400 Bad Request`.
3. Verify in the application UI that the restricted action (e.g., user deletion) was successfully executed.

---

## Remediation
To fix this vulnerability, secure the application configuration and code using these best practices:
* **Deny by Default:** Ensure access control rules apply to all HTTP methods universally unless explicitly allowed.
* **Avoid Method-Specific Restrictions:** In Java web applications (`web.xml`), avoid using `<http-method>` tags within `<security-constraint>` blocks unless absolutely necessary, or use `<http-method-omission>` to securely invert the logic.
* **Framework Routing:** Ensure the application framework strictly maps specific routes to specific methods, returning a `405 Method Not Allowed` error for any unmapped verbs.
