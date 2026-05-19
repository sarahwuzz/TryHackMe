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

okay, so we got 2 port open, ssh and http,
and i will check the http on browser

<img width="1266" height="547" alt="Pasted image 20260518220509" src="https://github.com/user-attachments/assets/d5c92c4e-09c4-488a-bf41-6cbaf94c5a95" />

yeah we got this website, but we don't know what they username and password, so i will open burpsuite

<img width="1590" height="523" alt="Pasted image 20260518224859" src="https://github.com/user-attachments/assets/02d5195e-e2b1-499a-ba4c-362e8aff0fb6" />

and in proxy http history, we got this version disclosure again, same as nmap, but in burpsuite is more detailed, even we got the php version, you can search this version in CVE if you want to know they vulnerability.

cause here the purpose is want to deploy the machine, so i didn't search their vulnerability, i'm searching what their username and password, cause i want to login and deploy the machine.

so i try to login with common username like admin, and common password like admin

<img width="541" height="468" alt="Pasted image 20260518225850" src="https://github.com/user-attachments/assets/a4ae0b3e-bfca-4415-81f1-6e3e8a0a63d2" />

and login must failed cause i don't know the username and password.
after failing to login because I was just guessing, we check the burp suite again, there's something new

<img width="1593" height="435" alt="Pasted image 20260518230557" src="https://github.com/user-attachments/assets/1c60b51e-9ba4-436f-94ee-6b7304cd8379" />

if earlier we just have GET, now we got a POST, and i will try to do brute force use list common username and common password in intruder

<img width="1589" height="543" alt="Pasted image 20260518231547" src="https://github.com/user-attachments/assets/918fa0d0-e13a-4109-b2bd-2095c17abb2f" />

we can change the username and password in intruder use add, but i don't want to change the username i will try with admin, and i wanna change the password.
if you confused what password is commonly people use, for the first you can just search in google

<img width="1098" height="544" alt="Pasted image 20260518232253" src="https://github.com/user-attachments/assets/87f9b0bb-8fbc-489b-bee1-5c46d89d28d5" />

and i will try with these list, by manually adding the payload

<img width="1401" height="506" alt="Pasted image 20260518232614" src="https://github.com/user-attachments/assets/17d397dd-e9f0-4e05-a236-0211fa21d64e" />

and let's try to attack!

<img width="1425" height="513" alt="Pasted image 20260518232947" src="https://github.com/user-attachments/assets/1deaffc5-48fa-41a0-8b99-5ac9ffa96abf" />

here the problem, cause all of this status code is 302, we don't know which one is correct, so we should do a bit configuration.

<img width="1401" height="482" alt="Pasted image 20260518233857" src="https://github.com/user-attachments/assets/c6d5f7d9-9b7c-4e44-95a0-e28f91be2ecb" />

we try to change the redirections to always and we start to attack again

<img width="1463" height="668" alt="Pasted image 20260518234230" src="https://github.com/user-attachments/assets/6d8a9267-4ff5-463f-99ad-b12190a81c57" />

here we already changed the status code to be 200, it's good, but... CSRF token is incorrect, so we should get the CSRF token and do a configuration.
first we should add a macros

<img width="1158" height="352" alt="Pasted image 20260519000941" src="https://github.com/user-attachments/assets/ee4c30d1-beef-413c-8055-062537c44d8e" />

<img width="780" height="701" alt="Pasted image 20260519001105" src="https://github.com/user-attachments/assets/3d54a84b-95b6-4c14-8be1-83eefb086a75" />

and we just click ok and ok again.
and then we go to session handling and add

<img width="824" height="347" alt="Pasted image 20260519001253" src="https://github.com/user-attachments/assets/33e1025f-70f9-41a4-9f5a-64406621db8e" />

in session handling editor we should add rule description, like this:

<img width="618" height="350" alt="Pasted image 20260519001442" src="https://github.com/user-attachments/assets/1163e4e1-6a1e-4760-ab34-762aa1354aac" />

and add rule actions, choose run a macros

cause the  parameter name is user_token, how do we know the  parameter name is user_token? here:
(we can see it on view-source login page)

<img width="506" height="55" alt="Pasted image 20260519001756" src="https://github.com/user-attachments/assets/f1b62620-d79d-4b16-b2d2-81d79dce70fd" />

where we put that parameter name? here:

<img width="724" height="620" alt="Pasted image 20260519001921" src="https://github.com/user-attachments/assets/2fd90541-a8c7-4d3f-922e-07357c852dc2" />

make sure you've also clicked the macros (just click on macro 3) and click ok.
after that, we click to the scope section and check only intruder and target, dont forget to check include all URLs
and let's go to try attack again!

<img width="1437" height="567" alt="Pasted image 20260519003726" src="https://github.com/user-attachments/assets/c571287e-ec70-4955-974c-461892321845" />

and BOOOMM we got the password!
how do we know this is the password?
cause in response we don't have "login failed" while the other have it.
so just try to login!

<img width="454" height="455" alt="Pasted image 20260519004003" src="https://github.com/user-attachments/assets/b88c8f93-7795-4cec-bc53-27c10780b5fc" />

i used that password, but i can't login cause CSRF token is incorrect.
but don't worry, just refresh the page, and then try to login again

<img width="901" height="802" alt="Pasted image 20260519004321" src="https://github.com/user-attachments/assets/2dd168ea-93f1-4f3f-9e78-5e2e4bea2d81" />

we're done!
Thank you all. I apologize if there are any mistakes or if my explanations aren’t clear, because I’m still learning myself.
