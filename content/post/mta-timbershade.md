
+++
title = "Writeup: Timbershade - TRAFFIC ANALYSIS EXERCISE"
date = "2019-10-27T1:00:00+02:00"
description = "I had already solved one exercise from @malware_traffic's website about network traffic related to malware infections. I have been slow to pick a new exercise from the very long list but I chose Timbershade and it was a lot of fun! Short one, but learned a lot once again."
draft = false
toc = false
categories = ["post"]
tags = ["forensic", "malware", "traffic analysis", "wireshark"]
images = [
  "https://research.checkpoint.com/wp-content/uploads/2019/07/russia_1021x580.jpg"
] # overrides site-wide open graph image

+++

## Malware Traffic Analysis

<a href="https://twitter.com/malware_traffic" target="_blank">@malware_traffic</a> <a href="https://malware-traffic-analysis.net/" target="_blank">blog</a> has a lot of knowledge so I highly recommend to bookmark it somewhere. The real treasure is of course the amazing <a href="https://malware-traffic-analysis.net/training-exercises.html" target="_blank">exercises page</a>. Depending on the exercise, you get a pcap and other files. The pcap file is a traffic capture which we can analyse in Wireshark and find out where things went wrong!

Being able to effectively analyse traffic is a very important skill for the security of any organisation. It helps the security team to find out where the problem happened and how to mitigate it. It is also super fun!


<br>
## Timbershade

Today, I'll do one of the recent exercises <a href="https://malware-traffic-analysis.net/2019/01/28/index.html" target="_blank">"2019-01-28 - TRAFFIC ANALYSIS EXERCISE - TIMBERSHADE"</a>. This time we got two files: a pcap and a text file with Windows security alerts.

QUESTIONS:

• What is the IP address of the infected Windows host?<br>
• What is the MAC address of the infected Windows host?<br>
• What is the host name of the infected Windows host?<br>
• What is the Windows user account name for the infected Windows host?<br>
• What is the SHA256 file hash of the Windows executable file sent to the infected Windows host?<br>
• Based on the IDS alerts, what type of infection is this?<br>

<br>
## My Answers

1) What is the IP address of the infected Windows host?<br>

![IP address](/timbershade/source-ip.png)
The IP of the infected Windows host is 172.17.8.109

2) What is the MAC address of the infected Windows host?<br>
3) What is the host name of the infected Windows host?<br>

I'll answer both questions together because we can find the hostname and the MAC address quickly through different ways:
<a href="https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/">https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/</a><br>
In this case, we filter by “bootp” > Go to Bootstrap Protocol and then in the options we find the hostname and MAC address:<br>
![Hostname and MAC](/timbershade/mac-and-host.png)

4) What is the Windows user account name for the infected Windows host?<br>

To find this information we can look at the Kerberos traffic. To do that, filter the traffic by “kerberos.CNameString” select one of the frame with the source being our infected Windows host, navigate the frame details sections:
![CNameString](/timbershade/username.png)

This will add the CNameString to all the Kerberos request so we can find one of them with the username and not the hostname. CNameString values for hostnames always end with a $ (dollar sign), while user account names do not.
![Username](/timbershade/username2.png)

The username is “margaret.dunn”.

4) What is the SHA256 file hash of the Windows executable file sent to the infected Windows host?<br>

We can find the executable file in: File > Export objects > HTTP. I downloaded the .bin file and in my terminal, I checked its sha256:
{{< hackcss-alert type="info" >}}
root@kali:~# sha256sum actiV.bin <br>
9f6e3e65aedca997c6445329663bd1d279392a34cfda7d1b56461eb41641fa08 actiV.bin
{{< /hackcss-alert >}}

5) Based on the IDS alerts, what type of infection is this?<br>

Looking at the text file with all the alerts, we easily find several lines showing:
{{< hackcss-alert type="info" >}}
ET TROJAN ABUSE.CH SSL Blacklist Malicious SSL certificate detected (Dridex)<br>
192.241.220.183 -> 172.17.8.109<br>
IPVer=4 hlen=5 tos=0 dlen=1045 ID=0 flags=0 offset=0 ttl=0 chksum=25788<br>
Protocol: 6 sport=3389 -> dport=49214
{{< /hackcss-alert >}}

We can easily conclude that it was a Trojan.Dridex infection which is a banking trojan and spyware trageting Windows systems. You can read more about it in <a href="https://blog.malwarebytes.com/detections/trojan-dridex/" target="_blank">this article</a>.

That's all for today! Go try some of his exercices and contact me on <a href="https://twitter.com/victor_theyknow" target="_blank">twitter</a> if you need help!

![Final gif](/city.gif)
