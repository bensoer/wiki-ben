# Iptables

==Background==
Problems related to security and access can come from outside or inside. These can be malicious in intent, the result of "just looking around", or accidental. It is important to verify the cause and intent of the intrusion by careful examination of log files. Once the intrusion has been characterized, steps can be taken to block the appropriate addresses and ports using a firewall. We will discuss the various techniques for controlling access to a network and protecting critical components using firewalls. The following are some of the more important methods of attack and ways of protecting yourself against them:

===Unauthorized Access===
Unauthorized users outside your company attempt to connect to company file servers. Accessing an NFS server for example. These attacks can be avoided by carefully specifying who can gain access through services such as NFS and SAMBA

===Exploitation of Known Weaknesses In Applications===
Some applications and network services were not originally designed with strong security features and are inherently vulnerable to attack. Examples are remote access services such as rlogin, rexec, etc. The best way to protect against this type of attack is to disable any vulnerable services or severely restrict them.

===Denial Of Service===
Denial of service attacks are designed to overload a service and cause it to cease functioning or prevent others from accessing the service or program. These may be performed at the network layer by sending carefully crafted and malicious packets that cause network connections to fail. They may also be performed at the application layer, where carefully crafted application commands are given to a program that cause it to become extremely busy or stop functioning. Preventing network traffic from suspicious sources from reaching your hosts and preventing suspicious program commands and requests are the best ways of minimizing the risk of a denial of service attack.

===IP Spoofing===
IP Spoofing is a security exploit where an Intruder attempts to send packets to a system which appear to originate from a source other than the Intruder's own. If the target system already has an authenticated TCP session with another system on the same IP network, and it mistakenly accepts a spoofed IP packet, then it may be induced to execute commands in that packet, as though they came from the authenticated connection. The attacker pretends to be an innocent host by following IP addresses in network packets. For example, a well-documented exploit of the BSD rlogin service can use this method to mimic a TCP connection from another host by guessing TCP sequence numbers. Verifying the authenticity of packets and commands will prevent this attack. Prevent routing with invalid source addresses. Introduce unpredictability into connection control mechanisms, such as TCP sequence numbers and the allocation of dynamic port addresses.

===Eavesdropping===
A host is configured to "listen" to and capture data that is flowing on the network. Usernames and passwords are thus captured from user login network connections. Broadcast networks like Ethernet are especially vulnerable to this type of attack. One of the best ways to protect against this attack is to use of data encryption and secure connections (secure shell).

===IP Firewalling===
Firewalls are very effective in preventing or reducing unauthorized access, network layer denial of service, and IP spoofing attacks. They are not very useful in avoiding exploitation of weaknesses in network services or programs and eavesdropping. These can be mitigated using other techniques. A firewall is a secure and trusted machine running specialized software that is inserted between a private network and a public network. The firewall software is configured with a set of rules that determine which network traffic will be allowed to pass and which will be blocked. The most sophisticated firewall arrangement involves a number of separate machines and is known as a perimeter network. Two machines act as "filters" called chokes to allow only certain types of network traffic to pass, and between these chokes reside network servers such as a mail gateway or a World Wide Web proxy server. This configuration can be very safe and easily allows quite a great range of control over who can connect both from the inside to the outside, and from the outside to the inside. Typically though, firewalls are single machines that serve all of these functions. These are a little less secure, but cheaper and easier to manage than the more sophisticated perimeter networks. The Linux kernel provides a range of built-in features that allow it to function quite effectively as an IP firewall. The network implementation includes code to do IP filtering in a number of different ways, and provides a mechanism to dynamically design and implement the sort of rules that the firewall will apply to packets entering and leaving the network.

===IP Filtering===
This is simply a mechanism that decides which types of IP packets will be processed normally and which will be deleted and completely ignored, as if they had never been received.

We can apply different criteria to determine which packets are to be filtered:
* Protocol type: TCP, UDP, ICMP, etc.
* Socket number (for TCP/UPD)
* Datagram type: SYN/ACK, data, ICMP Echo Request, etc.
* Datagram source address.
* Datagram destination address.

