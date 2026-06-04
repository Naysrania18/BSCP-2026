 
## Lab: Stored XSS into anchor href attribute with double quotes HTML-encoded  
**Difficulty:** Apprentice  
**Category:** Cross-Site Scripting (XSS)

---

## 📝 Lab Description

This lab contains a **Stored Cross-Site Scripting (XSS)** vulnerability in the comment functionality.

Your goal is to:
> Submit a comment that triggers `alert()` when the **comment author name is clicked**.

---

## 🎯 Objective

Inject a payload into the stored comment data so that clicking the author’s name executes JavaScript.

---

## 🔍 Vulnerability Analysis

When a user submits a comment, the application renders the author name as a hyperlink using the **Website field**:

```html
<a id="author" href="USER_INPUT">Author Name</a>
