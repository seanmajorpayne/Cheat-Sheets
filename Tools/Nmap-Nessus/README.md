# Network Scans with Nmap & Nessus

NAT Network Mode

Metasploitable Login Credentials
msfadmin
msfadmin

Kali Linux Credentials
root
toor

```
ifconfig
- Gets the interface configuration lists which shows IP addresses, MAC addresses, etc

netstat -tnlp
// Displays open TCP ports

ps aux
// View up and running services
```

## Scan Types

### Passive Scanning
Listen to network traffic
- Tcpdump
- Wireshark
- ARP tables

#### Wireshark

```
$ wireshark &	// The ampersand causes a task to run in the background
```

We want to run Wireshark on the eth0 interface since all of our devices will be on this
"wired" connection.

Try doing some pings internally from VMs to each other. Multiple VMs if possible.
Visit a few websites as well. We'll see plenty of packets populated in Wireshark.

Going to the Statistics -> Conversations menu item.
This is an overview of which IPs have communicated with each other, how many packets,
bytes, etc. 

In the ethernet tab, we see the MAC addresses of various systems. The addresses filled
with f's mean the traffic was broadcasted (ff:ff:ff:ff:ff:ff). ARP requests are the
examples for these packets.

In the TCP and UDP tabs, we see the IP addresses of Systems and the ports on which they 
communicated.

Conversation types: By default, 5 are selected, but you can select other types of
communication as well.

Right-Click and filter any traffic to get specific interactions.

#### ARP

ARP is a protocol used to map physical addresses (MAC) to virtual addresses (IP).

Computer A wants to talk to Computer C. It sends a broadcast to the network switch asking
"who has computer C's MAC address?"
```
ARP Request
From: 172.16.99.222 (00:D0:58:2D:6E:04)
To: 172.16.99.139 (FF:FF:FF:FF:FF:FF)
```

Every computer on the network hears this. The only system that responds is the computer
that has that MAC address (Computer C).
```
ARP Reply
From: 172.16.99.139 (00:02:16:DD:98:0D)
To: 172.16.99.222 (00:D0:58:2D:6E:04)
```

Each of the machines will start building an ARP table as these requests occur. They store
the IP & MAC addresses in their ARP table. These are generally short-lived ~15 minutes.
They are used so the computers don't have to continually do broadcast requests to
computers they are communicating with frequently. To view the ARP tables:

```
Mac
$ arp -an	// a - all, n - don't resolve to names, use numbers
Windows
$ arp -a
Linux
$ arp
```

### Active Scan
You scan on target systems
Sending packages. Leaves traces that you were there. Can alert admins that you are trying
to break in.
- Nmap
- Hping
- Scapy
- Ping, tracert, etc

Passive Scans will not reveal all the information you need. You won't see services that
aren't being used. Passive scans can also easily be fooled by sending out bogus information.

#### HPING
Command Line
TCP/IP packet analyser
Inspired to the "ping" command
Supports TCP, UDP, ICMP
	- Firewall Testing
	- Port Scanning
	- Remote OS Fingerprinting
	- DoS attacks

```
hping3 -h	// help
hping3 --scan 0-500 -S 172.16.99.139	// Scan ports 0-500 at this IP
port| serv name | flags | ttl | id | win | len
22	 ssh		 .S..A...   64   0   5840   46
```
Since we get back a SYN ACK packet, this shows the service is available.

```
hping3 --scan 0-500 -X 172.16.99.139	// Christmas Scan - Push, Urgent and Fin flags are set
All replies received. Done.
Not responding ports: ...
```
Since these are not valid packets, they all get dropped.

IP Spoofed DoS using Hping
```
hping3 --flood -S -V --rand-source www.owaspbwa.com		
// Sends packets as fast as possible. Syn packets so the server has to respond. 
// Verbose so we can see the results of sent packets. Random Source so every request seems
// like it is coming from different systems (Distributed DoS).
```

#### NMAP
Free & Open Source
Network Discovery & Security Auditing

What can be done with Nmap?
Host Detection
Port Scanning
Service and Version Detection
Operating System Detection
Firewall Detection
Vulnerability Assessment
Brute Force Attacks
Exploitation

```
nmap -n -sT 172.16.99.139 -p 22,23,80 --reason
// -sT is scan type, Syn is default
// Destination IP & ports
```

#### TCP/IP

The OSI Reference Model is useful for discussion, but it's rarely actually implemented.
Few tools & services keep things in well defined layers.

TCP

On the transport layer (Layer 4).
Connection oriented
Data Fragmentation
Error-Free
Flow Control

TCP 3-Way Handshake
```
Syn ->
<- Syn/Ack
Ack ->
Followed by Data Transfer
```

TCP Flags
```
1-bit flags
SYN, ACK, RST, FIN, PSH, URG
Syn - Synchronization, used as first step to establish a 3-way handshake
	- Only first packet from sender and receiver should have this
Ack - Acknowledgement
RST - Sent back when a packet is sent to a particular host that was not expecting a packet
FIN - Finished, no more data from the sender
PSH - Process these as they are received instead of buffering them
URG - Process the urgent packets
```

#### UDP

