# PortSwigger Lab: Reflected XSS into a JavaScript string with angle brackets HTML encoded

## Lab Information
* **Difficulty:** Apprentice
* **Vulnerability Type:** Reflected Cross-Site Scripting (XSS)
* **Context:** Input is reflected inside an existing JavaScript string literal, but angle brackets (`<` and `>`) are HTML-encoded by the application.

---

## Objective
Break out of the JavaScript string context and successfully execute the `alert()` function without relying on HTML tags.

---

## Walkthrough / Solution Steps

### Step 1: Analyze the Reflection Context
1. Open the lab in your browser.
2. In the search box, enter a unique, random alphanumeric string (e.g., `test123xyz`) and click **Search**.
3. Open **Burp Suite**, navigate to the **Proxy > HTTP history** tab, and locate the search request.
4. Right-click the request and select **Send to Repeater**.

### Step 2: Observe the Vulnerability in Burp Repeater
1. Go to the **Repeater** tab and send the request.
2. Search the response body for your random string (`test123xyz`).
3. Notice that your input is reflected directly inside a JavaScript block, contained within a single-quoted string literal:
   ```javascript
   var searchTerms = 'test123xyz';
   ```
4. Note that while angle brackets (`<`, `>`) are securely HTML-encoded, the application fails to escape or sanitize single quotes (`'`).

### Step 3: Craft and Inject the Payload
To trigger the XSS, we must break out of the string boundary using single quotes and maintain valid JavaScript syntax using math operators.

1. Replace the value of the search parameter in Burp Repeater with the following payload:
   ```javascript
   '-alert(1)-'
   ```
2. Send the request. The code will now reflect inside the server response like this:
   ```javascript
   var searchTerms = ''-alert(1)-'';
   ```

#### Code Breakdown:
* The first single quote (`'`) closes the developer's original string literal early.
* The subtraction operators (`-`) treat the payload as a mathematical expression, forcing the browser engine to evaluate and execute the `alert(1)` function.
* The trailing single quote (`'`) safely pairs up with the developer's closing quote, preventing syntax errors that would crash execution.

### Step 4: Verify and Solve the Lab
1. Right-click anywhere inside the Burp Repeater request window and select **Copy URL**.
2. Paste this copied URL directly into your browser's address bar and hit **Enter**.
3. The page will load, bypass the HTML encoding restrictions, and trigger an explicit browser **alert pop-up**.
4. The lab status will instantly update to **Solved**!
