Nmap or "Network Mapper" is a bulk network mapper giving in-depth details using various types of packets (including broken/invalid/corrupt ones). In simple essence, it is a port scanning tool that is able to determine and accurately guess vulnerabilities in a host that nmap is run on.

By default nmap performs a SYN scan against any complaint TCP stack. This avoids an platform specific issues. Nmap can be used to scan from 1 to thousands of hosts and determine the states of any of those systems

Nmap can be execute in console using the command <code>nmap</code> or with a GUI by entering <code>nmapfe</code> into a terminal

==Installation==
Nmap can be installed on most linux distros through the package manger
 <nowiki>
sudo apt-get install nmap #Ubuntu/Debian

sudo dnf install nmap #Fedora/CentOS</nowiki>

You can download a version for windows here: https://nmap.org/download.html

==Port States==
After a scan, nmap prints out a report of its findings along with the state of each port it has scanned. Each state has a specific meaning
{| class="wikitable"
!State
!Description
|-
|OPEN||An application on the target host is listening for connections/packets on that port
|-
|FILTERED||There is a firewall, or something else that is blocking the port and nmap cannot detect if it is open or closed
|-
|UNFILTERED||A port is classified as unfiltered when nmap cannot detect if it is open or closed; However, it is responsive to probes
|-
|CLOSED|| There is no application that is listening on that port
|}

Filtered vs Unfiltered is simply determined as to whether the packets sent by nmap are being dropped or not. It is a calculation based on response time and type of packets sent to the server

==Configuring Nmap Ports==
By default nmap will scan all ports inclusively up to 1024, and then higher numbered ports that are listed in the nmap-services file. We can specify what ports to scan using the <code>-p</code> flag
Additionaly by using <code>U</code and <code>T</code> syntax we can specify what protocol to use for each port as well.
 <nowiki>
nmap -p U:53,T:10-111 www.host.com
</nowiki>
The above example will scan port 53 on the UDP protocol and ports 10 - 111 on the TCP protocol. The end system that will be scanned is www.host.com

==Hiding From An IDS==
The main problem with nmap is that it is an exceptionally noisy program. Many intrusion detection systems (IDS) can pick up nmap scans immediatly. In the interest of stealthy reconnaisance, nmap has a number of features it mitigate if not remove chances of being detected by an IDS.

===Timing===
nmap scans can be timed to run at slower intervals. If the server is highly active, the mixing in of nmap traffic will be lost. An IDS may either not pickup on it due to the irregularity of the packets, or it may be identified as a false posotive by the network administrator

nmap has the following parameters to configure timing of a scan
{| class="wikitable"
!Timing Value (T)
!Type
!Description
|-
|1||Paranoid||Sends a packet every 5 seconds
|-
|2||Sneaky||Sends a packet every 15 seconds
|-
|3||Polite||Sends a packet every 0.4 seconds
|-
|4||Normal||Automatically, as quick as possible
|-
|5||Aggressive||Wait 1.2 seconds for a response before skipping
|-
|6||Insane||Waits for 0.3 seconds for a response before skipping
|}
Needless to say to avoid detection, using the Aggressive and Insane modes are highly not recommended. Additionally, Aggressive and Insane modes will skip ports if there is no response within the wait time, thus making the end resulting reports to contain errors.

Using the Paranoid and Sneaky options will create a highly reliable report and will reduce detection, but will take much longer to complete

An example of using the Paranoid mode in an nmap scan would look like this
 <nowiki>
nmap -T1 www.host.com</nowiki>

===Decoy Packets===
Decoy scanning is another option with nmap where it will send multiple spoof packets along with its own to mitigate IDS detection.

The syntax looks like this
 <nowiki>
nmap -D <decoy1[,decoy2][,ME],...></nowiki>

Where decoy1 and decoy2 are explicitly selected IPs that will be used as decoys. The keyword "ME" can then be used to represent nmaps own IP. By placing "ME" in the list of decoys, nmap will send the packets in the order specified with its own legitimate packet always in the same position relative to the decoys. If the keyword "ME" is not included, nmap will simply place its own IP in random positions in the decoy list.

Note that in order for decoy packets to work, the decoy IPs must belong to a legitimate system. If they are not, the host being scanned is likely to become a target of a SYN Flood and be bombarded with SYN/ACKS

The decoy packet option additionally has the ability to generate a random number of decoy packets to be used. These IPs are randomly calculated on the source host. As noted already above though, this option is strongly not recommended as you will be unable to determine whether the randomly generated IPs are valid or not. The syntax for nmap to generate random IP decoy packets is as follows
 <nowiki>
nmap -D RND:[numberofips] [host]</nowiki>


nmap fortunately though offers some tools for ensuring legitimate IPs are selected
 <nowiki>
nmap -iR 100 www.host.com</nowiki>
This will have nmap scan www.host.com but use up to 100 randomly chosen legitimate IP addresses to use as decoys during its scan. nmap will automaticaly randomize itself in with the packets

==Scanning Ping Dropping Hosts==
Many end systems have configured their firewalls to drop all ICMP ping requests. This poses a problem for nmap which by default, before each scan, attempts to ping the host system to determine if it is up. This allows it to skip hosts when scanning a network that may not have all of its IPs assigned. If the host blocks ping, nmap will end up skipping that host and not scanning it. Even by giving an explicit IP, by default if the ping request fails, the scan is skipped. To avoid this problem pass the <nowiki>-Pn</nowiki> parameter. This tells nmap to not ping the host initialy to determine if it exists, and to scan it regardless. When scanning a host you know exists, this is exceptionaly useful

An example of a scan with the initial ping removed may look like this:
 <nowiki>
nmap -Pn www.host.com</nowiki>

==Other Notable Flags==

{| class="wikitable"
!Flag
!Description
!Example
|-
| -O ||Enables OS Fingerprint Scanning||nmap -O www.host.com
|-
| -A ||Enable OS Fingerprint Scanning along with version detection, script scanning and traceroute||nmap -A www.host.com
|-
| -S ||Spoof The Source IP. Not recommended flag as nmap will then not receive the response||nmap -S 192.168.0.1 www.host.com
|-
| -6 ||Enable IPv6 Scanning||nmap -6 www.host.com
|-
| -sn ||Executes a Ping scan. No port scanning occurs. Useful for determining what hosts in a network are up||nmap -sn 192.168.0.1/24
|}

Full listing of flags can be found in the man pages aswell by typing <nowiki>man nmap</nowiki> in the terminal

==Notes==

==Sources==
http://linux.die.net/man/1/nmap