No handshake before connection </br>
No error checking (lost packets occur) </br>
Faster data transfer </br>
No validation mechanism (IP spoof possible) </br>
Some usage areas
- DNS
- DHCP
- Multimedia

### Nmap Ping Scan

Reduce the set of IP ranges into a list of active or interesting hosts </br>
No port scan -sn </br>
Only print out the available hosts that responded to the host discovery probes </br>

Default behavior for Privileged User
```
ICMP echo request
SYN -> TCP 443 port
ACK -> TCP 80 port
ICMP Timestamp request
* This is important to remember since this will affect host discovery
```

For Unprivileged User
```
SYN -> TCP 80, 443 ports
```

ARP Scan in local networks

```
nmap -sn 172.16.99.0/24 -n | grep "Nmap scan" | cut -d " " -f5
// Do an Nmap Scan only on active hosts in the 255 IP range and resolve names to IP numbers.
// Find the lines with Nmap scan at the beginning, cut these by spaces and grab the fifth
// column.
```

### Nmap Port Scans

#### SYN Scan

Most popular </br>
Quick </br>
Relatively stealthy since it never completes a TCP connection. This means the destination
does not log the transaction. </br>
Allows clear differentiation between open, closed & filtered states </br>
Also known as Half-Open scan </br>
Syn/Ack -> Port is listening </br>
RST -> Port is closed </br>

```
nmap -sS 172.16.99.0/24 --top-ports 50
// Syn scan on top 50 ports in 255 IP range

In Wireshark
ip.addr==172.16.99.139 && tcp
// You can watch a Syn Scan and see the packets being sent

nmap -sS 172.16.99.139 -p80
```

Different Port States
```
Open
SYN ->
<- SYN/ACK
RST ->

Closed
SYN ->
<- RST/ACK

Filtered
Syn ->
No response or <- ICMP Unreachable (Both are firewall behaviors)

For UDP, no response would indicate open|filtered
```

Some Common Nmap Scans

```
nmap -sS 172.16.99.206
// top 1000 scanned by default
nmap -sS 172.16.99.206 -p22,80,100-200
// Scan by port #
nmap -sS -sU 172.16.99.206 -pT:80,443,U:53,139-150
// Scan with UDP as well, scan specific ports for TCP/UDP
nmap -sS 172.16.99.206 --top-ports 20
// Scan top 20 ports
nmap -sS 172.16.99.206 -F
// Top 100 ports
nmap -sS 172.16.99.206 -p1-65535
// Scan all ports, used for pen testing of course!
nmap -sS --top-ports 10 --open 172.20.1.0/24 -Pn -n
// Skips host discovery and scan all target hosts (i.e. 255 IPs)
```

