# Vulnerability Assessment: Broken Authentication via Brute-Force

- Vulnerability Tittle : Broken Authentication / Missing Rate Limiting

- OWASP Top 10 Risk : A07:2021-Identification and Authentication Failures

- Severity : High

- Target URL : http://10.49.180.226/login.php

### Description

The primary authentication interface lacks protection against automated credential stuffing and brute-force attacks. The system does not enforce account lockout policies after multiple failed login attempts, nor does it implement rate-limiting or anti-automation mechanisms (e.g., CAPTCHA).

### Attack Configuration And Session Handling

- Attack Type : Sniper (targeting the `password` parameter).

- Session Handling Rule :
  - Added a custom rule description named `Override CSRF token` invoked on every      request within the scope.
  - Added a custom rule action ("Run a macro") invoked on every request within        the defined tool scope.

- Scope Check : Target & Intruder.

- Macros Configuration : Executed a macro to fetch a fresh and valid Anti-CSRF token from the login response prior to each login attempt.

- Parameter Handling : Configured parameter updating to automatically match and replace the CSRF token parameter name in the request body to bypass the token validation. 

- Redirections : Always

- Payload Type : Simple List Custom password [wordlist Passwords](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/100k-most-used-passwords-NCSC.txt)

### Exploitation Evidence (Proof Of Concept) :

The brute-force attack successfully bypassed the Anti-CSRF protection. By configuring Burp Suite to always follow redirections, the successful login attempt was clearly identified by a successful HTTP Status Code (200 OK) and a unique response length.

<img width="1463" height="668" alt="Pasted image 20260518234230" src="https://github.com/user-attachments/assets/6d8a9267-4ff5-463f-99ad-b12190a81c57" />

*the screenshot above shows CSRF Token is incorrect because it was taken before configuring*

After obtaining the valid credentials through this method, unauthorized access to the DVWA admin dashboard was fully achieved.

<img width="1437" height="567" alt="Pasted image 20260519003726" src="https://github.com/user-attachments/assets/c571287e-ec70-4955-974c-461892321845" />

the last step is to enter the username and password that we already know.

*if there is an invalid CSRF token or the like, just refer to the login page*

### Remediation & Mitigation :

To secure the authentication interface against brute-force attacks, the following mitigations should be implemented:
1. Implement Account Lockout Policy: Temporarily lock accounts (e.g., for 15 minutes) after 5 consecutive failed login attempts.
2. Implement Rate Limiting: Limit the number of login requests a single IP address can make within a specific timeframe.
3. Integrate CAPTCHA: Introduce an anti-automation mechanism like reCAPTCHA on the login page to block automated tools like Burp Suite.
