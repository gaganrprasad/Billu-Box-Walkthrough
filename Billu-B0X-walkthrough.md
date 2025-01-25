# Billu: Box Walkthrough

## Target Information

- **Target IP**: '192.168.227.83' 
- **CTF Type**: Web Exploitation & Privilege Escalation
- **Difficulty Level**: Beginner to Intermediate
- **Objective**: Gain root access to the Billu: Box machine.

### Table of Contents
1. [Introduction](#introduction)
2. [Reconnaissance](#reconnaissance)
   - [Network Discovery](#network-discovery)
   - [Port Scanning](#port-scanning)
3. [Enumeration](#enumeration)
4. [Exploitation](#exploitation)
   - [Initial Access](#initial-access)
5. [Privilege Escalation](#privilege-escalation)
6. [Post-Exploitation](#post-exploitation)
7. [Conclusion](#conclusion)

## Introduction
Billu: Box is a beginner/intermediate-level VM that is designed to help you practice web-based exploitation techniques and privilege escalation. This walkthrough will guide you through reconnaissance, exploitation, and privilege escalation to capture the flag on the Billu: Box machine.

## Reconnaissance
### Network Discovery
The first step in the reconnaissance phase is to discover the network and identify the target machine. We can
use tools like `netdiscover` to scan the network and identify the target machine's IP, below is the command to do so
1.Open a terminal in Kali Linux.(open as a root user or use the command 'sudo su -')
2.netdiscover

 After running the command, I identified the target machine's IP, which was '192.168.227.83'

 ### Port Scanning
Next, we perform an Nmap scan to identify open ports and services running on the target.

1.nmap -sC -sV 192.168.227.83
For this case, I am using Zenmap, a GUI version of Nmap. The scan shows us that there are two ports open:

Port 22 - Used for SSH
Port 80 - Used to serve a web application

## Enumeration
Let's head to its port 80 and see what's the web application . So it looks like a custom page which is asking for a username and password .
I tried performing sql injection using tools like "sqlmap" and "burpsuite" and even some knowm parameters manually but was out of luck ,and none worked.
next used a tool called "dirb" for directory enumeration and also on dirbuster (NOTE: it is always a good practice to use multiple tools rather than to just rely on a sinlge one), follow the commands below:

->dirb http://192.168.227.83/  
and i came across an interesting directory "/in.php"
when i visited it was page named "phpinfo()" which contained my info including version info etc..
Let's keep it for now. This can come handy in future. The next link I come across is /test.php
When opened, this is what it says: " 'file' parameter is empty. Please provide file path in 'file' parameter"

## Exploitation

From the looks of it i can say that file is a variable sent via POST request and it may be vulnerable to LFI (local file inclusion). so i thought of trying it out
I sent a POST request by using a simple cURL command:
->curl -X POST --data "file=/etc/passwd" http://192.168.227.83/test.php
and yes it is vulnerable to LFI!

Now , Since i know that index.php asks for username and password and a POST request is being made, it is safe to say that the rest of the PHP code would be in the same file. Since i just came to know that we have an LFI vulnerability, let's see if i can exploit that to read the code of index.php.

->curl -X POST --data "file=index.php" http://192.168.227.83/test.php
We can see that a file called "c.php" is being included in the code.
so lets try to curl the c.php file

-> curl -X POST --data "c.php" http://192.168.227.83/test.php 
after running this command i found credentials for MYSQL in the line "$conn = mysqli_connect("127.0.0.1","billu","b0x_billu","ica_lab");"
Username: billu
Password: b0x_billu

## Initial Access
i thought i could login using these credentials in the website found earlier but couldn't so i head back to dirbuster and there i found a directory "/phpmy" in dirbuster tool
(I couldn't find this directory in "dirb" results)
then i used the above credentials to login to PHPMyAdmin and was successfull

Here under "ica_lab" under "auth" section i found the username and password and used it to login to the website and was successfull
user and pass: biLLu , hEx_it

But found nothing interesting, then i inspected the webpage but it was of no use 
was stuck here for a while 
then i suddenly remembered that i was not logged in to phpmyadmin as root and was a normal user (i also came to know and confirmed this by looking into the information schema and found a table named USER_PRIVILEGES and confirmed i was just a user and not root )
so i thought i could look into the config file of this php and could find something 
and yehh i used curl to get the config file , below is the command :

->curl -X POST --data "file=/var/www/phpmy/config.inc.php" http://192.168.227.83/test
after executing the command i was lucky and found the below credentials:

Username: root
Password: roottoor

## Privilege Escalation
what i usually do when i find root credentials id use those to login into SSH if it is open 
And yes if you remember in this case also SSH is open (check out nmamp scan and youll find ssh is open)
so i used the below command to login to ssh:
go to the terminal and then,
->ssh root@192.168.227.83 

when next prompted type "yes"
and for password use "roottoor"

## Post-Exploitation

and then i got the shell , i was named root but to confirm i used the below command:
-> id (and found i was root)
hence it was confirmed that i was root 
hence i had completed my objective

## conclusion
This concludes the walkthrough of the Billu: Box machine
In this write-up, we have discussed how to perform a penetration test on a vulnerable machine.
This was a fun challenge and I was able to gain root access

I hope this write-up has been helpful in understanding the steps involved in a penetration test.
                                    "THANK YOU"



   
