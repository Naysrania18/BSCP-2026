# PortSwigger Web Security Academy Lab Write-up.

## Lab Name: Reflected XSS into attribute with angle brackets HTML-encoded
**Difficulty:** Apprentice  
**Category:** Cross-Site Scripting (XSS)

---

### 📋 Lab Description
This lab contains a reflected cross-site scripting (XSS) vulnerability in its search blog functionality. The application implements a defensive filter where angle brackets (`<` and `>`) are HTML-encoded, making traditional script tags ineffective. 

**Objective:** Solve the lab by performing a cross-site scripting attack that injects a new attribute to call the `alert()` function.

---

### 🔍 Vulnerability Analysis & Context Discovery

1. **Initial Vector Testing:**
   Inputting a standard script payload like `<script>alert()</script>` into the search bar fails to execute. Inspecting the source code reveals that the browser HTML-encodes the angle brackets into safety entities:
   ```html
   &lt;script&gt;alert()&lt;/script&gt;
   ```
   This prevents the browser from interpreting the input as an executable HTML tag.

2. **Analyzing the Reflection Context:**
   By analyzing the page source after searching for a random string (e.g., `TESTING123`), the input reflection point is discovered inside the `value` attribute of an existing HTML `<input>` element:
   ```html
   <input type="text" placeholder="Search this blog..." name="search" value="TESTING123">
   ```

3. **Identifying the Bypass Strategy:**
   * The server filters `<` and `>`, but it **does not** filter or encode double quotes (`"`).
   * Since we are already inside an HTML tag, we do not need angle brackets. We can break out of the `value` attribute using a double quote (`"`) and inject new event-driven attributes directly into the existing `<input>` tag.

---

### 🚀 Exploitation & Solution Steps

1. Navigate to the blog's search functionality.
2. Enter the following payload into the search input box:
   ```text
   " autofocus onfocus="alert(1)
   ```
3. Click **Search** or hit enter.

#### How the Payload Works (Code Execution Mechanics):
Once submitted, the backend server reflects the payload into the input container, reconstructing the HTML DOM as follows:
```html
<input type="text" placeholder="Search this blog..." name="search" value="" autofocus onfocus="alert(1)">
```
* **`"` (Double Quote):** Successfully closes the original `value="` attribute.
* **`autofocus`:** A boolean attribute that forces the browser to automatically direct keyboard/mouse focus to this specific input field as soon as the page loads.
* **`onfocus="alert(1)"`:** An event handler that triggers the JavaScript engine to execute `alert(1)` the exact millisecond the element receives focus.

Because `autofocus` immediately targets the input element on page load, the `onfocus` event fires automatically, executing the JavaScript payload without requiring manual user interaction.

---

### 🛡️ Remediation & Mitigation
To fix this vulnerability, the web application should implement **Context-Aware Output Encoding**. 
* Since the user input is reflected inside an HTML attribute context, the application must HTML-entity encode quotes (`"` to `&quot;` and `'` to `&#39;`) alongside angle brackets before rendering the data to the browser. This prevents users from breaking out of attribute strings.
