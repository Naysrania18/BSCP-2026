## PortSwigger Lab Solution: Username Enumeration via Different Responses## Lab Description
This lab is vulnerable to username enumeration and password brute-force attacks, allowing for user enumeration, password cracking, and unauthorized account access. The objective is to identify valid usernames by analyzing differing server responses and subsequently brute-forcing the password.
------------------------------
## Step-by-Step Solution## Phase 1: Enumerate a Valid Username

   1. Capture the Login Request
   * With Burp Suite running, navigate to the login page.
      * Submit an invalid username and password (e.g., invaliduser / password123).
      * Go to Proxy > HTTP history and locate the POST /login request.
   2. Send to Intruder
   * Highlight the value of the username parameter in the request.
      * Right-click and select Send to Intruder.
   3. Configure Intruder
   * In the Intruder tab, verify the username parameter is automatically marked as a payload position (shown with § symbols):
      
      username=§invalid-username§&password=password123
      
      * Ensure the Sniper attack type is selected.
   4. Load Username Payloads
   * Go to the Payloads side panel.
      * Ensure Simple list payload type is selected.
      * Paste the list of candidate usernames under Payload configuration.
   5. Run the Attack
   * Click Start attack and wait for the attack to complete.
   6. Analyze Results
   * Sort the results by the Length column.
      * Look for one entry with a different response length than the others.
      * Example: If the response typically says "Invalid username" and one says "Incorrect password", that specific username is valid.
   
------------------------------
## Phase 2: Brute-Force the Password

   1. Reset Payload Position
   * Close the attack results window.
      * Go back to the Intruder tab and click Clear §.
      * Change the username parameter to the valid username identified in Phase 1.
      * Add a payload position to the password parameter:
      
      username=identified-user&password=§invalid-password§
      
      2. Load Password Payloads
   * In the Payloads side panel, clear the previous username list.
      * Paste the list of candidate passwords.
   3. Run Password Attack
   * Click Start attack and wait for completion.
   4. Identify Successful Login
   * Sort results by the Status column.
      * Look for a 302 Redirect response (instead of the usual 200 OK responses).
      * Note the successful password in the Payload column.
   5. Log In
   * Use the identified username and password to log in and solve the lab.
   
------------------------------
## What Makes This Vulnerability Possible?## The Logic Flaw
The vulnerable authentication logic follows this pattern:

if username exists in database:
    if password matches:
        return "302 Redirect - Login successful!"
    else:
        return "Incorrect password"  # Information leakelse:
    return "Invalid username"  # Information leak

This flow gives attackers clear, structural clues based on which condition fails.
## How Attackers Exploit It

* Response Content Differences: The server returns distinct error messages (Invalid username vs Incorrect password).
* Response Length Differences: Messages of different lengths can be detected automatically via automation tools.
* Timing Differences: Some implementations validate the username before the password, creating measurable timing discrepancies.

------------------------------
## Impact on Websites## Security Risks

| Risk | Description |
|---|---|
| Account Takeover | Attackers identify valid accounts and target them with password brute-force attacks. |
| Credential Stuffing | Enumerated usernames are paired with breached passwords from other sites. |
| Privacy Breach | Attackers can confirm whether specific individuals hold accounts on the platform. |
| Phishing Targeting | Valid usernames are harvested for targeted social engineering attacks. |

## CVSS Severity
This vulnerability typically maps to a CVSS score of 5.3 (Medium):
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N
------------------------------
## How to Prevent Username Enumeration## 1. Use Generic Error Messages## Vulnerable Code (Java)

if (user == null) {
    return "User not found";  // Leaks information
} else if (!passwordMatches) {
    return "Wrong password";  // Leaks information
}

## Secure Code (Java)

// Always respond with the same generic messageif (user == null || !passwordMatches) {
    return "Incorrect credentials";  // Generic response
}

## Secure Code (Elixir/Phoenix)

case Accounts.get_user_by_username(username) do
  nil -> send_resp(conn, 400, "Incorrect credentials")
  user ->
    if user.password == password,
      else: send_resp(conn, 400, "Incorrect credentials")
end

## 2. Ensure Consistent Response Length

* Make error messages the exact same length regardless of which condition fails.
* Use padding or additional generic text if needed.

## 3. Maintain Consistent Response Timing

* Vulnerable: Valid usernames trigger password hashing (slower), while invalid usernames return immediately (faster).
* Secure: Always perform the same cryptographic operations regardless of username validity. Use constant-time comparison functions.

## 4. Implement Rate Limiting

* Limit failed login attempts per IP address.
* Enforce CAPTCHAs after a set number of failures.
* Implement progressively increasing delays between requests.

## 5. Additional Defensive Measures

* Account Lockout Policies: Temporarily lock accounts after consecutive failed attempts.
* Multi-Factor Authentication (MFA): Mitigates the risk of successful password brute-forcing.
* Behavioral Monitoring: Track and alert on multiple login failures across different usernames coming from the same source.

------------------------------
## Summary

| Aspect | Details |
|---|---|
| Vulnerability | Server response differs based on username validity. |
| Attack Vector | Brute-force enumeration using automated tools. |
| Primary Defense | Generic error messages like "Incorrect credentials". |
| Secondary Defenses | Consistent response length/timing, rate limiting, and account lockouts. |


