---
layout: post
title: "How to Capture and Crack WPA/WPA2 Wifi Passwords"
categories: Wifi_hacking
author:
meta:
searchable: true
---


#####NOTE  put picture here r#########



### Intro

This is a detailed article on how to capture WPA/WPA2 Wifi handshakes and crack the hash to retrieve a networks password. It is purely written for the education value of showing you how easy it is to break into your own home Wifi network if you use a weak password.

Alternativly if you are an aspiring Pentester or RedTeam enthusiast you can use this article as a guide on your own networking equipment to learn how to test the security of any wifi network. I do not condone using this guide on unsuspecting individuals without their explicit consent.  

<!--cut-->

### Equipment 

Now that we have that paliminary stuff out of the way I'll run though some of the equipment you will need to follow this guide.  

- A laptop with that has a wireless chip that supports monitor mode\
( don't worry we will test to see if your Wifi chip supports monitor mode later in the article)   
    
- A Linux operation system environment, ideally Kali Linux but any Linux distro will do.  

That’s it! real simple huh 

If you are running on a Windows machine and don't want to dual boot Linux or if you are a Linux user and don’t want to install anything extra on your OS you can follow this article to create a live boot USB of kali Linux if you have a spare USB of at least 8GB laying around. 

Follow the [official kali linux documentation](https://www.kali.org/docs/usb/live-usb-install-with-windows/)

This way won't have to mess with your current installation of windows, just hit F12 ( or whatever your boot selector button is ) and choose the USB key to boot from the list of boot devices.

![The San Juan Mountains are beautiful!](/assets/ArticleImg/Wifi/AcerPic4.png "San Juan Mountains")

### Environment

There are a couple of different environments as described above that you can follow this article with. 
I'll try to provide a description of all of them so regardless of your on hand equipment you can still try to apply it to your use case.

 **Kali Linux**
 
 ---

If your dual booting Kali Linux or running from a live cd then boot into your OS and open up a root terminal from the "Application menu" in the top left of the screen as seen in the pictures below.  

Please Note as you are booting into the OS the default credentials are

```
Username: kali    
Password: kali        
```

![terminalPic1 ](/assets/ArticleImg/Wifi/terminalPic3.png "terminal pic 1 kali")


Then type this into the root terminal window to list the wifi devices on your system that supports monitor mode.

```bash
airmon-ng
```
If you see a result something like this than your in luck! You have a wifi device that supports monitor mode : )


![Finding out if you have a wifi device that supports monitor mode ](/assets/ArticleImg/Wifi/wifidevicePic1.png "moniter terminal")

 **Other Linux distros**
 
 ---

 On other Linux distros such as ubuntu the one I'll be using, do not come with the necessary softwear ["aircrack"](https://github.com/aircrack-ng/aircrack-ng) installed by default so we are going to have to install it ourselves. 

 To do this open up a new terminal and type 
```bash
 sudo apt-get install aircrack-ng
```
Then follow the instructions for it to install, you might have to press y along the way.  
After that is complete open up a new terminal and type
```bash
airmon-ng
```
If you see a result something like this you are set to go! You have a wifi device that supports monitor mode : )

![Othe linus Finding out if you have a wifi device that supports monitor mode ](/assets/ArticleImg/Wifi/unTerminal.png "moniter terminal other linux")


If your laptop doesn't come with a wifi device that supports monitor mode can purchase USB Wi-Fi card that plugs into your computer a has a much higher omnidirectional range than what your typical Wifi laptop chip would, with the option to use directional antennas if you want. 

![wifi usb ](/assets/ArticleImg/Wifi/alfaHalf.png "Alfa AWUS036NHA WIFI USB adaptor")

