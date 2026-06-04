# PortSwigger Web Security Academy Writeup

## Lab: Stored XSS into anchor href attribute with double quotes HTML-encoded

**Difficulty:** Apprentice  
**Category:** Cross-Site Scripting (XSS)

---

## 📝 Lab Description

This lab contains a stored cross-site scripting vulnerability in the comment functionality. To solve this lab, submit a comment that calls the `alert()` function when the comment author name is clicked.

---

## 🎯 Objective

Inject a JavaScript payload into the database via the comment section so that clicking the comment author's name triggers an alert popup.

---

## 🔍 Vulnerability Analysis & Methodology

### Identifying the Injection Point

When posting a comment, the application provides four fields:

- Comment  
- Name  
- Email  
- Website  

Upon submission, the application renders the author's name as a clickable hyperlink using the Website input inside an HTML anchor tag:

```html
<a id="author" href="USER_INPUT">Author Name</a>
Analyzing the Defense

If you attempt a traditional attribute breakout using double quotes (e.g., " onmouseover="alert(1)), the application HTML-encodes the characters into &quot;.

The resulting code looks like this:

<a id="author" href="&quot; onmouseover=&quot;alert(1)">Author Name</a>

Because the quotes are broken into literal text, we cannot break out of the href attribute.

The Bypass Strategy

Since we are confined inside the href attribute, we can leverage the javascript: pseudo-protocol.

Browsers treat javascript: inside an href as executable code rather than a standard web URL when the link is clicked.

This allows us to execute JavaScript completely within the attribute boundaries without needing any double quotes.

🛠️ Exploit Steps
Step 1: Submit the Malicious Comment

Navigate to any blog post on the lab website.
Scroll down to the Leave a comment section.

Fill out the fields with the following details:

Comment: Test Comment
Name: Attacker
Email: attacker@test.com

Website (Crucial Field):

javascript:alert(1)

Click Post Comment.

Step 2: Verify the Code Injection

Go back to the blog post page and locate your comment.
Right-click your comment name (Attacker) and select Inspect.

Verify that the payload is safely embedded inside the DOM without being broken by encoding:

<a id="author" href="javascript:alert(1)">Attacker</a>
Step 3: Trigger the Payload

Click directly on the comment author's name (Attacker) on the live webpage.

The browser will execute the javascript: link, causing an alert(1) popup to appear.

The lab status will update to Solved.

🛡️ Remediation (How to Fix)

To secure the application against this specific variant of Stored XSS:

URL Validation

Implement a strict allowlist for the Website field. Ensure the input must begin exclusively with safe protocols like:

http://
https://
Reject Pseudo-Protocols

Explicitly block inputs beginning with:

javascript:
data:
Context-Aware Encoding

Use context-aware output encoding to ensure user-supplied URLs cannot execute code in an attribute context.
