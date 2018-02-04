---
layout: post
title: HTB Solidstate Write-up
---
{{ page.title }}
================
<p class="meta">4 Feb 2018 - St. Louis</p>
This is my first write up. It’ll show, I’m sure. (Specifically, my note taking was lame, so there will be missing details.)

The system name: Solidstate

## **Phase 1: Enumeration**

I like to start with the basics--nmap is your friend! I’m still nailing down my preferred switches, etc. Looking through the artifacts, I have three scans.

1. `nmap -o 10.10.10.51 -A -p* -sV -Pn -T5 10.10.10.51`

This reports the following open ports and services (edited):

*   22/tcp, ssh, OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
*   25/tcp, smtp, JAMES smtpd 2.3.2
*   80/tcp, http, Apache httpd 2.4.25 ((Debian))
*   110/tcp, pop3, JAMES pop3d 2.3.2
*   119/tcp, nntp, JAMES nntpd (posting ok)
*   4555/tcp, james-admin, JAMES Remote Admin 2.3

2. `nmap -o 10.10.10.51_2 -p 22,25,80,110,119,4555 -sV -Pn -T5 10.10.10.51`

3. `nmap -oX 10.10.10.51_3 -p* -sV -Pn -T5 10.10.10.51`

The second scan had cleaner output than the first one. The third was run to pull the results into metasploit as an exercise to get more familiar with its capabilities.

Looks like JAMES is going to be a thing. So, run that through searchsploit (you are running this on Kali, right?):

`# searchsploit james`


Exploit Title | Path (/usr/share/exploitdb/)
Apache James 2.2 - SMTP Denial of Service | exploits/multiple/dos/27915.pl
Apache James Server 2.3.2 - Remote Command Exe | exploits/linux/remote/35513.py
WheresJames Webcam Publisher Beta 2.0.0014 - R | exploits/windows/remote/944.c

A DOS attack isn't useful and we're not messing around with a webcam, so 35513.py looks like it's a good place to start.

In the meantime, there's a web server to enumerate. At this stage of the game, I'm still a n00b when it comes to web sites and web apps. Basically, I'm working on figuring out what makes sense to me and what works with my preferences. I gave dirbuster and nikto a go. Apologies, though, I don't have what I used at the command line for nikto. I’m not going to include the results of those scans, because they didn’t end up mattering too much.

## Phase 2: Exploit

While the scans were running, I visited 35513.py. Reading through it, we see the default creds for the remote administration site are root/root. Seriously? Yeah, that worked. The script submits a job to JAMES that will run once a user logs in, and it runs in the environment of that user. Interesting, but I didn’t find that useful. Instead, I used the creds to log into the remote administration tool. Unfortunately, this is where my note taking is lacking and I'm missing some details. I was able to enumerate the users of the email system and there was a menu setting to reset passwords. So, I reset the passwords, fired up an email client and slurped down all the email. There were three users, as well as I can recall, a manager, the email admin, and Mindy. There was an email from the manager to the email admin asking for creds to be set up for Mindy on the server. ...and an email from the email admin to Mindy with her userid and password, so she can ssh into the box.

SSH in as Mindy and I find myself in a restricted shell (/bin/rbash). Flag one was found and posted. My notes don’t indicate which commands were explicitly available. There was a subdirectory in Mindy’s home directory that had allowed commands in it. So, after doing some research on breaking out of rbash or other restricted shells, I gave scp’ing a copy of bash into the subdirectory a try. It worked! Logged back in as Mindy, called the version of bash in the subdirectory and I was off to the races.

## Phase 3: Privilege Escalation

This one took some time because of my lack of experience. In the end, I used a linux enumeration script (linuxenum.sh--header indicates it’s from www.rebootuser.com), I will freely admit I found in /tmp left behind by someone else. It revealed a world-writable file, /opt/tmp.py, that was read and executed by some job every minute. I modified the file to grab the root flag, drop it into /var/tmp, and re-permissioned for me to grab the contents. (I wanted to go back in and find what was running the job, I couldn’t find it in cron. It was retired before I could do so.)

## Lessons Learned
As an early CTF box for me, this was an interesting challenge. It took me about a week, in clock-time, from start to finish. I think I spent beween 10 and 24 hours on it in effort. Some things to keep in mind:
- Enumerate, enumerate, enumerate.
- Nikto isn’t quite as useful as dirbuster.
- Take better notes. Write it all down. (Need to work on organizing.)
- Metasploit is interesting, but didn’t help as much as I’d hoped for organizing notes and artifacts I’m interested in.
- Sometimes, you gotta try it and see.
