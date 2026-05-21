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