I would recommend the [Alfa AWUS036NHA WIFI USB adaptor](https://www.alfa.com.tw/products/awus036nha?variant=36473966166088)
It is pretty much the gold standard for wifi packet capturing in the wifi hacking community and it has the proper chipset and drivers in order to be put into monitor mode on almost all modern computers. 


### Theory / Explanation  

Im now going to explain to you how the design of WPA/WPA2 Wifi networks can be exploited in order to gain persistent access to them. Note that if you are only interested in the methods you may skip down to the methods section, but be aware that knowing how the thory behind tools work and what weaknesses they are expoiting are very usefull for torubleshooting problems and efficent use of the tool as a whole. 

So in order to breach a wireless network and gain persistent access to it you will need to figure out the password that is set on their Wifi router (wireless access point). Most wireless routers nowadays use the WPA/WPA2 encryption standard to verify users trying to connect to the network and to encrypt traffic going over the network. whether you are an enterprise user or using consumer grade hardwear I can almost guarantee that your router is using the WAP/WPA2 encryption standard, so that will be the one that we focus on cracking today.

The part that we are interested in is the way in which the WPA2 encryption protocol tries to authenticate a user, because if we can replicate that, then we can gain repeated access to the access points ourselves. 

###### <ins>What is WPA2 4-way handshake?</ins>

The way that WAP/WPA2 wireless access points verify a device trying to connect to it is by preforming what is called a 4-way handshake between the device and the access point.  

<p align = center> <img src="/assets/ArticleImg/Wifi/WPA4way.jpg" />    </p>
<p align = center> -- Image Credits to cylab.be -- </p>

As you can see in the diagram above a 4-way handshake starts off by having what is called a Pre-shared key (PSK) this is the plaintext password that you enter into your Wi-Fi router when you sent it up and the plaintext password that you enter into your phone before you can access your WI-FI, basically a traditional password.

This Pre-shared key is then combined with Service Set Identifier (SSID) which is the name you give your router ( EX: TheSmithsWifi ) and the combination is fed into the hashing algorithm PBKDF2 as seen below.
<p align = center> <img src="/assets/ArticleImg/Wifi/PBalgo.png" />    </p>
<p align = center> -- Image Credits to cylab.be -- </p>

A hashing algorithm is a set of mathematical operations that are easy to do in one direction (don’t take a lot of time to compute) but are impossibly long to compute in reverse and get the original text fed into the algorithm. As you can see by the SHA1 hashing algorithm below our plaintext password "Password123" has been turned into the SHA1 hash of itself, this is a very simple example of what is going on in the hashing algorithm involved in the 4-way handshake.
<p align = center> <img src="/assets/ArticleImg/Wifi/exampleHash.jpg" />    </p>

The PBKDF2 containing the (PSK) and the (SSID) are then packed into a format called Pairwise Master Key (PMK), this information along with many other one time randomly generated keys (Anonce) verified by a (MIC) Message Integrity Code are sent back and forth between the client and the access point in order to verify and grant access to a client trying to connect.    

The full  4-way handshake process is quite complicated and out of the scope of this article, if you would like to understand to process in greater detail please check out [Alexandre's post](https://cylab.be/blog/32/how-does-wpawpa2-wifi-security-work-and-how-to-crack-it) over at cylab.be, their article goes in to great detail about every step in the handshake process. However to crack a WPA2 network we only need to understand the basics of the simplified explanation above. 


#### Gathering info to crack the password

So now that we know how a 4-way handshake works, we can try to capture all the information we need to reverse engineer the plaintext password (PSK) out of the handshake. 

The PBKDF2 hash mentioned earlier would take far too long to compute the original plaintext password by reversing the algorithm, even if you had access to a ridiculous amount of computing power (approximately 67 years to the heatdeath of the universe for passwords over 12 random characters). So the method that we use is to guess the plaintext password and then run that guess though the same hashing algorithm as the password we are trying to crack, in theory if the end result is the same then our guess is the right password. 


We first have to capture packets containing the authentication routine between the client and the access point before we can attempt the guess the plaintext password.

The packets we capture have to contain these in order to preform and attack:
- The access points name (SSID) (in order to add it to the plaintext password guess to compute the PBKDF2 hash)
- The MAC addresses of the client and the AP (to ID the info in the packets)
- The Nonces and each messsages message integrity code (MIC)

Using this captured information a password guessing attack would follow this routine. 

<p align = center> <img src="/assets/ArticleImg/Wifi/guessAttack.jpg" />    </p>
<p align = center> -- Image Credits to cylab.be -- </p>


### Methods to Capture a 4-way handshake

There are two different methods of capturing the 4-way handshake depending on the window of time have to conduct your attack and the amount of "noise" and evidence you want to leave behind in the defenders logfiles. 

##### <ins>The Quiet approach </ins>

If you have prolonged access to yours targets AP and you don’t want to alert defenders that there might be an attack on their network then quiet approach will suit your needs the best.

  <p align = center> <img src="/assets/ArticleImg/Wifi/QuietDia2.png" />    </p>
  
  
  
This approach works by waiting until clients that are connected to a network leave the network and have to reauthenticated upon wanting to rejoin the network. While they are rejoining they have to perform a 4 way handshake in order to reauthenticated themselves and access the network again. If we monitor the traffic whenever they are reconnecting we can capture the necessary information from the 4-way handshake to perform an offline hash cracking attack offsite.

An example of the above scenario would be if you were tasked with testing the security of a corporate office buildings wireless network. At the end of a work day all of the employees would leave with their wireless devices such as cellphones and laptops, disconnecting from the wireless network. If you set up in range of the corporate network the next morning and monitor it you can capture all of the employees reconnecting to the corporate network with their devices and capture the 4-way handshake which is need to perform an offline hash cracking attack. 


##### <ins>The "Disruptive" approach </ins>

If you have a short window of time to access yours targets AP and you don't care about alerting active defenders that there is an attack on their network as well as leaving evidence of an attack in the routers log files this "loud" approach will suit your needs the best. 

  <p align = center> <img src="/assets/ArticleImg/Wifi/loudDia.png" />    </p>

This approach works by actively kicking the client off of the network so they are forced to automatically rejoin the network giving up their 4-way handshake to anyone monitoring the network by reconnecting to the AP automatically. The way the attacker kicks the client off the network is by sending deauthentication-packets to the MAC address of a connected client, the client MAC address can be obtained by simply monitoring the network. 
These deauthentication-packets act somewhat like a DDOS attack on the connected client overwhelming them with incoming packets until they disconnect from the network and try to reconnect to a AP again.

 An example of the above scenario would be if you got a short 30min tour though a restricted section of a corporate office building that say had a special separate network for the executives of the company. You wouldn’t have time to wait for one of the executives to disconnect and reconnect to the network and give you the 4-way handshake so you would pick an executive devices mac address and send deauthentication-packets until it was kicked of the executive network. Once the device reconnects to the network while we are listening to it, we just captured it’s 4-way handshake in a relatively short amount of time.   


### Putting Theory into Pratice

During the rest of the tutorial I'm going to be using Kali Linux to preform the commands, but if you are using ubuntu or any other Linux distribution the steps should be the same.  

To start the program to monitor Wifi traffic root terminal and type

```bash
airmon-ng
```
 (normally not advised to open a root terminal but this is a live boot kali session so it is designed with this in mind) 

<p align = center> <img src="/assets/ArticleImg/Wifi/tPic1.png" />    </p>

This command displays your Wifi cards interface and as you can see my cards interface is called "wlan0"

Next we want to put our wifi card into monitor mode, that means rather than connecting our computer to a network it monitors the wifi networks in our area to do this we type airmon-ng start, then whatever yor wifi cards interface is called.

airmon-ng start  "wireless interface"

In my case my cards wireless interface is called "wlan0" so I would type 
```bash
airmon-ng start wlan0 
```
NOTE: give this command a bit of time to execute it can hold on the command line for a second.

<p align = center> <img src="/assets/ArticleImg/Wifi/tPic2.png" />    </p>

If you did not get this output and it returned and error instead try to update your system with 
```bash
Sudo apt update 
```
Or kill the processes that could interfere with the wifi card by typing.
```bash
airmon-ng check kill
```
If you did have success we can now put our wifi card into monitor mode and monitor our local wifi traffic.
To put your card into monitor mode type airodump-ng, then the montior interface name it just told us in the terminal output.

airodump-ng  "new wirless interface"

In my case my cards monitor interface is called "wlan0mon" so I would type 
```bash
airodump-ng wlan0mon   
```

<p align = center> <img src="/assets/ArticleImg/Wifi/seScan1.png" />    </p>
<p align = center> -- Note: the BSSID and ESSID of some wifi networks have been obscured to protect the privacy of my neighbours  -- </p>

This gives you a live view of all of the Wifi networks broadcasting in your area along with their metadata. 
Some information displayed on the live terminal view is

BSSID - (basic service set identifier) essentially the MAC adress that uniquely identifies the access point.  
PWR - The power of the signal you are getting from the AP, gets higher the closer you are to the AP.  
Beacons - Number of announcements packets sent by the AP.  
\#Data - Number of captured data packets from the AP.  
#/s - Number of data packets per second, ovet the last 10 seconds. 
CH - Channel number the AP is brodcasting on.  
MB - Maximum speed supported by the AP.  
ENC - Is the type of encryption algorithm it is using.  
AUTH - Is the type of cipher that is detected. EX:  CCMP, WRAP, TKIP, etc.   
ESSID - The SSID(Service Set Identifier) or the access point's name.  


Now that you can see the list of all of the active wifi access points in your local range press CTRL+C to exit the active window so that you can copy and paste the info you need to start an attack. 


First open a new seperate terminal window and cd into a directory you know how to access, in my case im going to cd into the Documents folder 
```bash
cd /home/kali/Documents
```
Then open up a text editor and copy and paste the line of text containting the access point you want to attack into it.

To narrow the monitoring of wifi traffic to a spesific access point we want to attack use the BSSID and the channel of the desired AP in this command. 

airodump-ng --bssid "your AP's bssid" -c "your AP's channel"  --write "name of file you want to create"  "wifi card interface" 

<p align = center> <img src="/assets/ArticleImg/Wifi/textedit.png" />    </p>

In my case the commands bssid is "F4:EC:38:FC:40:92", the channel is "4", the filename is "WPAcrackTest1" and the wifi card interface is "wlan0mon"
so my command would be. 
```bash
airodump-ng --bssid F4:EC:38:FC:40:92 -c 4 --write WPAcrackTest1 wlan0mon  
```
<p align = center> <img src="/assets/ArticleImg/Wifi/terminalPicMon.png" />    </p>

This is the point where you have to choose whether you want the conduct a quiet attack or a disruprive attack. Follow along below for the quiet approach or skip down to the [disruprive attack](#the-quiet-approach)


##### <ins>The Quiet approach</ins>

In order to do the silent attack you just need to wait for a client on the target network to reconnect and preform a 4-way handshake. 

After preforming the command from the prevois section to monitor the wifi traffic of a specific access point you sould see something like this.

<p align = center> <img src="/assets/ArticleImg/Wifi/targetAP.png" />    </p>

Highlighted in red are all of the connected clients to the access point. In this example network I already have my phone connected to the network when I started monitoring. 

In order to simulate someone leaving their network and rejoining over a period of time I turned my phone off and on again to force it to reconnect as a client to the access point.  

<p align = center> <img src="/assets/ArticleImg/Wifi/APcap.png" />    </p>

As you can see in the top left corner of the terminal the 4-way handshake of my phone joining the AP was successfully captured, now we have eveything we need to preform an offline attack on the hash. 

##### <ins>The “Disruptive” approach</ins>

A situation where you would use the Disruptive attack woukld be if you don't have enough time to wait for a client on the target network to reconnect. To make the client disconncet on our terms we are going to send deauthentication packets to kick them off their network at a time our our chooseing. 

Open up a new terminal window and type in this command. 

aireplay-ng --deauth "amount of deauth packet you want to send" -D -a "your AP's bssid"

In my case the command is 
```bash
aireplay-ng --deauth 100 -D -a F4:EC:38:FC:40:92 
```
<p align = center> <img src="/assets/ArticleImg/Wifi/deauth.png" />    </p>

Continue to repeat this command with interval of about 2 minute breaks until you see the 4-way handshake get caputred in the top left corner of the terminal. Be carful that you leave that 2 minuite rest period between each deauth command or else the devices that you knocked of the netwrok won't have an opportunity reconnect and give you the handshake.   

<p align = center> <img src="/assets/ArticleImg/Wifi/APcap.png" />    </p>

Now we have eveything we need to preform an offline attack on the hash. 


Go to the /home/kali/Documents folder or whichever folder you decided to send the output to in order to view the captured .cap files.

<p align = center> <img src="/assets/ArticleImg/Wifi/capFile.png" />    </p>

The .cap file show above containes all of the captured packets of the 4-way handshake and all of the info needed to preform the offline guessing attack previously disscussed in the [theory](#gathering-info-to-crack-the-password) section of the article. 

Aircrack-ng actual comes pre-packaged with simple password cracking tool so we can actualy take a spin at cracking the hash that we catured from our example wifi access point. 

To do this we are going to need a wordlist, or in other words a very long list of commonly used password that aircrack-ng can use to reapeatly guess the password of the hash that we captured. Luckly Kali linux comes with a pretty good wordlist already on the OS. 

Open up a new terminal and cd into the directory that containes the .cap file that we obtained earlier, in my case im going to cd into the Documents folder again. 
```bash
cd /home/kali/Documents 
```
Then I will unzip the wordlist "rockyou" where it is stored in kali linux
```bash
gzip -d /usr/share/wordlists/rockyou.txt.gz
```
Then in order to run the rockyou wordlist against the hash we obtained from our Wifi AP we run command  
```bash
aircrack-ng WPAcrackTest1-01.cap -w /usr/share/wordlists/rockyou.txt
```
<p align = center> <img src="/assets/ArticleImg/Wifi/crackComm.png" />    </p>

Once this final command is executed aircrack-ng will start to compare every password in the list to the password hash captured in the 4-way handshake until it finds a match or runs out of passwords to compare it against. 


<p align = center> <img src="/assets/ArticleImg/Wifi/crack.gif" />    </p>



As you can see below after around 4 minutes aircrack has found a matching hash for our password "coconut15" which is indeed the password that was set on my example router!

<p align = center> <img src="/assets/ArticleImg/Wifi/finalCrack2.png" />    </p>

The password cracking setup that we just used with aircrack-ng was a very simple wordlist attack using a relativly short wordlist and running it's attack on the computers CPU. This was a good enough setup to crack a short and commonly used password in a reasonable about of time as the password such as we have seen done here today.
  
If you would like to learn more about how to crack more difficult passwords that have uppercase and lowercase letters as well as numbers and special charaters stay tuned for my next article about password cracking using hashcat. That article will go in depth how using a more powerful GPU will increase your hashrate by a significant amount as well as creating your own targeted wordlist using open source intelligence and the syntax for creating rulesets to brute force passwords in hashcat. 

If you have any questions about this article or if I missed anything please reach out to me at DropperSec@protonmail[.]com



