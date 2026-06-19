PortSwigger Lab Solution: Username Enumeration via Different Responses
Lab Description
This lab is vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password. The goal is to enumerate a valid username, brute-force the password, and access the account page .

Step-by-Step Solution
Phase 1: Enumerate a Valid Username
Step 1: Capture the Login Request

With Burp Suite running, navigate to the login page and submit an invalid username and password (e.g., invaliduser / password123)

Go to Proxy > HTTP history and locate the POST /login request 

Step 2: Send to Intruder

Highlight the value of the username parameter in the request

Right-click and select Send to Intruder 

Step 3: Configure Intruder

In the Intruder tab, verify the username parameter is automatically marked as a payload position (shown with § symbols):

text
username=§invalid-username§&password=password123
Ensure Sniper attack type is selected 

Step 4: Load Username Payloads

Go to the Payloads side panel

Ensure Simple list payload type is selected

Paste the list of candidate usernames under Payload configuration 

Step 5: Run the Attack

Click Start attack

Wait for the attack to complete

Step 6: Analyze Results

Sort the results by the Length column

Look for one entry with a different response length than the others 

Examine the response: most entries will show Invalid username, but the anomalous one will show Incorrect password 

Example: If the response typically says "Invalid username" and one says "Incorrect password", that username is valid 

Phase 2: Brute-Force the Password
Step 1: Reset Payload Position

Close the attack results window

Go back to the Intruder tab and click Clear §

Change the username parameter to the valid username you identified

Add a payload position to the password parameter:

text
username=identified-user&password=§invalid-password§
``` [citation:1]
Step 2: Load Password Payloads

In the Payloads side panel, clear the username list

Paste the list of candidate passwords 

Step 3: Run Password Attack

Click Start attack

Wait for completion

Step 4: Identify Successful Login

Sort results by the Status column

Look for a 302 redirect response (instead of 200 responses) 

Note the password in the Payload column

Step 5: Log In

Use the identified username and password to log in and solve the lab 

What Makes This Vulnerability Possible?
The Logic Flaw
The vulnerable authentication logic follows this pattern:

python
if username exists in database:
    if password matches:
        return "302 Redirect - Login successful!"
    else:
        return "Incorrect password"  # Different message
else:
    return "Invalid username"  # Different message
This gives attackers different clues based on which condition fails .

How Attackers Exploit It
Response Content Differences: The server returns different error messages (Invalid username vs Incorrect password) 

Response Length Differences: Messages of different lengths can be detected automatically 

Timing Differences: Some implementations validate username before password, creating measurable timing discrepancies 

Impact on Websites
Security Risks
Risk	Description
Account Takeover	Attackers identify valid accounts and target them with password brute-force attacks 
Credential Stuffing	Enumerated usernames can be combined with breached passwords from other sites 
Privacy Breach	Attackers can confirm whether specific users have accounts on the platform
Phishing Targeting	Valid usernames can be used for targeted social engineering attacks
CVSS Severity
This vulnerability typically has a CVSS score of 5.3 (Medium) with the vector:
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N 

How to Prevent Username Enumeration
1. Use Generic Error Messages
Vulnerable Code (Java):

java
if (user == null) {
    return "User not found";  // Leaks information
} else if (!passwordMatches) {
    return "Wrong password";  // Leaks information
}
Secure Code (Java):

java
// Always respond with the same generic message
if (user == null || !passwordMatches) {
    return "Incorrect credentials";  // Generic response
}
``` [citation:3]

**Secure Code (Elixir/Phoenix)**:
```elixir
case Accounts.get_user_by_username(username) do
  nil -> send_resp(conn, 400, "Incorrect credentials")
  user ->
    if user.password == password,
      else: send_resp(conn, 400, "Incorrect credentials")
end
``` [citation:8]

### 2. Ensure Consistent Response Length

- Make error messages the **exact same length** regardless of which condition fails
- Use padding or additional generic text if needed [citation:3]

### 3. Maintain Consistent Response Timing

- **Vulnerable**: Valid username triggers password hashing (slower), invalid username returns immediately (faster) [citation:2][citation:7]
- **Secure**: Always perform the same operations regardless of username validity
- Compare with `bcrypt` or use constant-time comparison functions

### 4. Implement Rate Limiting

- Limit failed login attempts per IP address
- Use CAPTCHA after a certain number of failures
- Implement progressively increasing delays [citation:3]

### 5. Additional Defensive Measures

- **Use account lockout policies** (temporary or permanent)
- **Implement multi-factor authentication** (MFA)
- **Monitor for suspicious login patterns** (multiple attempts, different usernames from same IP)

---

## Summary

| Aspect | Details |
|--------|---------|
| **Vulnerability** | Server response differs based on username validity |
| **Attack Vector** | Brute-force enumeration using automated tools |
| **Primary Defense** | Generic error messages like "Invalid username or password" |
| **Secondary Defenses** | Consistent response length, consistent timing, rate limiting, account lockouts |
