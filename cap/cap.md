# Cap Box writeup

## Sean Kannanaikal Start Date 9/3/2021

Cap is a popular vulnerable machine from hackthebox with a rating of 4.3 stars.  It seems to be an excellent beginner penetration testing box to practice on.

***

## Information Gathering

I started off this pentest adventure by identifying all the open ports for Cap via the command
```
nmap 10.10.10.245
```

![Identifying Open  Ports](img/capture1.PNG)

From here I see 3 ports open ftp, ssh, and http.  I tend to gravitate towards looking into the web interfaces first so I started to work on enumerating port 80 first.

### Port 80 enumeration

![Web Interface](img/capture2.PNG)

When we connect to the system over port 80 this is the website it is hosting.  I started off by noticing a user by the name of Nathan so I started making a list of users that we could utilize in the future.

From there I started playing with the options in the drop down under Nathan's name to no avail it seems those parts of the website haven't been coded yet. 

I then changed my direction towards the options on the dashboard.

Starting off with the Network Status page displayed down below.

![Network Status Page](img/capture3.PNG)

It seems to be a large page displaying the results from the Netstat command interesting we will investigate that later in greater detail.

I next moved onto IP Config and just as I suspected it is displaying the results of ifconfig as shown in the capture below.

![IP Config Page](img/capture4.PNG)

Unfortunately for us this doesn't give us any new information about the box that we dont know of already.

Then I looked into the final of the 3 pages within the dashboard the Security Snapshot which started giving me a new page incrementing the value found in the url by 1 however only the first one seems to have any information so I decided to download it.

![Security Snap shot Page](img/capture5.PNG)

Once I clicked the download page I was given a pcap file which I decided to open up in wireshark.  

The capture seems to be of 24 packet http interaction between my kali machine and the vulnerable machine.  

![Pcap Investigation](img/capture6.PNG)

I wanted to move on temporarily and look into the other ports available to see if I can  pull anything else.

### Port 21 Enumeration

I wanted to start off with ftp I'm going to see how the service interacts when we attempt to connect to it.

![FTP login attempt](img/capture7.PNG)

So I initially attempted to see if it allows anonymous ftp login access however it doesn't so I moved on to attempting to bruteforce the credentials with the user **Nathan**.  Using the command down below

```
hydra -L names.txt -P /usr/share/wordlists/fasttrack.txt
```

### Port 22 Enumeration


