# PortSwigger Lab Solution: 2FA Simple Bypass

## Lab Description
This lab features a two-factor authentication (2FA) mechanism that can be bypassed. You are provided with valid credentials for both your account and the victim's account:
* **Your credentials:** `wiener:peter`
* **Victim's credentials:** `carlos:montoya`

The goal is to access Carlos's account page without having access to his 2FA verification code.

---

## Step-by-Step Solution

### Understanding the Vulnerability
The 2FA flow contains a critical logic flaw: the server allows direct access to protected pages without verifying that the 2FA step was completed. This issue stems from faulty **session state management**.

The intended application flow is:
1. User submits username and password → Server validates credentials.
2. Server redirects to the 2FA verification page (`/login2`).
3. User enters the security code → Access is granted to `/my-account`.

**The Flaw:** After a user completes Step 1, the application fails to strictly verify that Step 3 was completed before serving protected content.

### Exploitation Steps

#### Step 1: Log in to Your Own Account
* Navigate to the login page.
* Log in using your credentials: `wiener:peter`.
* Check the provided email client for your 2FA verification code.
* Complete the 2FA verification to observe the normal application flow.

#### Step 2: Observe the URL Pattern
* Once logged in and verified, navigate to your account page.
* Note the protected page URL: `https://[lab-url]/my-account`.

#### Step 3: Log Out
* Log out of your account completely.

#### Step 4: Log in as the Victim
* Log in using the victim's credentials: `carlos:montoya`.
* The application will prompt you for a 2FA verification code.

#### Step 5: Bypass 2FA
* When prompted for the verification code, **do not enter it**.
* Instead, manually change the URL in your browser's address bar to point directly to `/my-account`.

#### Step 6: Access the Account
* The server processes the request and loads Carlos's account page.
* You have successfully bypassed the 2FA mechanism.

---

## Technical Analysis

### The Logic Flaw
The application decouples the initial authentication check from the authorization check:

* `POST /login` → User validated → Session created
* `GET /login2` → 2FA page displayed
* `POST /login2` → 2FA code validated → Access granted to `/my-account`

The vulnerability occurs because the server grants access to `/my-account` by only checking that `POST /login` occurred, completely ignoring whether `POST /login2` was successfully processed.

### Attack Mechanics
* **Direct Path Access:** An attacker navigates directly to protected endpoints post-authentication.
* **Missing State Validation:** The server fails to enforce a "2FA verified" flag on session states.
* **Session Replay/Reuse:** The initial session cookie is treated as fully authorized prematurely.

---

## Impact

### Security Risks
* **Complete Account Takeover:** Attackers with compromised primary credentials can bypass 2FA entirely.
* **Security Control Nullification:** The secondary authentication layer is rendered useless.
* **Unauthorized Data Access:** Attackers gain immediate access to sensitive user data.
* **Privilege Escalation:** If administrative accounts use this flow, attackers can gain full system control.

### Severity
* **Critical:** This flaw negates the security benefits of MFA, reducing system security back to simple single-factor authentication.

---

## Remediation & Prevention

### 1. Maintain Proper Session State
Do not grant access based solely on the primary authentication status.

**Vulnerable Implementation Example:**
```python
if user_authenticated:
    grant_access_to("/my-account")  # Only checks login
```

**Secure Implementation Example:**
```python
if user_authenticated and user_2fa_verified:
    grant_access_to("/my-account")
else:
    redirect_to("/login2")  # Enforce 2FA completion
```

### 2. Validate Multi-Factor Status per Request
* Implement a explicit session variable (e.g., `is_mfa_completed = true`) only after successful second-factor validation.
* Verify this specific variable on every restricted endpoint.
* Never trust client-side state or easily manipulated cookies to track MFA status.

### 3. Implement Strict Session Binding
* Tie the 2FA challenge directly to the specific lifecycle of the initial login session.
* Expire the incomplete session if the 2FA step is abandoned or times out.

### 4. Defense-in-Depth
* **Rate Limiting:** Restrict the number of 2FA verification attempts to prevent brute-forcing.
* **Account Lockout:** Temporarily lock accounts or sessions after consecutive failed 2FA entry attempts.
* **Anomaly Detection:** Monitor for instances where restricted endpoints like `/my-account` are accessed without a preceding sequence from `/login2`.

---

## Summary

| Component | Details |
| :--- | :--- |
| **Vulnerability** | 2FA bypass via direct navigation. |
| **Attack Vector** | Log in, then manually change the URL to `/my-account`. |
| **Root Cause** | Missing server-side verification of the 2FA state flag. |
| **Primary Defense** | Enforce a strict `user_2fa_verified` check on all protected pages. |
| **Secondary Defenses** | Rate limiting, session timeouts, and sequence monitoring. |
