# Raspberry Pi WPA2 Enterprise
Guide on how to connect your Raspberry Pi to a WPA2-Enterprise Network

# Introduction
The Raspberry Pi (RPi) is a low-cost Single Board Computer. Since it’s original release in 2012, a total of over 36 million devices have been sold so far, making it one of the best selling computers of all time. Despite the original objective to provide a low-cost computing platform enabling people to explore computing and learn to write code, the RPi owes its popularity to its low price, connectivity and versatility. This has allowed both professionals and hobbyist to make use of the RPi in various applications ranging from simple projects to full commercial products. 

In healthcare, recent advancements of sensor technology and the application of “internet of things” (IoT) devices hold a potential to transform healthcare. The use of these devices allows for improved insight of disease by gathering and exchanging data with other systems over the Internet without any human intervention. A good example is remote patient monitoring, which has seen a substantial uprise fueled by the recent COVID-19 pandemic. The application of IoT devices may have a substantial impact on reducing healthcare costs and improving treatment outcomes. However, prototyping these kind of solutions do demand all sorts of different sensors with different types of interfaces. The RPi may offer a low-cost and scalable platform to develop IoT solutions due to its abundant connectivity options (USB, GPIO, etc )and open source software which allows for easy integration of different protocols.  

Despite this, using a RPi in a hospital environment is not always easy. Especially when it comes to WiFi connectivity without a dedicated IoT-network. The RPi is shipped without an operating system (OS), allowing the end-user to choose their OS to suit their needs. Raspberry Pi OS is a linux-based operating system optimized for the RPi and provides an adequate choice OS for the RPi. There a multiple variations of the OS including 64- and 32-bit versions and versions with and without a graphic user interface (GUI). Raspberry Pi OS by default does not support enterprise WiFi security (WPA2-Enterpise and 802.1X), a common standard in hospital wireless networks.

# Solution

As Raspberry Pi OS is based on Linux, it can be configured to allow connectivity to WPA2-Enterprise network using the terminal and tweaking the network interface file, DHCP and WiFi config files.  Therefore, I put together this guide on how to modify these files allowing you to connect your RPi to a WPA2-Enterprise network.

# Guide

This is an instruction on how to connect your Raspberry Pi to a WPA2-Enterprise secured Wireless Network. This requires the use of terminal.


Disclaimer: Always inform you IT-department of your intentions

There are three steps involved:
**1. Modify the Network Interface File**

```
$ sudo nano /etc/network/interfaces
```

In the file, add the following lines:

```
auto lo
iface eth0 inet dhcp

auto wlan0
allow-hotplug wlan0
iface wlan0 inet dhcp
 pre-up wpa_supplicant -B -D wext -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
 post-down killall -q wpa_supplicant
```
Save your changes before exiting the file
You can review your changes by using $ cat /etc/network/interfaces without going into edit-mode


**2. Modify the DHCP Config File**

```
$ sudo nano /etc/dhcpcd.conf
```

Add the following lines:

```
interface wlan0
env ifwireless = 1
env wpa_supplicant_driver = wext, nl80211
```
Save your changes

Reboot your RPi. Upon restart the default network manager should now be disabled.

**3. Modify the WiFi Config File

```
$ sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

Again, add the following lines:
```
network={
  ssid="NAME OF THE NETWORK"
  priority=1
  proto=RSN
  key_mgmt=WPA-EAP
  pairwise=CCMP
  auth_alg=OPEN
  eap=PEAP
  identity="YOUR USERNAME"
  password="YOUR PASSWORD IN CLEAR TEXT"
  phase1="peaplabel=0"
  phase2="auth=MSCHAPV2"
}
```

**Optional**
Instead of clear text, a password hash can be used for better security.
```
echo -n TYPEYOURPASSWORD | iconv -t utf16le | openssl md4
```
Copy your Hash in the wpa_supplicant.conf
```
password=hash:COPYYOURHASHHERE
```
