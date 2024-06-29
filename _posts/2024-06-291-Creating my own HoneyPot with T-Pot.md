---
title: Creating my own HoneyPot with T-Pot
date: 2024-06-29 11:00:00 +0000
author: 0xbirb
categories:
  - Writeups
  - Backdoor
tags:
  - Honeypot
  - TPot
  - security
  - infosec
pin: false
---


For this project, i'll be using a Virtual Private Server offered by the German cloud provider NetCup. You will learn how to install [T-Pot](https://github.com/telekom-security/tpotce), the all in one HoneyPot.

My server has the following dimensions witch are more than enough to satisfy the [system requirments](https://github.com/telekom-security/tpotce/blob/master/README.md#system-requirements):

	- RAM 8,192MiB (8GB)
	- 4x CPU Cores of x86 Architecture
	- 160 GiB Harddrive

<br>

## Setting up our server

For the OS I choose Debian 12 with code name "Bookworm". If you're also using a VPS, I recommend choosing a SSH-Key instead of a username and password.

![[Pasted image 20240629160532.png]]

After waiting a few minutes, the image is successfully deployed and we can try connecting via SSH using the initial user we created

![[Pasted image 20240629160553.png]]
In order to access the system easily from my workstation, I've generated a fresh SSH Key to authenticate against the server and added it to the server using ssh-copy-id:

```bash
#generate strong key, optionally add passphrase
ssh-keygen -t ed25519 -C "YourSSH-Key Label"

#next. copy public key to our server
ssh-copy-id user@vps_public_ip

#authenticate to the server using the ssh key
ssh -i "YourSSH-KeyLabel.pub" user@vps_public_ip
```

![[Pasted image 20240629160615.png]]
The command shows it executed properly, we should now be able to authenticate with our newly generated ssh-key.

![[Pasted image 20240629160643.png]]
voilà! we get a shell and can now proceed with the installation of t-pot.


## Setting up T-Pot

This is a fairly straight-forward process. We can generate our own ISO File using makeiso.sh
script from T-Pot's GitHub.

```bash
#install git if not already installed
sudo apt install git

#copy tpot repo
git clone https://github.com/telekom-security/tpotce
cd tpotce
```

Install T-Pot as current user

```bash
./install.sh
```

The installation will do the following which we have to keep in mind:

- Changes the SSH Port to tcp/64295, since we want to use ssh's standardized port 22 as a honeypot
- Disable the DNS Stub Listener to avoid port conflicts with honeypots
- Set the firewall target for the public zone to ACCEPT
- Add Docker's repository and install Docker
- Add the current user to the docker group
- Add and enable `tpot.service` to `/etc/systemd/system` so T-Pot can automatically start and stop

![[Pasted image 20240629160708.png]]

When prompted for the type, I went with the Full Hive (option h) - this will require the most resources but will include all features. 
If you're unsure what to go for it might be sensible to check t-pot's [system requirments](https://github.com/telekom-security/tpotce#system-requirements) 

Once the installer is finished we need to reboot the machine and access it on port 64295 using ssh:
![[Pasted image 20240629160729.png]]

**Optional:** Opt-Out from submitting data to Telekoms Sicherheitstacho

```bash
#stop t-pot
systemctl stop tpot

#edit .yml file
vim docker-compose.yml

#remove Ewsposter service from the .yml file

#start t-pot 
systemctl start tpot
```

![[Pasted image 20240629160804.png]]
Remove the Ewsposter section to remove Telemetry


<br>

Now that we verified ssh login works, we can check out  our main form of administration using the Web-server created by T-Pot

```bash
https://vps_public_ip>:64297
```

Since we do not have a officially signed certificate, we will have to click through the warning of our web browser and proceed to log in using the defined user during the installation of t-pot

![[Pasted image 20240629160827.png]]

We are now greeted by t-Pots interface and can navigate to the Attack Map or the Kibana Dashboard where we are able to view the specific attacks that are happening on our honeypots

![[Pasted image 20240629160840.png]]

Great, we now completed the installation of t-pot. In the following sections, I will go into depth on what kind of honeypots are now set up and how to play around with the data.


## Overview of the Dashboards

### Kibana

On the landing page, we can choose the Kibana Dashboard witch is pretty cool, we can review each HoneyPot in detail and how exactly its being attacked.

![[Pasted image 20240629160901.png]]


For example, we can look at the entirety of the T-Pot Honeypots which we deployed. After around two Days of running it, we have a total of 66,234 attacks. For this reason, I also suggest lowering the amount of logs that we store to a period of 7 Days instead of 30 Days.

![[Pasted image 20240629160906.png]]

### Attack Map

The Attack Map is a live Dashboard of our HoneyPot and how it's being attacked from the globe. The marked Dots indicate known malicious adversaries that automatically try to compromise systems in the wild, you can gain a little intel by hovering over them.

![[Pasted image 20240629160915.png]]
### CyberChef

Cyberchef is a simple, intuitive web app for analyzing and decoding data without having to deal with complex tools or programming languages. You can "BAKE" recipes and reuse them

![[Pasted image 20240629160922.png]]

### SpiderFoot

SpiderFoot is an automated OSINT Tool that allows you to scan domains, IP adresses, hostnames, entire subnets and much more from within your newly setup VPS.

There's different types of scans:

-  Get anything and everything about the target; **ALL**

- Understand what information this target exposes to the Internet; **Footprinting**

- Best for when you suspect the target to be malicious but need more information; **Investigation**

- When you don't want the target to even suspect they are being investigated; **Passive**

Depending on the target, SpiderFoot can scrape lot's of data and provide you with valuable information like the email address format that the corporation uses. This can be quiet neat in engagements. 

Keep in mind that scanning a target only should be done in a ethical manner and not without permission, since public scanning is forbidden in certain countries.

![[Pasted image 20240629160933.png]]

## Using T-pot to analyze attacks

Placeholder

