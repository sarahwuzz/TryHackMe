first time i always used nmap to scanning the port

ip target : `10.49.180.226`

i used this command : `nmap -sV -sC -Pn 10.49.180.226`

```bash
Nmap scan report for 10.49.180.226
Host is up (0.21s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 19:05:31:bd:ba:fb:99:61:ec:4d:ca:1b:aa:3d:fa:64 (DSA)
|   2048 9f:fc:82:d8:d0:94:57:b7:50:0b:0d:9c:d2:8c:f5:8d (RSA)
|   256 1d:4c:07:af:20:48:40:a5:11:71:c8:48:97:9c:ff:6c (ECDSA)
|_  256 d0:48:56:0b:4f:14:b2:0b:3e:d5:5c:59:7a:e6:33:9f (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-title: Login :: Damn Vulnerable Web Application (DVWA) v1.10 *Develop...
|_Requested resource was login.php
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
