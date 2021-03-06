# Vulnerable Machine Stapler Writeup
## Sean Kannanaikal Start Date: 8/31/2021

![Login-Page](img/vm-login-screen.PNG)

[Stapler](https://www.vulnhub.com/entry/stapler-1,150/) is a Vulnerable VM found at Vulnhub.com.  The creator claims it has "2 ways to get a limited shell and at least 3 ways to get root access".  My goal in this VM is to get a complete boot2root with this VM.

***

# Information Gathering
I started of by trying to find the IP address of the VM.  It is currently running within a NAT Network between my VMs.  In order to find out the IP of the machine I started a scan on the entire subnet.

```
nmap 10.1.1.0/24
```

This command allowed me to scan the entire network and it gave me the results shown below.

![Host-Discovery-Scan](img/host-discovery.PNG)

There looking at the scan results I was able to determine the vulnerable VM was found at the IP **10.1.1.10**.

I Then decided to do a more focused scan in order to understand the OS, and specific versions of the ports.

```
sudo nmap -A -sV 10.1.1.10 -p 0-65535
```

With this extend nmap scan I will be scanning all potential ports that the system could be using alongside doing a service detection scan. The results I obtained are depicted in the scan down below.

![Service-Detection-Scan1](img/service-detection1.PNG)
![Service-Detection-Scan2](img/service-detection2.PNG)
![Service-Detection-Scan3](img/service-detection3.PNG)
![Service-Detection-Scan4](img/service-detection4.PNG)

With all the data that I just btained I needed to begin to research and cross reference the results I obtained to make my way through this treasure chest of information and get the lay of the land. However, I wanted to try gathering information on each of the ports individually by playing with the software.

## FTP investigation
In order to get further information I began attempting to interact with each of the open ports starting off with FTP.

I started off by attempting to connect to the ftp via the command
```
ftp 10.1.1.10
```

I was successfully able to connect and get prompted with a login screen but also was given a neat little message when I connected.

![FTP-Login](img/ftp-login.PNG)

"Harry, make sure to update the banner when you get a chance to show who has access here"

Ok so I'm beginning to get an office feel with the stapler and now someone is talking to Harry and explaining to him some tasks he has to get finished.  Not only that but maybe its safe to assume that Harry has access to the FTP server if he can show who has access by updating the banner displayed.  Not only that but I tried to see if I could login via the popular anonymous login account.  

Simply put if one types in the username anonymous followed by a random password they can log into the ftp server as long as anonymous login is setup.

Wonderful it seems this FTP server has anonymous login setup not a good sign as it allows attackers such as myself to access important resources.  Not only that but from the previous message addressed to Harry I was able to paint a small picture of the infrastructure within my head.  Interested by this I began to create a list of interesting findings with both the name Harry and anonymous noted.

Moving on now that I am in the ftp server I began to search a round for information.

```
ls
```

![FTP-Files](img/ftp-files.PNG)

Intersting a file titled note I decided to transfer that file over to my local machine for future inspection.  I then tested to see if I could upload any files from my system  however I get a 550 Permission denied error. Oh well but I do have a note which could be intersting. I disconnected from the ftp server and went to see the note.

![FTP-Note](img/ftp-note.PNG)

Interesting two new users Elly and John.  I immediately added these names to the name file and began to attempt to bruteforce login with the username **John**.

Using Hydra I attempted to find John's credentials to get access to this payload that Elly developed.

```
Hydra -l John -P /usr/share/wordlists/fasttrack.txt 10.1.1.10 -t 10 ftp
```

No results hmmm... Ok I tried one more time with another wordlist.

```
Hydra -l John -P /usr/share/wordlists/rockyou.txt 10.1.1.10 -t 10 ftp
```
Rockyou.txt is famous for being quite a large file so instead of waiting for the entire bruteforce to finish to finish I began looking into the ftp server from another angle.

Once again to get a more focused view of ftp I did a simple nmap scan

```
sudo nmap -A -sV 10.1.1.10 -p 21
```

Ok so just as I tested anonymous login is allowed.  I was also able to find the version of the software is vsFTPd 3.0.3 labeled to be secure fast, stable.  I wonder?  I began to search through google for vulnerabilities associated with the system.  Unfortunately it may seem that vsFTPd 3.0.3 may seem quite secure aside from a Denial of Service attack which I may code up during a later time, but for now I will move onto another port.  Lets do SSH as its the next numerically speaking.

## SSH Investigation

```
sudo nmap -A -sV 10.1.1.10 -p 22
```

![SSH-Service-Detection](img/ssh-service-detection.PNG)

Unfortunately I was provied by no glaring issues so I tried connecting to the ssh server.

![SSH-Login](img/ssh-login.PNG)

Interesting another name Barry which I immediately added to the names listing.

## Attempting Enumeration

At this point I decided to run some enumerations on many of the ports as I was not able to find any glaring vulnerabilities in the sytems.

Starting with ftp I utilized nmap scripts in an attempt to find some weaknesses

```
sudo nmap -p 21 -sS --script ftp-anon.nse,ftp-brute.nse,ftp-vuln-cve2010.4221.nse,ftp-vsftpd-backdoor.nse 10.1.1.10
```

This returned no new results so I was stumped and decided to proceed for another avenue of attack.

I then began attempting some ssh enumeration especially after finding out online that SSH does not have many vulnerabilities per say that can be utilizied to gain access.  Nonetheless I wanted to see if there was anything so I went through the enumerating process.

```
sudo nmap -p 22 -sS --script ssh-brute.nse,ssh-auth-methods.nse,ssh-publickey-acceptance.nse,ssh-hostkey.nse 10.1.1.10
```

After going through this it seems that ssh is not accepting of any other authentication methods besides passwords as a result my only option will be to bruteforce the system which I will turn to as a last resort.

In another attempt I attempted to break into the mysql account so I attempted enumerating via the command

```
nmap -p 3306 --script mysql-databases.nse,mysql-dump-hashses.nse,mysql-empty-password.nse,mysql-enum.nse,mysql-info.nse,mysql-users.nse 10.1.1.10
```

![MySQL-Enumeration](img/mysql-enum.PNG)

I was actually given a list of valid usernames that dont accept any passwords in order to access including administrator accounts and even root.  So I immediately assumed I hit the jackpot and began to authenticate to the server with root at first by running the command

```
mysql -h 10.1.1.10 -u root
```

but then I ended up getting the error

```
ERROR 1045 (28000):  Acess denied for user 'root'@10.1.1.8 (using password: NO)
```

I was confused at first because these were valid credentials apparently so after looking up online it seems that potentially the mysql server only allows localhost logins with these accounts and so I can only use these later once I actually have access in the server so for now I have hit another dead end.

I then proceeded to enumerate port 12380 which suprisingly had a web interface and when I looked through the code I found some potentially interesting information.

![Web-Code](img/port-webCode.PNG)

Another name 'Zoe' which we can add to our list not only that but the background

From here I began attempting to enumerate port 139 SMB.

I have never really heard of SMB so I began looking into the protocol and I found it to be Server Message Block protocol and it acts as a file share protoocl for communicating with servers.

This article [SMB Enumeration Guide](https://www.hackingarticles.in/a-little-guide-to-smb-enumeration/) was quite helpful in allowing me to learn and go about this enumeration process.

So I stsarted off by running nmblookup as a way to see if I could identify any information via the command

```
nmblookup -A 10.1.1.10
```

![NMB-Lookup](img/nmblookup.PNG)

From the results I was succesfully able to obtain the hostname of **"RED"** which may turn out to be useful in the future so I will make note of it for now and continue on enumerating the port.

I then proceeeded to do an nbtscan of the entire subnet to see if there were any other devices within the network for NetBIOS related information.  I was unable to find any new information or new devices besides the already vulnerable vm as shown by the results below.

![nbtscan](img/nbtsubnetscan.PNG)

```
nmap --script nbstat.nse 10.1.1.10
```

This was the next enumeration scan that I used an nmap script that woudl allow me to identify all names and systems owned by it.

![NMAPNBSTAT](img/nmapNBSTAT.PNG)

Here we can see the results of the nmap scan once again not turning up anything new and revelating but acts as a good enforcement of the information that we have previously found as we corroborate all the infromation together.

I then learned about the tool **enum4Linux** which ended up being our savior.  

I did a simple scan with enum4linux via the command

```
enum4linux 10.1.1.10
```

And was able to get a listing of many of the users 

![Domain-Users](img/users.PNG)

I proceeded to add these to the list and was able to find a password policy set up which asks that minimum passwords should be used with a length of 5.

I then used these lists of users as a way to enumerate into the other protocols that I discovered and was having a hard time accessing.

First with FTP

```
hydra -v -L names.txt -P /usr/share/wrodlists/rockyou.txt 10.1.1.10 -t 4 ftp
```

Then with SSH

```
hydra -v -L names.txt -P /usr/share/wrodlists/rockyou.txt 10.1.1.10 -t 4 ssh
```