It is important to understand that IP filtering is a network layer facility. This means it doesn't understand anything about the application using the network connections, only about the connections themselves. For example, you may deny users access to your internal network on the default telnet port, but relying on IP filtering alone will not stop them from using the telnet program with a port that you do allow to pass through your firewall. Using proxy servers for each service that you allow across your firewall can prevent this. The proxy servers understand the application they were designed to proxy and can therefore prevent abuses, such as using the telnet program to get past a firewall by using the World Wide Web port. The IP filtering ruleset is made up of many combinations of the criteria listed previously. For example, say you wanted to allow web users within your network to have no access to the Internet except to use other sites' web servers. You would configure your firewall to allow forwarding of:  
* Packets with a source address on your network, a destination address of anywhere, and with a destination port of 80.
* Packets with a destination address of your network and a source port of 80 from a source address of anywhere

We have used two rules here. We have allowed our data to go out, but also the
corresponding reply data to come back in. Linux simplifies this and allows us to
specify both rules in one command.

===Setting Up Linux Firewalling===
Most Linux distributions have high-level utilities to configure and deploy firewalls. As a means of acquiring an in-depth understanding of how a firewall functions, we will design and deploy firewalls using a low-level utility such as iptables. Linux kernels 2.3.15 and later support the fourth generation of Linux IP firewall called netfilter. The netfilter code is the result of a major redesign of the packet handling flow in Linux. The netfilter is a multifaceted utility, which can be configured using a utility called iptables.

==Iptables==
Iptables is actually only part of the mechanism that makes up the whole firewall system on Linux. There are two components that make it up in total: the kernel portion is known as netfilter and iptables is actually the extensible configuration tool that manages and runs netfilter.

===Basic Structure===
There are 3 main chains that make up the netfilter system. INPUT, OUTPUT and FORWARD. Each of these chains is for a specific use:
* INPUT chain is for packets that are destined FOR the host machine
* OUTPUT chain is for packets that are coming FROM the host machine - the source address of the packet is the machine running iptables
* FORWARD chain is for packets that are not destined for the iptables hosting machine that recieved it. This chain is used, as the name implies, for forwarding packets from the iptables machine to the given system. This is useful if iptables is being hosted on a router or gateway server that can then view the traffic going to and from the systems behind it and decide whether to let it through

netfilter then also has 3 unique tables. filter, mangle and nat. Each table can has its own predefined chains. Each table can only apply certain actions on the packets that pass through it. filter is the primary table that interacts with the INPUT,OUTPUT and FORWARD chains. By default iptables will apply its commands to the filter table, explicit naming of the other tables is needed to apply actions to those tables.

===The Filter Table===

==Saving Changes Permanently==
At restart, most systems will reset whatever has been put as the iptable rules with the system default. In order to keep the rules that were created we need to override the systems loading scripts. This can be done using <tt>iptables-save</tt> and <tt>iptables-persistent</tt>

The below demonstrates how to use these programs on Ubuntu and Debian, additional steps are needed for CentOS and other RHEL based platforms. Please see the sources at the bottom of this page for a link
to the full guidelines

===Exporting/Restoring Iptable Rules===
<tt>iptables-save</tt> and <tt>iptables-restore</tt> are tools that allow you to export and restore the current iptables rules to a file. Exporting for IPv4 and Ipv6 can be done simply as this
 <nowiki>
iptables-save > /etc/iptables/rules.v4
ip6tables-save > /etc/iptables/rules.v6</nowiki>

You can then do the inverse and restore the iptables from these files using <tt>iptables-restore</tt> with the following syntax
 <nowiki>
iptables-restore < /etc/iptables/rules.v4
ip6tables-restore < /etc/iptables/rules.v6</nowiki>

===Configuring Persistence===
As of Ubuntu 10.0.4 and Debian 6, <tt>iptables-persistent</tt> is available and compatible with automatically restoring the iptable rules. <tt>iptables-persistent</tt> requires that the ip table
rules be stored in <tt>/etc/iptables/rules.v4</tt> for IPv4 rules used by <tt>iptables</tt>, and <tt>/etc/iptables/rules.v6</tt> for IPv6 rules used by <tt>ip6tables</tt>

You can install iptables-persistent with the following command
 <nowiki>
sudo apt-get install iptables-persistent</nowiki>
This will install and configure the persistence module. During setup it will prompt as to whether to export the current iptables and ip6tables contents. If you have already done this via iptables-save,
then you can skip this step. Otherwise select yes, and iptables-persistent will automatically export the current iptables entries and generate the required directories and files to operate

==Sources==
https://www.thomas-krenn.com/en/wiki/Saving_Iptables_Firewall_Rules_Permanently

==Notes==
