
+++
title = "Sunday honeypot analysis"
date = "2019-10-06T1:00:00+02:00"
description = "Last month, I started participating to the Internet Storm Center by setting up a VPS on DigitalOcean with a honeypot called DShield. I get reports every day so I thought it would be fun to check out the reports from this Sunday and see if we could find something cool!"
draft = false
toc = false
categories = ["post"]
tags = ["dshield", "honeypot", "ISC", "malware", "traffic analysis"]
images = [
  "https://i.imgur.com/EqrmlFe.jpg"
] # overrides site-wide open graph image

+++

## About the Internet Storm Center and Dshield

<a href="https://www.dshield.org/about.html">The Internet Storm Center</a> (ISC) was created in 2001 and has been analyzing threats and problems since then with the help of volunteers around the world. In their own words: "The ISC uses the <a href="https://github.com/DShield-ISC/dshield">DShield</a> distributed intrusion detection system for data collection and analysis. DShield collects data about malicious activity from across the Internet. This data is cataloged and summarized and can be used to discover trends in activity, confirm widespread attacks, or assist in preparing better firewall rules".

Pretty cool, right? Volunteers can set up DShield on a Raspberry Pi for example, or a VPS like I did, save in DShield their API key from their ISC account, and reports will be sent to ISC twice an hour! And you have a full overview of what and who has been targeting your honey pot as well as 404 reports. Here is how it looks:

![DShield screenshot](/honeypot1/honeypot.png)

As you can see, today there has been already over 5.000 reported scans of my honeypot (and it's only 3pm here) and I can also see more info about the sources:

![Example of a source scanning my honeypot](/honeypot1/source-info.png)

Here is an example of a source that has been scanning my honeypot today a lot for port 22. Of course, these scans are rarely made by an actual hacker, but rather by bots that were created by different hacking organizations including state-sponsored groups and that are looking for vulnerable systems on the internet 24⁄7. As you can see, this IP comes from China and ISC gives a lot more info on the page too.

My long-term plan is to develop a python script that would take each reports available from a certain date range, and build a more interesting report out of this larger sample instead of presenting only one day. If you want to check a specific day in the past, you need to download an excel report and my excel skills are terrible so let’s stick to Sunday as ISC provides use already with a nice report page for today. Alright, let’s have a look!

<br>
## The ports

![Targeted ports](/honeypot1/target-ports.png)

The most important piece of information is the targeted ports. It gives us a better understanding of the threats currently out in the wild. If there is a clear spike in the scan of a particular port it indicates the beginning or return of a broad-based attacks, or some type of worm that uses vulnerable systems to scan for more victims after it infects them.

Here we can see that my honeypot has been largely scanned for port 22, which is the standard port for the SSH remote login protocol. I have noticed that it stays as the top 3 most scanned port everyday, probably to try brute-forcing it (get access to the machine by ssh and you’re in…) and also possibly to find a response from two old trojans that are possibly still active, like Shaft:

{{< hackcss-alert type="info" >}}
Name: Shaft<br>
Ports: 22, 5002, 18753 (UDP), 20432, 20433 (UDP)<br>
Files: idle - 28,969 bytes tcp.log - ??? bytes pp.pl - 2,795 bytes sniff.pid - 6 bytes s - 7,654 bytes chattr - 7,656 bytes vi - 437,428 bytes tcsh - 262,756 bytes ps - 31,312 bytes shaftmaster - 25,123 bytes shaftnode - 15,184 bytes shaftnode.c - 19,806 bytes hitlist - ??? bytes<br>
Created: Oct 1998<br>
Actions: Distributed DoS tool / Steals passwords. Is able to either send UDP, TCP or ICMP floods, or all three at the same time.<br>
Lenguage: Written in C.
{{< /hackcss-alert >}}

The next most scanned is port 23. As I understand it, this one has always been a clear favorite and, as explained in <a href="https://www.dshield.org/diary/Increase+in+Port+23+%28telnet%29+scanning/21115">this ISC report</a>, the exploit compromises telnet servers following this procedure:

1- brute force password (usually a well known default password)<br>
2- Download additional malware via ftp/http or tftp (typically multiple binaries for various architectures)<br>
3- run the malware, which will typically setup a client for a DDoS botnet.<br>

So if you're running a telnet server: change your default password!

The 3rd most scanned was port 465 and there has been a lot going on since 1995 around this port! But realted to security, there was not much information about why this port should be scanned so much. I found the <a href="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-4015">CVE-2011-4015</a> which "allows remote attackers to cause a denial of service (interface queue wedge) via malformed UDP traffic on port 465, aka Bug ID CSCts48300". Not too exciting.

Port 445 was also heavily scanned. This port can be used by hackers to remotely commandeer Windows machines, and the overall consensus is that you do NOT want this port to be exposed to the internet. Just have a look at this <a href="https://www.speedguide.net/port.php?port=445">terrible list of trojans and worms</a>.

One port that has been scanned a lot more in the last couple of months is port 1443:

![Graph of port 1443](/honeypot1/port-1443.png)

As explained in <a href="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-4684">CVE-2014-4684</a>, "the database server in Siemens SIMATIC WinCC before 7.3, as used in PCS7 and other products, allows remote authenticated users to gain privileges via a request to TCP port 1433". Interesting that the port for this CVE from 2014 has been scanned for a lot in the last 2 months...</p>

<br>
## The sources

![Graph of the source IPs](/honeypot1/source-ip.png)

Three main IP come out on top:

- 218.92.0.208: China. Bruteforcing port 80 and 22.<br>
- 92.118.38.53: Romania. Fairly new, not much info about previous reports.<br>
- 112.85.42.72: China. Old IP used for Forum spamming, now an Asterisk VoIP and port 22 Scanner.<br>

<br>
## The rest of the world

Here is more details about all the info collected by the DShield honeypots around the world:

![Most scanned ports](/honeypot1/top-10-ports.png)

And the top 10 offensive IPs where I added the country of origin:

![Most offensive countries](/honeypot1/countries.png)

![Map gif](/Hacker.gif)

There is a lot more information but I thought it was already a good start! I had a lot of fun putting all this info together and hopefully I can do a similar one every Sunday. Please have a look at <a href="https://github.com/DShield-ISC/dshield">DShield</a> and consider contributing to the project. If you need help setting it up, feel free to contact me on <a href="https://twitter.com/victor_theyknow">Twitter</a>!</p>