If a system is configured not to answer ICMP requests and 80/443 filtered, nmap will think
the host is down (even if it's up). This is the reason for -Pn.

#### TCP Scan

-sT
If you aren't a privileged user, you can't interrupt a 3 way handshake, since it works at
the OS level.

TCP connect scans will work as a non-privileged user since you don't interrupt the handshake.
But, you are also completing a connection, so the destination will log the interaction.

SYN -> </br>
<- SYN/ACK </br>
-> ACK </br>
-> RST </br>

```
nmap -sT -n -Pn 172.16.99.206 --top-ports 10
```

#### UDP Scan

-sU </br>
Takes a long time (timeouts) </br>
Important ports: 53, 69, 67-68, 123, 161-162 </br>
Sends empty UDP packets in general </br>
Should run with version detection for more accurate results </br>
Exploitable UDP services are quite common </br>

```
nmap -n -Pn -sU 172.16.99.206 --top-ports 10 -sV --reason
// Resolve names to #s, don't run host discovery, UDP, IP, top 10 ports, Version detection
// give reason for connection/failure.
```

UDP Port States
```
Open
UDP Packet ->
<- UDP Response

Closed
UDP Packet ->
<- ICMP Unreachable

Filtered
UDP Packet ->
<- Other ICMP Unreachable

Open|Filtered
UDP Packet ->
No response
```

### Version and OS Detection
```
nmap -n -Pn -sS 172.16.99.206 --top-ports 10
// Without version Detection
nmap -n -Pn -sS 172.16.99.206 --top-ports 10 -sV
// With Version Detection
```

Version detection is important, because Nmap will mislabel services running on non-default
ports.

```
$ netstat -tnlp
// We can see SSH is running
$ service ssh stop
// stop the service
$ nano /etc/ssh/sshd_config
// Change the port # to 443 & uncomment
$ service ssh start
$ nmap -n -sS 172.16.99.222 -p443
// nmap will report https! This is not correct.
$ nmap -n -sS 172.16.99.222 -p443 -sV
// nmap will report ssh with version. This is correct.
```

Operating System detection

```
nmap -sS -n -Pn 172.16.99.139 --top-ports 100 -O
nmap -sS -n -Pn 172.16.99.139 --top-ports 100 -O --osscan-guess
// For more aggressive checks
```

If all ports are filtered, then Nmap won't have a way to identify the OS. It needs 1 open
TCP port and 1 closed to make an accurate assessment.

### Input/Output Management

```
nmap -n -Pn -sS --top-ports 3 172.16.99.0/24
nmap -n -Pn -sS --top-ports 3 172.16.1-255.0-255
nmap -n -Pn -sS --top-ports 3 172.16.99.0/24 10.0.0.0/16
```

For real world scenarios, you'll want to create a list of known IPs with a ping scan and
save them in a file for multiple uses.

```
nmap -sn 172.16.99.0/24 | grep "Nmap scan" | cut -d " " -f5 > ipList.txt
nmap -sS -iL ipList.txt
```

Output Saving Formats

```
-oN: Normal (Readable - What you see on the screen)
-oG: Grepable (parsing)
-oX: XML
-oA: Save in all formats
```

Example

```
nmap -n -Pn -sS 172.16.99.139,206 --top-ports 3 -oX resX.xml
nmap -n -Pn -sS 172.16.99.139,206 --top-ports 3 -oA resAll
```

## Script Scanning

-sC is default </br>
Network Discovery </br>
Sophisticated Version Detection </br>
Vulnerability Detection </br>
Backdoor Detection </br>
Vulnerability Exploitation </br>

--script for specific types of scripts </br>
--script "default and safe"

```
nmap --script-updatedb
// Update the script database
locate *.nse | grep telnet
```

## Bypassing IPS/IDS

Timing - Extend the duration between packets, Disable parallel scanning </br>
Fragmentation -f, Split up TCP header over several packets. Generally not supported by 
Version detection or Scripts. </br>
Source Port - --source-port, using ports that services need to support 
(i.e. 80, 443, etc). This means sending FROM the port on your machine  </br>
Randomised Scanning Order --randomize-hosts  </br>
IP Spoofing - Usually won't receive replies -S  </br>
Firewall and IPS/IDS detection - TTL (Time to Live), --badsum. See difference between two 
packet transport times. Badsum is usually not verified by firewalls, so if they are sent 
back, it's probably a firewall
		
### Timing

You can use names or numbers
```
-T0 (paranoid) 	- 5 min
-T1 (sneaky) 	- 15 sec
-T2 (polite)	- .4 sec
-T3	(normal)	- Default, parallel scan
-T4	(aggressive)
-T5	(insane)

--max-retries 2
Number of retries when there is no answer, when Nmap receives no response it's either
filtered or lost. Rate limiting may temporarily block as well. Limiting this # can
reduce time of scan.

--host-timeout 30m
Max wait duration on a host
```

Closing Parallel Scanning
```
-T 0|1|2
--scan-delay 1
--max-parallelism 1
--max-hostgroup 1
```

## Other Nmap Scans

### NULL, FIN, XMAS Scans (-sN, -sF, -sX)
Packets without SYN, ACK & RST Flags set </br>
NULL Scan - No flag is set </br>
Fin Scan - Only FIN flag is set </br>
XMAS Scan - FIN, PSH, URG flags are set </br>

States of These Scans
```
Closed
<- RST Packet returned

Open|Filtered
No Response

Filtered
<- ICMP Unreachable
```

### Ack Scan (-sA)

Send a Packet with ACK Flag </br>
Cannot know if port is opened or closed </br>
Used to detect filtering </br>

ACK Scan States
```
Unfiltered (closed or open)
<- RST

Filtered
No return

Filtered
<- ICMP Unreachable
```

Idle Scan (-sl)

Truly blind TCP port scan </br>
No packets from you </br>
Zombie host to gather information </br>

```
Open
Attacker -> SYN/ACK -> Zombie
Attacker <- RST <- Zombie
Attacker (Spoofing Zombie) SYN -> Target
Zombie <- Syn/Ack <- Target
Zombie -> RST -> Target
Attacker -> SYN/ACK -> Zombie
Attacker <- RST <- Zombie
Packet IP ID # has incremented by 2

Closed
Attacker -> SYN/ACK -> Zombie
Attacker <- RST <- Zombie
Attacker (Spoofing Zombie) SYN -> Target
Zombie <- RST <- Target
Zombie (No Reponse)
Attacker -> SYN/ACK -> Zombie
Attacker <- RST <- Zombie
Packet IP ID # has incremented by 1

Filtered
Attacker -> SYN/ACK -> Zombie
Attacker <- RST <- Zombie
Attacker (Spoofing Zombie) SYN -> Target
Target (No Response)
Attacker -> SYN/ACK -> Zombie
Attacker <- RST <- Zombie
Packet IP ID # has incremented by 1

Note the filtered is not distinguished from the closed.

Unexpected SYN/ACK is responded to with a RST
Every packet has an IP ID, which is simply incremented by most OSs
```

You can determine machines that have suitable IP ID Sequencing with the IPIDSEQ script.
Incremental Sequencing machines can be used as a Zombie.

```
nmap --script ipidseq 172.16.99.0/24 --top-ports 2
// Example results
Host script results:
|__ipidseq: Randomized

Host script results:
|__ipidseq: Incremental!
```

Example Idle Scan

```
nmap -sI 172.16.99.2 -Pn -n 172.16.99.206 --top-ports 3
// First IP is the zombie, second is the target
```

## Vulnerability Scans








