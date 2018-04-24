# Wi-Fi & Wired Hacking 

All of the following requires tools that are default installed on Kali Linux. </br>
You can set up a VirtualBox instance with Kali and run as a NAT network to </br>
experiment with these tools. Use only on Networks that you have permission </br>
to pen test.

Virtual Machines to Install
```
https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/
// MSEdge on Windows 10 Stable -> Virtual Box
```

Open Settings
```
Set up VMs on a NAT network
Kali should get 1 GB Ram, 1 CPU
Windows should get 2 GB Ram, 2 CPU
```

Snapshots (Storing state of a Virtual Machine)
```
Click the Snapshot Button
Click Take
This way you can come back later
```

Basic Linux Operations
```
// Update the source list
apt-get update
// Install Terminator
apt-get install terminator
// Install needed updates
apt-get upgrade
```

## Networks

Series of connected devices </br>
One device is a server, which contains data shared between devices </br>
In Wi-Fi, the server is the router and data is the internet </br>
All data is sent as packets over the air </br>
Anyone in range can sniff these

### Connecting Wireless Adapters to Kali

Built in network cards will not work with VMs </br>
Even if Kali is a main machine, you need an adapter for more powerful features </br>
You need to download the VirtualBox Oracle VM Extension Pack to get USB features </br>
Add the device in settings -> ports -> USB </br>
With Kali running, VirtualBox -> Devices -> USB, make sure adapter is checked </br>

## Packet Sniffing

### MAC Address

- Media Access Control, A physical static address assigned by the network card manufacturer
- Used by devices to identify each other & transfer packets to the right place
- Each packet has a source & destination

#### Changing your MAC Address

MAC addresses are static, so they can only be changed temporarily in the RAM.</br>
This may help you avoid blacklists or align with whitelists.

```
$ iwconfig
$ ifconfig wlan0 down		// Disable wireless card
$ macchanger --help			// View macchanger help
$ macchanger --random wlan0	// Get a random MAC for wlan0
$ macchanger wlan0 up		// Enable wireless card
```

Two ways to capture packets
- Managed mode, which only gets packets with your MAC address
- Monitor mode, which grabs all packets within Wi-Fi range

### Airodump-ng

Part of the Aircrack-ng package. It's a packet sniffer that captures all traffic within
range. We can also scan and gather info about wifi networks around us.

```
// enable monitor mode
$ airmon-ng start [interface]
// start airodump-ng
$ airodump-ng [interface]
// stop monitor mode
$ airmon-ng stop [interface]
```

Manual monitor mode
```
$ ifconfig wlan0 down
$ iwconfig wlan0 mode monitor
$ ifconfig wlan0 up
$ airodump-ng wlan0
```

One more method
```
$ airmon-ng
$ ifconfig wlan0 down
$ airmon-ng check kill
$ airmon-ng start wlan0
$ iwconfig wlan0mon
```

Running airmon-ng
```
$ airodump-ng mon0
// Start sniffing traffic
BSSID - MAC address
PWR - How close the device is
Data - # of Useful packets
And various encryption forms

$ airodump-ng --channel [c] --bssid [b] --write [o] [interface]
// Example of specific target
// The second section contains clients associated with the access point
// The output will use 4 formats, including .cap, .csv & .xml
```

## Deauthentication Attacks

Disconnect any device from any network within the wi-fi range, even if the network
is protected with a key.

Hacker sends deauthentication packets to the router, pretending to be the target machine
by spoofing the MAC address.

Hacker simultaneously sends packets to the target machine (spoofing the router MAC address)
telling it to re-authenticate itself.

### Aireplay-ng

De-authenticate all clients
```
aireplay-ng --deauth [# of packets] -a [AP] [interface]
```

De-authenitcate specific client
```
aireplay-ng --deauth [# of packets] -a [AP] -c [target] [interface]
```

## Creating a Fake Access Point (honeypot)

Creating an open AP attracts many clients to automatically connect. We can sniff the
unencrypted traffic created by the clients.

Two cards required
- One network card needs to be connected to the internet
- Another card needs to broadcast as an access point (wi-fi card)










