# Tryhack me Room Bounty Hacker
## Sean Kannanaikal Start Date: 9/20/2021 - End Date: 9/20/2021



***

# Information Gathering

In order to understand the system I started off with a simple nmap scan on the ip.

```
nmap 10.10.55.213
```

<inser image here>

As seen from the results shown above I was able to locate 3 open ports on the system: HTTP(80), SSH(22), and FTP(21).

Before I began experimenting with these ports I ran a more complex scan in order to find any other bits of information that may be relevant for me.

```
nmap -A -sV 10.10.55.213 -p 1-65535
```

<insert image here>

No new ports were shown to be open so I decided to begin enumerating the ports.

## Port 80 Enumeration

I connected to the site

<insert image here>

A funny little bit of dialogue that definetely got me slightly motivated to become a hacker truly worth of the crew's respect.  With my spirits lifted high hoping to make my fellow crew members proud I decided to enumerate port 80.

Quick little directory listing to identify any hidden pieces of information.
```
dirb http://10.10.55.213/
```

<insert image here>

Unfortunately nothing bummer.

So I thought to move onto the next port FTP.

## Port 21 Enumeration

I immediately started off by seeing if I can log into the ftp server via anonymous login

<insert image here>

Voila We are in we have an initial point in access

a quick list of the files reveals

<insert image here>

2 files *locks.txt* and *note.txt*

<insert image here>

<insert image here>


I read through them a listing of either useraccounts or passwords and a little message respectively.  I initially thought to bruteforce my way into the ftp client with the new locks.txt file I obtained but I decided to instead attempt to brute force ssh so I could have a larger more interactive shell.

## Port 22 Brute Force Exploitation

Knowing that this was a traditional ssh server I spun up a quick hydra command to bruteforce the account.

```
hydra -L /locks.txt -p /usr/share/wordlists/fasttrack.txt -t 10 10.10.55.213 ssh
```

When I noticed that it was going to take an hour and especially when it was 11:20 I was about to give up hope until I remembered from note.txt. It was signed by *lin*.

I immediately created a new command

```
hydra -l lin -p /locks.txt t 10 10.10.55.213 ssh
```

Then bam I got the user name and password

## Access Obtained

I logged into the system via ssh with the acquired credentials and then I was able to find the user.txt file and then I looked for ways to obtain root privlieges.

## Privliege Escalation

I started off by running a simple

```
sudo -l
```

to see if I can even run any commands as root and voila I can.

<insert image here>

I can run tar.  So I then proceeded to go to gtfobins and found the command

```
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

With that I got the root shell.

<insert image here>

From there I navigated to the root directory and obtained the key.


