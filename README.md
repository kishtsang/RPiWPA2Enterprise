# Raspberry Pi WPA2 Enterprise
Guide on how to connect your Raspberry Pi to a WPA2-Enterprise Network

# Introduction
The Raspberry Pi (RPi) is a low-cost Single Board Computer. Since its original release in 2012, a total of over 37 million devices have been sold so far, making it one of the best-selling computers of all time. Despite the original objective to provide a low-cost computing platform enabling people to explore computing and learn to write code, the RPi owes its popularity to its low price, connectivity, and versatility. This has allowed both professionals and hobbyists to make use of the RPi in various applications ranging from simple projects to full commercial products. 

In healthcare, recent advancements in sensor technology and the application of “internet of things” (IoT) devices hold the potential to transform healthcare. The use of these devices allows for improved insight into disease by gathering and exchanging data with other systems over the Internet without human intervention. A good example is remote patient monitoring, which has seen a substantial uprise, fuelled by the recent COVID-19 pandemic. The application of IoT devices may have a considerable impact, reducing healthcare costs and improving treatment outcomes. However, prototyping these kinds of solutions does require all sorts of different sensors with different types of interfaces. The RPi offers a low-cost and scalable platform to develop IoT solutions due to its abundant connectivity options (USB, GPIO, etc.) and open-source software which allows for easy integration of different protocols.  

Despite this, deployment of Raspberry Pi’s in a hospital environment for prototyping is not an easy task. Especially when it comes to WiFi connectivity without a dedicated IoT network. Therefore, despite its potential, the use of RPi in clinical settings has been moderate so far.,,, However, there is a solution to this problem.

# Solution

The RPi is shipped without an operating system (OS), allowing the end-user to choose their OS to suit their needs. Raspberry Pi OS (formerly known as Raspbian) is a Linux-based operating system optimized for the RPi. There are multiple variations of the OS, including 64- and 32-bit versions and versions with and without a graphic user interface (GUI). By default, Raspberry Pi OS does not support enterprise WiFi security (WPA2-Enterprise and 802.1X), a common standard in hospital wireless networks.

As Raspberry Pi OS is based on Linux, it can be configured to allow connectivity to WPA2-Enterprise networks using the terminal and tweaking the network interface file, DHCP- , and WiFi-config files. Therefore, I put together this guide on how to modify these files allowing you to connect your RPi to a WPA2-Enterprise network.

# Guide

This guide involves three steps:

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

Reboot your RPi. Upon restart you will notice (in the GUI) that the default network manager of Raspberry Pi OS has been disabled.

**3. Modify the WiFi Config File**

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
  identity="YOUR USERNAME" *
  password="YOUR PASSWORD IN CLEAR TEXT"
  phase1="peaplabel=0"
  phase2="auth=MSCHAPV2"
}
```
* Quotation marks are required

**Optional**
Instead of clear text, a password hash can be used for better security.
```
echo -n TYPEYOURPASSWORD | iconv -t utf16le | openssl md4
```
Copy your Hash in the wpa_supplicant.conf
```
password=hash:COPYYOURHASHHERE
```


