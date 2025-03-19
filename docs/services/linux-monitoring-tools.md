# Linux Monitoring Tools

There are a number of tools in Linux that can be used to gather information about your system and determine if it is or has been hacked in some way

==ifconfig/ip address==
<code>ifconfig</code> or <code>ip address</code> can give a summary overview of the network cards on the system

By default ifconfig will report the status of all active network cards. To view all cards, active or inactive include <code>-a</code>

An example of <code>ifconfig -a</code> printout could look like this:
 <nowiki>
eth0 Link encap:Ethernet HWaddr 00:01:02:45:45:5B
   inet addr:192.168.1.20 Bcast:192.168.1.255 Mask:255.255.255.0
   UP BROADCAST RUNNING PROMISC MULTICAST MTU:1500 Metric:1
   RX packets:1449310 errors:0 dropped:0 overruns:0 frame:0
   TX packets:19866 errors:0 dropped:0 overruns:0 carrier:0
   collisions:232 txqueuelen:100
   Interrupt:5 Base address:0xb400
lo Link encap:Local Loopback
   inet addr:127.0.0.1 Mask:255.0.0.0
   UP LOOPBACK RUNNING MTU:16436 Metric:1
   RX packets:70 errors:0 dropped:0 overruns:0 frame:0 
   TX packets:70 errors:0 dropped:0 overruns:0 carrier:0
   collisions:0 txqueuelen:0</nowiki>

An important note is to make sure your card is not in <code>PROMISC</code> mode as the sample readout shows for eth0. PROMISC mode means that the card, instead of only listening for packets addressed to it, it is listening to all packets on the network. In terms of hacking, many hackers will do this so as to try to collect important data within your network that the system is connected to. Unless you are running an intrusion detection system on the local system, this mode should not be appearing

==Checking Network Connectivity with ping==
Ping is commonly disabled on servers as hackers have notoriously used it for simple DOS attacks. It's also used for reconnaissance in mapping out the network they may be interested in. It is not uncommon to ping a destination and get no results back. That being said ping is a useful tool for testing connectivity whether be the end host or interfaces in between

You can execute ping by calling:
 <nowiki>
ping <ip or domain of destination></nowiki>

You can also limit how many hops ping will make between it and its destination (TTL - time to live) by using the <code>-t</code> flag followed by the number of hops to limit it by

==Checking Network Connectivity with traceroute==
Traceroute is a tool that shows all of the routers a request went through and thier latencies before arriving at its destination. It can be used to determine why a server may be slow, and if that may be caused by congestion at certain routers in the network

Traceroute also allows use to get an interesting view of how ISP's and the target host are connected to eachother and connected to the internet

You can execute traceroute like this:
 <nowiki>
traceroute <ip or domain of destination></nowiki>

http://www.visualroute.com/ is an interesting tool that lets you visualize with a GUI traceroute and placement of routers (although not 100% accurate)

==Checking Network Prrocesses with netstat==

netstat displays protocol statustics and current TCP/IP connections

===Tags===

{| class="wikitable"
!Tag
!Use
|-
| -n || list all connections by IP rather then host names. This is useful if name resolutions are slow or are failing
|-
| -s || allows you to view per-protocol statistics for TCP, UDP, ICMP and IP
|-
| -p <protocol> || allows you to view information from a specific protocal
|-
| -a || lists all ports that are either in active use or being listened to by local servers
|-
| -A || specifies the adddress family reported. The listing uncludes the ports in use as they are associated with your network infterface cards. Local UNIX address family socket connections aren't reported, including local network-based connections in use by programs, such as any X Window program you might have running
|}

Some examples with these tags include:
 <nowiki>
netstat -s -p icmp</nowiki>
This call will show us statistics for ICMP
 <nowiki>
netstate -s -p ip 3</nowiki>
This example will show statistics about IP protocols. Additionaly it will also auto update itself every 3 seconds

An extremely useful call to view all internet connections on your system is:
 <nowiki>
netstate -anp --ip</nowiki>
Note that this only looks at internet connections using the <code>AF INET</code> socket. AF INET is a unix socket for TCP/IP sockets used across a network. The alternative <code>AF UNIX</code> is a socket type local to the kernel. It is primarily used for interprocess communication but nothing goes out to the network through it

