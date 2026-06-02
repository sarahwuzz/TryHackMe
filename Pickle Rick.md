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

<img width="1225" height="307" alt="Pasted image 20260601085635" src="https://github.com/user-attachments/assets/e67ad789-f685-4c0f-9f24-ce4d4c56a580" />

- Vulnerability Testing: Tested the command panel for OS Command Injection by executing basic system commands such as id, whoami, or  network utility tools like ping -c 4 127.0.0.1. The web application immediately returned the system output with www-data privileges, confirming the vulnerability.

- Payload Selection: Since the initial Nmap scan revealed the target web server runs on Apache, a PHP-based reverse shell payload was selected from the [Pentestmonkey reverse shell](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) guide. The payload was configured with the attacker machine's IP address and listening port.

- Exploitation and Initial Access: Established a Netcat listener on the attacker machine, then executed the PHP reverse shell command through the vulnerable web panel.

### Exploitation Evidence (Proof Of Concept)

After executing the payload, the Netcat listener successfully caught the connection, granting initial access to the target system. To verify the successful exploitation and provide evidence, basic enumeration was conducted directly through the established reverse shell session.

As shown in the evidence section below, running the whoami command confirms that the current session is running under the www-data user account, and the pwd command indicates the initial working directory on the server.

<img width="537" height="165" alt="Pasted image 20260601093132" src="https://github.com/user-attachments/assets/aa7b06e0-fa11-453b-a10e-687a1d5720fc" />

#### Additional Information

- Locating the First Ingredient: After gaining the shell session, the initial directory was listed. A file named Sup3rS3cr3tPickl3Ingred13nt.txt was discovered right in the web root folder. Reading this file revealed the first ingredient needed for the challenge.

- Locating the Second Ingredient: Navigated through the system files to the home directory of the user Rick, located at /home/rick. Inside this directory, a file named second ingredients was found, which contained the second required ingredient.

### Remediation & Mitigation

- Avoid Direct System Execution Functions: The most effective defense is to completely avoid passing user-supplied input directly into dangerous system interpreters or execution functions such as PHPs system, exec, shell_exec, or passthru. Utilize built-in application programming interfaces (APIs) instead of operating system commands wherever possible.
  
 -  Implement Strict Input Whitelisting: If system commands are absolutely necessary, implement a strict whitelist that only permits explicitly allowed, safe alphanumeric characters. Reject any input containing command separation characters and metacharacters, such as semicolons, ampersands, vertical bars, dollar signs, brackets, backticks, and newlines.

 -  Apply Input Sanitization and Parametrization: Use robust, built-in programming libraries to sanitize inputs before they reach the system. In environments where commands must be executed, arguments should be strictly separated from the command itself (parameterized) to ensure the system treats user input strictly as data, not as executable code.

 -  Enforce the Principle of Least Privilege: Run the web server process (e.g., Apache or Nginx) under a highly restricted, non-privileged service account such as a properly hardened www-data. Ensure this user has no access to powerful system binaries, cannot read sensitive configuration files, and is blocked from executing outbound network connections (like reverse shells) through firewall rules.

# Vulnerability Assessment : Privilege Escalation via Sudo Misconfiguration

- Vulnerability Tittle: Privilege Escalation via Sudo Misconfiguration `su`

- OWASP Top 10 Risk : A04:2021 - Insecure Design

- CWE ID : CWE-269: Improper Privilege Management

- CVSS : CVSS:3.1/AV:L/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H – 8.8

- Severity : High

- `/bin/su`

### Description

During the post-exploitation enumeration phase, a local privilege escalation vulnerability was discovered due to a security misconfiguration in the Linux sudo privileges configuration. The `su` binary was found to be executable via sudo without requiring a password, allowing it to execute with the privileges of the superuser (which is `root`).

Under standard security practices, administrative tools like `su` should never be allowed to run via sudo without password authentication for a low-privilege service account. An attacker with low-privilege access (such as `www-data`) can abuse this misconfiguration by executing the binary under the sudo context. When this command is executed via the misconfigured sudo rules, it immediately spawns a terminal with root privileges, leading to a complete system compromise.

### Attack Configuration

- Local Sudo Privilege Enumeration :

  After gaining initial access as the www-data user, a local enumeration command was executed to identify misconfigured privileges and commands that can be run via sudo:

  ```bash
  find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
  ```

- Following the binary checks, the `sudo -l` command was executed to evaluate the current users sudo permissions. The output revealed that the www-data user had the highest level of clearance, configured as `ALL NOPASSWD ALL`. This critical misconfiguration allowed the execution of any administrative command via sudo without password authentication, enabling the use of `sudo su` to instantly spawn a root shell.

### Exploitation Evidence (Proof Of Concept)

To verify that full administrative control had been achieved, standard identity check commands (`whoami` and `id`) were run, confirming a successful transition from `www-data` to `root`.

<img width="598" height="333" alt="image" src="https://github.com/user-attachments/assets/6eb9672f-d066-47cd-a0ea-2bed4a499d8f" />

*(this screecshot i took from my obsidian,cause the screecshot from shell is missing.)*  
  
### Remediation & Mitigation

- Restrict Insecure Sudo Privileges: Conduct a strict audit of the sudoers configuration file and remove or restrict excessive permissions granted to low-privileged service accounts that do not require administrative access.

Thank you!
