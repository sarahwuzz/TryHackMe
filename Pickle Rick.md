# Vulnerability Assessment : OS Command Injection

- Vulnerability Tittle : OS Command Injection

- OWASP Top 10 Risk : A03:2021-Injection

- CWE ID :  CWE-78 Improper Neutralization of Special Elements used in an OS Command

- CVSS Score :  CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H - 9.8

- Severity : Critical

- Target URL : http://10.48.180.50 & http://10.48.180.50/login.php

### Description

During the security assessment on the Pickle Rick, an OS Command Injection vulnerability was discovered within the web application's command panel. The vulnerability occurs due to a lack of input sanitization and filtering on the user-supplied input fields.
Instead of validating the input, the application directly passes the user's string into a dangerous system execution function (such as `system()` or `shell_exec()`), allowing it to be executed directly by the target's underlying operating system shell. An attacker can exploit this flaw by injecting system commands (e.g., `whoami`, `ls -la`, or network commands like `ping`) to enumerate the system, view sensitive source files, and ultimately execute a reverse shell to gain unauthorized Remote Code Execution (RCE).

### Attack Configuration

- Target Web Discovery: Accessed the target web application and inspected the page source code, revealing a    hidden username comment `R1ckRul3s`.

- Directory Fuzzing: Conducted web directory enumeration using FFUF *(or others tools you usually used)*       with .php and .txt extensions, successfully discovering login.php and robots.txt.

- Credential Gathering: Analyzed robots.txt which contained a cleartext string used as the login password      for the administrative portal.

- Authentication: Navigated to login.php and successfully authenticated using the discovered credentials,      granting access to the web command panel.
