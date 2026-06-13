# PortSwigger Lab Writeup: URL-based access control can be circumvented

## Lab Details
- **Vulnerability Category:** Broken Access Control
- **Difficulty:** Practitioner
- **Objective:** Access the unauthenticated admin panel and delete the user `carlos`.
- **Target Vulnerability:** Frontend reverse proxy / WAF relies on URL-based restrictions, while the backend web framework supports the `X-Original-URL` header to override routing.

---

## Vulnerability Explanation
Websites often use frontend servers (like reverse proxies or Web Application Firewalls) to enforce access control rules at the platform layer (e.g., block all external traffic to `/admin`). 

However, some backend frameworks support non-standard HTTP headers like `X-Original-URL` or `X-Rewrite-URL`. If a backend framework interprets this header, it overrides the URL routing internally. 

An attacker can send a request to an allowed frontend path (like `/`) but inject `X-Original-URL: /admin`. The frontend allows the request because it only checks `/`, but the backend processes it as a request for `/admin`, circumventing the access control entirely.

---

## Step-by-Step Walkthrough

### Step 1: Observe the Frontend Restriction
1. Access the lab link given by PortSwigger.
2. In your browser bar, append `/admin` to the lab URL (e.g., `https://<your-lab-id>.web-security-academy.net/admin`).
3. Notice that the page returns an **"Access denied"** error message. This restriction is enforced by the frontend system.

### Step 2: Intercept and Test Header Support
1. Open **Burp Suite** and ensure **Intercept is On** (or capture the traffic via HTTP History).
2. Refresh the homepage of the lab (`/`) to capture a clean `GET / HTTP/1.1` request.
3. Right-click the captured request and select **Send to Repeater** (or press `Ctrl + R`).
4. Navigate to the **Repeater** tab.
5. Modify the request by adding a custom header to verify if the backend recognizes it:
   ```http
   GET / HTTP/1.1
   Host: <your-lab-id>.web-security-academy.net
   X-Original-URL: /invalid-route-test
   ...
   ```
6. Click **Send**. If the response returns a **404 Not Found**, it confirms that the backend is processing the `X-Original-URL` header and looking for `/invalid-route-test` instead of the root `/`.

### Step 3: Bypass Access Control to View the Admin Panel
1. Modify the `X-Original-URL` value in your Repeater request to target the protected path:
   ```http
   GET / HTTP/1.1
   Host: <your-lab-id>.web-security-academy.net
   X-Original-URL: /admin
   ```
2. Click **Send**.
3. You will receive an HTTP **200 OK** containing the HTML code for the Admin Panel page. 
4. Review the HTML body to find the specific URL parameter structure used to delete users. Look for the string `delete?username=carlos`.

### Step 4: Delete the User Carlos to Solve the Lab
1. Craft the final exploit request in **Burp Repeater** by passing the deletion endpoint into the header query parameter:
   ```http
   GET / HTTP/1.1
   Host: <your-lab-id>.web-security-academy.net
   X-Original-URL: /admin/delete?username=carlos
   ```
2. Click **Send**.
3. The lab interface will refresh and update with a **"Congratulations, you solved the lab!"** banner.

---

## Remediation / Prevention
* **Never rely solely on URL-based access control** implemented at the frontend layer.
* Enforce rigorous access control mechanics directly on the server-side/backend application layout.
* Configure the frontend proxy or web server to completely strip out or ignore non-standard routing headers like `X-Original-URL` and `X-Rewrite-URL` before passing requests to downstream backends.