===Generated Report Notes===
When netstat prints out its  report there are a few interesting things to note about it:

* '*' under the Foreign or Local Address means the interface is listening on all network interfaces rather than on just a single interface. The port is either the symbolic or numberic service port identifier the server is using
* Send-Q and Recv-Q are the number of bits received through the connection but that have not been passed on to the local program
* State information is useful to identify the state of the connection. The possible states include : <code>LISTEN</code>, <code>SYN SENT</code>, <code>SYN REVC</code>, <code>ESTABLISHED</code>, <code>FIN WAIT</code>, <code>FIN SENT</code>
** If your netstat report returns with allot of connections in <code>TIME_WAIT</code> status, this is an indication that someone may be creating useless TCP connections and attempting a DDOS
** Typical TCP States Include: <code>SYN_SENT</code>, <code>ESTABLISHED</code>, <code>FIN_WAIT_1</code>, <code>FIN_WAIT_2</code>, <code>TIME_WAIT</code>, <code>CLOSED</code> In that order

==Checking Processes with ps==
ps reports on process status. As with netstat you should be familiar with every program running on your system and why it is running there. With a properly secured system, you should be able to identify every process listed in the ps report. You should not see any processes you can not identify

===Tags===
{| class="wikitable"
! Tag
! Use
|-
| -a || Selects all processes with controlling terminals, usually user programs running interactively in the foreground
|-
| -x || Select processes without controlling terminals, usually permanent system daemons running automatically in the background
|-
| -u || Optional parameter that produces additional user-oriented information including the user login name
|}

===Examples===
 <nowiki>
ps -aux | more</nowiki>
This will show all processes, including <code>init</code> which is the parent of all processes and it always runs

Note you may also see processes called <code>bdflushd</code> which is in charge of flushing modified file system buffers back to disk or <code>kswapd</code> which is a kernel thread that selects physical memory pages for swapping from memory to swap space on disk to free memory for other processes)

==Monitor System Resources with lsof==
lsof (Standing for LiSt Open Files) allows you to view current network connections and the files associated with them. Mainly you can interrogate a specific port with it to determine what exectly is running/connected/using that port on the system

losof provides a very verbose output. calling <code>lsof </code> in console will printout all open files corresponding to every active process on the host. This is quite lengthy so its best to narrow the list down with a few filtering tags

 <nowiki>
isof -i</nowiki>
Will print out all connections through the internet in a similar format to <code> netstat -a -p</code>. This is particularly useful when wanting to secure a Linux system as it allows us to view all open ports. Also lsof will list with the programs name, allowing us to find and shutdown unwanted programs easily.

You can narrow lsof down with the following filters
 <nowiki>
lsof -i:587
lsof -i:smtp
lsof -i:@somewhere.remote.net</nowiki>

* The first example will generate a report looking at whatever is connected or using port 587
* The second example will generate a report looking at what connections the well know service <code>smtp</code> is making
* The third example will generate a report for connections going to and from the host somewhere.remote.net

Additionaly you can further filter by protocol and port with the following call with lsof:
 <nowiki>
lsof -i tcp:5555</nowiki>
This will generate a report of all connections using TCP and port 5555

You can even use lsof to view all connections of a specific PID:
 <nowiki>
lsof -p 505</nowiki>
This will print out a report of all the connections being made by that process, including the file descriptors (listed under the column FD)

The file descriptors are defined as follows
{| class="wikitable"
!Descriptor
!Use
|-
| cwd || Variable represents the current working directory of the process
|-
| txt || Defines the program text, which is executable itself
|-
| mem || Is a file held in memory
|-
| 4 OR 21 || Represents a file in use by a particular process. If these values are followed by a 'u' it means they have both read and write access
|}

Using all of this together you can determine whether something physically exists on the system, or is simply being held in memory

==Using RSysLogs==
See [[RSysLog]]

==Top and HTop==
top and htop are essentialy identical programs that show the top running process on a machine. They are simply augment tools for ps. htop adds extra functionality to top
though in adding also live stat information about CPU usage, Memory consumption and Swap space usage along with the top programs.

top is included in most linux distributions, htop is avalable in both dnf and apt-get package managers

==Notes==

==Sources==
