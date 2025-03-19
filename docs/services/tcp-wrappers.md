# TCP Wrappers

TCP Wrappers are one of the most common ways to control access on a Unix or Linux System. The wrapper wraps around existing daemons and interfaces between clients and the server. This provides a layer of security between the external world and your service provided by your Linux/Unix system.

Xinetd is the fundamental program that started this movement. Any service managed by xinetd (aswell any preogram with build-in support for libwrap) can use TCP wrappers to manage access

==Directories==
Hosts.Allow: <code>/etc/hosts.allow</code> <br>
Hosts.Deny: <code>/etc/hosts.deny</code> <br>

Xinetd Super Server Configuration: <code>/etc/xinetd.conf</code>


==Xinetd==
xinetd can use the hosts.allow and hosts.deny files to configure access to system services. hosts.allow lists hosts allowed to access services controlled by xinetd and hosts.deny lists hosts denied from access to services controlled by xinetd.

hosts.allow has precedence over hosts.deny. There-for a host listed in both files or neither files will be given access. Hosts can be based on individual IP addesses or hostnames, or on a pattern of clients. The wrapper <code>tcpd</code> checks these hosts.allow and hosts.deny files to determine whether to allow access. As a system administrator you can configure the rules in these files to allow or deny each xinetd-managed service to specific hosts, domains or subnets

When a connection is initiated on one of the ports xinetd is bound to, xinetd looks up the service name by its port number in <code>/etc/services</code>, then the configuration of that service in <code>/etc/xinetd.conf</code>. If <code>tcpd</code> is listed as the server executable to run, its argument (the daemon's executable name) is looked up first in hosts.allow. If access is permitted by rules in that files, access to the service is allowed. Then, if access is denied by the rules in hosts.deny the connection is dropped. If no rule matches in either file, access to the service is allowed.

The parsing daemon stops at the first match by inspecting the files in this order:
#hosts.allow
#hosts.deny
#default: Allow

Because the default configuration is to allow all server requests, you must edit the hosts.access and hosts.deny files to restrict access. A typical and very simple configuration is to deny ALL service to ALL, but allow ALL service to hosts on your local subnet

===Permissive Strategy===
Leave hosts.allow empty and restrict access in the hosts.deny

===Paranoid Strategy===
In hosts.deny deny ALL (ie. ALL:ALL) and then configure hosts.allow to permit access


----
The following are important points to consider when using TCP wrappers to protect network services:
* Because access rules in hosts.allow are applied first, they take precedence over rules specified in hosts.deny.
* Therefore, if access to a service is allowed in hosts.allow, a rule denying access to that same service in hosts.deny is ignored.
* Since the rules in each file are read from the top down and the first matching rule for a given service is the only one applied, the order of the rules is extremely important.
* If no rules for the service are found in either file, or if neither file exists, access to the service is granted.
* TCP wrapped services do not cache the rules from the hosts access files, so any changes to hosts.allow or hosts.deny take effect immediately without restarting network services.

===Formatting Access Rules===
The format for both the hosts.allow and hosts.deny files are identical. Any blank lines or lines that start with a hash (#) are ignored and each rule must be on a new line. A rule basicily looks like a colon separated  rule with the daemon name and the user's who have permission

The basic syntax looks like this:
 <nowiki>
<daemon list>: <client list> [: <option>: <option>: ...]</nowiki>

{| class="wikitable"
! Attribute
! Value / Definition
|-
| <daemon list> || A comma separated list of process names (not service names) or the ALL wildcard
|-
| <client  list> || A comma seperated list of hostnames, host IP address, special <i>patterns</i> or special wildcars that identify the hosts effected by the rule
|-
| <options> || An optional action or colon separated list of actions performed when the rule is triggered. Option fields support expansions), launch shell commands, allow or deny access, and alter logging behavior.
|}

===Examples===
 <nowiki>
vsftpd : .example.com</nowiki>
This rule instructs TCP wrappers to watch for connections to the FTP daemon (vsftpd) from any host in the example.com domain. If this rule appears in hosts.allow, the connection will be accepted. If this rule ppears in hosts.deny, the connection will be rejected.
 <nowiki>
sshd : .example.com : spawn /bin/echo `/bin/date` access denied>>/var/log/sshd.log : deny</nowiki>
This example uses two options fields

The above rule states that if a connection to the SSH daemon (sshd) is attempted from a host in the example.com domain, execute the echo command (which will log
the attempt to a special file), and deny the connection. Because the optional deny directive is used, this line will deny access even if it appears in the hosts.allow file.

===Wildcards===

Wildcards allow TCP wrappers to more easily match groups of daemons or hosts.
They are used most frequently in the client list field of access rules.

The following wildcards may be used:
{| class="wikitable"
! Wildcard
! Use
|-
| ALL || Matches everything. It can be used for both the daemon list and the client list.
|-
| LOCAL || Matches any host that does not contain a period (.), such as localhost.
|-
| KNOWN || Matches any host where the hostname and host address are known or where the user is known.
|-
| UNKNOWN || Matches any host where the hostname or host address are unknown or where the user is unknown.
|-
| PARANOID || Matches any host where the hostname does not match the host address.
|}

===Patterns===

Patterns can be used in the client list field of access rules to more precisely specify groups of client hosts. These are mainly alternate varying shorthands to specify groups of client hosts.

====Period====
Placing a period (.) at the beginning of a hostname, matches all hosts sharing the listed components of the name.

 <nowiki>
ALL : .example.com</nowiki>
This example will match any host within the example.com domain

 <nowiki>
ALL : 192.168.</nowiki>
You can also place the period at the end. In the above example all IP's that start with <code>192.168</code> will be matched with this rule. Note that alternatively to this, netmask expressions are also acceptable and work the exact same. The following example is equivelent to the above example:
 <nowiki>
ALL : 192.168.0.0/255.255.254.0</nowiki>

Note though that prefix length style is not supported. The following example WILL NOT work:
 <nowiki>
ALL : 192.168.0.0/16</nowiki>

====Asterix====
The asterix (*) can be used to match entire groups of hostnames or IP address, as long as they are not mixed in a client list containing other types of patterns

The following example will match any host within the example.com domain
 <nowiki>
ALL : *.example.com</nowiki>

====Slash====
The slash (/) is specificaly used to identify a file name. This can be useful if rules specifying large numbers of hosts are necessary. The following example refers TCP wrappers to the <code>/etc/telnet.hosts</code> file for all Telnet connections:

 <nowiki>
in.telnetd : /etc/telnet.hosts</nowiki>

This way you can essentially segregate out permissions to other files

===Operators===
Currently there is only one operator <code>EXCEPT</code> which can be used in both the daemon list and the client list of a rules.

This <code>EXCEPT</code> rule allows specific exceptions to broader matches within the same rule.

In the following example from a hosts.allow file, all example.com hosts are allowed to connect to all services except cracker.example.com:
 <nowiki>
ALL: .example.com EXCEPT cracker.example.com</nowiki>
In the another example from a hosts.allow file, clients from the 192.168.0.x network can use all services except for FTP:
 <nowiki>
ALL EXCEPT vsftpd: 192.168.0.</nowiki>

===Logging===

Xinetd has option fields, and these can also be used to change the logging priority of a given rule. This can be done using the <code>severity</code> directive.

The following example, connections to the SSH daemon from any host in the example.com domain are logged to the default <b>authpriv</b> facility (because no facility value is specific) with a priority of <b>emerg</b>
 <nowiki>
sshd : .example.com : severity emerg</nowiki>

As noted you can also specify a facility group by specificying it in the <code>severity</code> directive. The following example logs any SSH connection attempts by hosts from example.com domain to the local0 favility with a priority of alert:
 <nowiki>
sshd : .example.com : severity local0.alert</nowiki>

Note though in practice, this example will not work until the syslog daomon (syslogd) is configured ot loc to the local0 facility. Some additional configuration is needed for this to work as expected

===Access Control===

Option fields also allow administrators to explicitly allow or deny hosts in a single rule by adding the allow or deny directive as the final option.

For instance, the following two rules allow SSH connections from client-1.example.com, but deny connections from client-2.example.com:
 <nowiki>
sshd : client-1.example.com : allow
sshd : client-2.example.com : deny</nowiki>
By allowing access control on a per-rule basis, the option field allows administrators to consolidate all access rules into a single file: either hosts.allow or hosts.deny. This is a useful consideration to make when using a Permissive or Paranoid security strategy with the hosts.allow and hosts.deny files

===Shell Commands===
Option fields allow access rules to launch shell commands through the following two directives:
* spawn — Launches a shell command as a child process.
This option directive can perform tasks like using /usr/sbin/safe_finger to get more information about the requesting client or create special log files using the echo command.

In the following example, clients attempting to access Telnet services from the example.com domain are quietly logged to a special file. Note the use of the <code>%h</code> expansion which is explained in the next section:
 <nowiki>
in.telnetd : .example.com : spawn /bin/echo `/bin/date` from %h>>/var/log/telnet.log : allow</nowiki>

* twist — Replaces the requested service with the specified command.
This directive is often used to set up traps for intruders (also called "honey pots"). It can also be used to send messages to connecting clients. The twist command must occur at the end of the rule line.

In the following example, clients attempting to access FTP services from the example.com domain are sent a message via the echo command:
 <nowiki>
vsftpd : .example.com : twist /bin/echo "Rabbit Hole Ahead, Turn Back!" </nowiki>

===Expansions===

Expansions, when used in conjunction with the spawn and twist directives provide information about the client, server, and processes involved.

The following is a list of supported expansions:
{| class="wikitable"
! Expansion
! Use
|-
| %a || The client's IP address.
|-
| %A || The server's IP address.
|-
| %c || Supplies a variety of client information, such as the username and hostname, or the username and IP address.
|-
| %d || The daemon process name.
|-
| %h || The client's hostname (or IP address, if the hostname is unavailable).
|-
| %H || The server's hostname (or IP address, if the hostname is unavailable).
|-
| %n || The client's hostname. If unavailable, unknown is printed. If the client's hostname and host address do not match, paranoid is printed.
|-
| %N || The server's hostname. If unavailable, unknown is printed. If the server's hostname and host address do not match, paranoid is printed.
|-
| %p || The daemon process ID.
|-
| %s || Various types of server information, such as the daemon process and the host or IP address of the server.
|-
| %u || The client's username. If unavailable, unknown is printed.
|}

The following sample rule uses an expansion in conjunction with the spawn command to identify the client host in a customized log file. It instructs TCP wrappers that if a connection to the SSH daemon (sshd) is attempted from a host in the example.com domain, execute the echo command to log the attempt, including the client hostname (using the %h expansion), to a special file:
 <nowiki>
sshd : .example.com : spawn /bin/echo `/bin/date` access denied to %h>>/var/log/sshd.log : deny</nowiki>

Similarly, expansions can be used to personalize messages back to the client. In the following example, clients attempting to access FTP services from the example.com domain are informed that they have been banned from the server:
 <nowiki>
vsftpd : .example.com : twist /bin/echo "%h has been banned from this server!"</nowiki>

'''***Note that in practice one should not antagonise a hacker. The user should not be informed about anything of thier status on the server
'''

==Xinetd Super Server==

The exenteded internet services daemon (xinetd) has a number of extended capabilities the extend further then the TCP Wrappers and portmapper tools explained in the previous section. The Xinetd Super Server is also substantially easier to manage and more user friendly

===Advantages of the Xinetd Super Server===
* xinetd provides access control in a way that is quite similar to TCP_wrappers or the portmapper.
* It provides access control for TCP, UDP, and RPC services.
* Implements access limitations based on time.
* Extensive logging capabilities for both successful and unsuccessful connections.
* Provides transparency to both the client host and the wrapped network service. Both the connecting client and the wrapped network service are unaware that TCP wrappers are in use.
* Legitimate users are logged and connected to the requested service while connections from banned clients fail.
* Centralized management of multiple protocols. — TCP wrappers operate separately from the network services they protect, allowing many server applications to share a common set of configuration files for simpler management.
* Provides for hard reconfiguration by killing services that are no longer allowed.
* Provides numerous mechanisms to prevent DoS attacks.
** Provides a compile-time option to include libwrap, the TCP_wrappers library.
** Limit on the number of daemons of a given type that can run concurrently. An overall limit of processes forked by xinetd.
** Limits on log file sizes.
* Provides a compile-time option to include libwrap, the TCP_wrappers library.
** Causes /etc/hosts.allow and /etc/hosts.deny access control checks in addition to xinetd access control checks.
Provides for the invocation of tcpd.
** All TCP-wrappers functionality is available.
* Services may be bound to specific interfaces.
* Services may be forwarded (proxied) to another system.

===Disadvantages of Xinetd Super Server===
* The configuration file, /etc/xinetd.conf, is incompatible with the older /etc/inetd.conf.
** You can however use a conversion utility, xtoa, that is included with the distribution.
* Time-outs and other problems occur for RPC services, especially on busy systems.
** However, xinetd and portmap may coexist, allowing RPC through the portmapper.
* The configuration file for xinetd is /etc/xinetd.conf, but the default file only contains a few defaults and an instruction to include the /etc/xinetd.d directory.
* To enable or disable a xinetd service, edit its configuration file in the /etc/xinetd.d directory.
* If the disable attribute is set to yes, the service is disabled. If the disable attribute is set to no, the service is enabled.
* If you edit any of the xinetd configuration files or change its enabled status using Serviceconf, ntsysv, or chkconfig, you must restart xinetd with the command service xinetd restart before the changes will take effect.

===Configuration===
The xinetd configuration file is, by default, /etc/xinetd.conf. Its syntax is quite different than, and incompatible with, /etc/inetd.conf. Essentially, it combines the functionality of /etc/inetd.conf, /etc/hosts.allow and /etc/hosts.deny into one file. Each entry in /etc/xinetd.conf is of the form:

 <nowiki>
service <service-name>
{
    <attribute> <operator> <value> <value>
}</nowiki>

Note that spacing and indentation has been known to matter on some systems

{| class="wikitable"
! Attribute
! Use
|-
| <service-name> || Arbitrary name for the service, though typically the name of the standard network service being configured. Additional and completely nonstandard services may be added, as long as they are invoked through a network request, including network requests from the localhost itself.
|-
| <attribute> || There are a number of attributes which available which are listed in the next section
|-
| <value> || The value assigned to the specified attribute
|-
| <operator> || Valid operators include : =, += and -= . Most attributes will assign values using the = operator, but some attributes allow the adding and removing of the values assigned to them thus using the += and -+ operators
|}

Note: If you are transfering a service from one that is usualy managed by itself to one being managed by Xinetd Super Server, you do not need to turn on the service. Xinetd will manage the starting and stopping of the service all within itself. When setting up the first time, setup the service in Xinetd in the following order:
#Shutdown the service on the host system
#Configure the service in the Xinetd config file
#Restart Xinetd service. DO NOT TOUCH THE OLD SERVICE
#Check the status of the Xinetd service (Fedora: systemctl status xinetd) for any errors in the logs. If there are erros about port binding failures, it is likely the service is still running on its own or has not released the port yet upon it being shutdown

===Attributes & Values===

====socket_type====
The type of TCP/IP socket used. Acceptable values are stream (TCP), dgram (UDP), raw, and seqpacket (reliable, sequential datagrams).

====protocol====
Specifies the protocol used by the service. Must be an entry in /etc/protocols. If not specified, the default protocol for the service is used.

====server====
Daemon to invoke. Must be absolutely qualified.

====server-args====
Specifies the flags to be passed to the daemon.

====port====
Port number associated with the service. If listed in /etc/services, it must match.

====wait====
There are two possible values for this attribute. If yes, then xinetd will start the requested daemon and cease to handle requests for this service until the daemon terminates. This is a single-threaded service. If no, then xinetd will start a daemon for each request, regardless of the state of previously started daemons. This is a multithreaded service.

====user====
Sets the UID for the daemon. This attribute is ineffective if the effective UID of xinetd is not 0.

====nice====
Specifies the nice value for the daemon.

====id====
Used to uniquely identify a service when redundancy exists. For example, echo provides both dgram and streams services. Setting id=echo_dgram and id=echo_streams would uniquely identify the dgram
and streams services, respectively. If not specified, id assumes the value specified by the service keyword.

====access_times====
Sets the time intervals for when the service is available. Format is hh:mmhh:mm; for example, 08:00-18:00 means the service is available from 8 A.M. through 6 P.m.

====only_from====
Space-separated list of allowed clients. The syntax for clients is as follows:

{| class="wikitable"
! Value
! Description
|-
| hostname || A resolvable hostname. All IP addresses associated with the hostname will be used.
|-
| IPadress || The standard IP address in dot decimal form.
|}

====net_name====
A network name from /etc/networks.


=====x.x.x.0, x.x.0.0, x.0.0.0, 0.0.0.0=====
The 0 is treated as a wildcard. For example, an entry like 88.3.92.0 would match all addresses beginning with 88.3.92.0 through and including 88.3.92.255. The 0.0.0.0 entry matches all addresses. 

=====x.x.x. {a, b, ....}, x.x.{a, b, ...}, x. {a, b, ...}=====
Specifies lists of hosts. For example, 172.19.32.{1, 56, 59} means the list of IP addresses, 172.19.32.1, 172.19.32.56, and 172.19.32.59.

=====IPaddress/netmask=====
Defines the network or subnet to match. For example, 172.19.16.0/20 matches all addresses in the range 172.19.16.0 through and including 172.19.31.255. If this attribute is specified without a value, it acts to deny access to the service..

====no_access====
Space separated list of denied clients. The syntax for clients is given previously. Note that placement of no_access matters in the stanza as placing it above a setting allowing users in will block it. The stanzas on xinetd are read in procedurally from top to bottom

====redirect====
This attribute assumes the syntax, redirect= IPaddress port. It has the effect of redirecting a TCP service to another system. The server attribute is ignored if this attribute is used.

====bind====
Space separated list of denied clients. The syntax for clients is given previously. Binds a service to a specific interface. Syntax is bind = IPaddress. This allows hosts with multiple interfaces (physical or logical), for example, to permit specific services (or ports) on one interface but not the other.

====log_on_success====
Specifies the information to be logged on success. Possible values are:
{| class="wikitable"
! Value
! Use
|-
| PID || PID of the daemon. If a new daemon is not forked, PID is set to 0.
|-
| HOST || Client host IP address.
|-
| USERID || Captures UID of the client user through an RFC1413 call. Available only for multithreaded, streams services.
|-
| EXIT || Logs daemon termination and status.
|-
| DURATION || Logs duration of session. By default, nothing is logged. This attribute supports all operators.
|}

====log_on_failure====
Specifies the information to be logged on failure. A message indicating the nature of the error is always logged. Possible values are:
{| class="wikitable"
! Value
! Use
|-
| ATTEMPT || Records a failed attempt. All other values imply this one.
|-
| HOST || Client host IP address.
|-
| USERID || Captures UID of the client user through an RFC1413 call. Only available for multithreaded, streams services.
|-
| RECORD || Records additional client information such as local user,remote user, and terminal type.
|}
By default, nothing is logged. This attribute supports all operators.


====disabled====
It has the same effect as commenting out the service entry in the /etc/xinetd.conf file. Syntax is disabled = yes. If left out, the service is enabled.

==Examples==

===Basic Examples===
Consider a simple example. The attributes (everything inside the braces and to the
left of the = symbol) are very straightforward in their meaning as are the associated
values (everything inside the braces and to the right of the = symbol):
 <nowiki>
service ftp
{
    socket_type = stream
    protocol = tcp
    wait = no
    user = root
    server =/usr/sbin/in.ftpd
    server-args = -l -a
}</nowiki>

 <nowiki>
service telnet
{
    socket_type = stream
    protocol = tcp
    wait = no
    user = root
    server = /usr/sbin/in.telnetd
}</nowiki>

===Access Control Examples===

 <nowiki>
defaults
{
    log_type = SYSLOG local4.info
    log_on_success = PID HOST EXIT DURATION
    log_on_failure = HOST
    instances = 8
}

service login
{
    socket_type = stream
    protocol = tcp
    wait = no
    user = root
    flags = REUSE
    only_from = 142.0.0.0
    no_access = 142.100.0.0
    log_on success += USERID
    log_on failure += USERID
    server /usr/sbin/in.ftpd
    server_args -l -a
}</nowiki>

The configuration specifies a login service entry for a server (milliways for example)
that allows access from any system whose IP address begins with 142 except for
those whose address begins with 142.100.

Note here also <code>no_access</code> is blow <code>only_from</code>. Because this configuration file is read from top to bottom. If <code>no_access</code> appears first, everyone will be blocked out and not exception will be made to allow users in the <code>only_from</code> attribute.

===Using bind Examples===
The bind attribute allows for associating a particular service with a specific interface's IP address. Suppose we have an internal ftp server that supplies read-only resources for company employees via anonymous ftp. This ftp server has two interfaces, one that attaches to the corporate environment and the other that connects to a private internal network that is generally accessible only to employees who work in that particular group. We can implement the desired functionality using the following entries in /etc/xinetd.conf:

 <nowiki>
defaults
{
    log_type = SYSLOG local4.info
    log_on_success = PID HOST EXIT DURATION
    log_on_failure = HOST
    instances = 8
}

service ftp
{
    id = ftp
    socket_type = stream
    protocol = tcp
    wait = no
    user = root
    only_from 172.17.0.0 172.19.0.0/20
    bind = 172.17.1.1 #widget
    log_on_success += USERID
    log_on_failure += USERID
    server = /usr/sbin/in.ftpd
    server_args = -l -a
}

service ftp
{
    id = ftp_chroot
    socket_type = stream
    protocol = tcp
    wait = no
    user = root
    bind = 24.170.1.218 #widget server
    log_on_success += USERID
    log_on_failure += USERID
    access_times = 8:30-1:30 13:00-18:00
    server /usr/sbin/anon/in.aftpd
}</nowiki>
Each of the two ftp service entries has a unique id attribute. There is no limit to the number of services with the same name as long as each has a unique identifier. In this case, we set the id attribute to ftp for the internal ftp server and ftp-chroot for the external anonymous server. Note that in the latter case the daemon invoked is /usr/sbin/anon/in.aftpd (this is
the effective equivalent to twist for TCP-wrappers), which is different than the former service. The use of bind in each case will allow packets destined only to that interface to
invoke the indicated daemon. Thus, we can reach the anonymous server by executing ftp widget server. Since there are no access controls for that service, everyone has access. However, access is granted only between 8:30 A.M. through 11:30 A.M. and I P.M. through 6 P.m. due to the access_times attribute. On the other hand, executing ftp widget will be successful only if the first two octets of the client IP address begin with 172.17 or the request comes from an address in the range 172.19.0.0 through 172.19.15.255 because of the only_from attribute.

===Using redirect Examples===

The redirect attribute provides a method for proxying a service through a server. In other words, the user may telnet to a particular server running xinetd, and that server would open another connection to a different system. This can implemented this with the following entries in an /etc/xinetd.conf file:

 <nowiki>
service telnet
{
    socket_type = stream
    wait = no
    flags = REUSE
    user = root
    bind = 172.17.33.111
    log_on_success = PID HOST EXIT DURATION USERID
    log_on_failure = RECORD HOST
}

service telnet
{
    socet_type = stream
    wait = no
    flags = REUSE
    user = root
    bind = 201.171.99.99
    redirect = 172.17.1.1 23
    log_on_success = PID HOST EXIT DURATION USERID
    log_on_failure = RECORD HOST
}</nowiki>

The REUSE attribute is essential since the server will be terminating and restarting continuously. The bind attribute has the following effect. If the command telnet 172.17.33.111 is used, the connection will be made to the server itself. If, on the other hand, the command telnet 201.171.99.99 is used, then the connection will be forwarded to 172.17.1.1 on port 23 (the telnet port).

==Incorporating hosts.deny and hosts.allow into Xinetd Super Server==
Including TCP_wrappers functionality in /etc/xinetd.conf is very simple. Wherever /usr/sbin/tcpd is set as the value for the attribute server, that service will
be wrapped. The following examples illustrate this:
 <nowiki>
defaults
{
    log_type = SYSLOG local4.info
    log_on_success = PID HOST EXIT DURATION
    log_on_failure = HOST
    instances = 8
}

service ftp
{
    id = ftp_chroot
    socket_type = stream
    protocol = tcp
    wait = no
    user = root
    only_from = 172.17.0.0
    log_on_success += USERID
    log_on_failure += USERID
    access_times = 8:30-6:30
    server = /usr/sbin/tcpd
    server_args = /usr/sbin/in.ftpd -l -a
}

service telnet
{
    socket_type = stream
    wait = no
    flags = NAMEINARGS REUSE
    user = root
    bind = 172.17.33.111
    server = /usr/sbin/tcpd
    server_args = /usr/sbin/in.telnetd
    log_on_success = PID HOST EXIT DURATION USERID
    log_on_failure = RECORD HOST
}</nowiki>

The xinetd hosts access control differs from the method used by TCP wrappers. While TCP wrappers places all of the access configuration within two files, /etc/hosts.allow and /etc/hosts.deny, each service's file in /etc/xinetd.d can contain its own access control rules.

The following hosts access options are supported by xinetd:
* only_from — Allows only the specified hosts to use the service.
* no_access — Blocks listed hosts from using the service.
* access_times — Specifies the time range when a particular service may be used. The time range must be stated in 24-hour format notation, HH:MM-HH:MM.

The only_from and no_access options can use a list of IP addresses or host names, or can specify an entire network. Like TCP wrappers, combining xinetd access control with the enhanced logging configuration can enhance security by blocking requests from banned hosts while verbosely record each connection attempt.

For example, the following /etc/xinetd.d/telnet file can be used to block Telnet access from a particular network group and restrict the overall time range that even allowed users can log in:

 <nowiki>
service telnet
{
    disable = no
    flags = REUSE
    socket_type = stream
    wait = no
    user = root
    server = /usr/sbin/in.telnetd
    log_on_failure += USERID
    no_access = 10.0.1.0/24
    log_on_success = PID HOST EXIT
    access_time = 09:45-16:15
}</nowiki>
In the above example, when a client system from the 10.0.1.0/24 network, such as 10.0.1.2, tries access the Telnet service, it will receive the following message:
 <nowiki>
Connection closed by foreign host.</nowiki>
In addition, their login attempt is logged in /var/log/secure as follows:
 <nowiki>
May 15 17:38:49 boo xinetd[16252]: START: telnet pid=16256
from=10.0.1.2
May 15 17:38:49 boo xinetd[16256]: FAIL: telnet address from=10.0.1.2
May 15 17:38:49 boo xinetd[16252]: EXIT: telnet status=0 pid=16256</nowiki>

==Creating a 'defaults' Entry==
The purpose of the defaults entry in the /etc/xinetd.conf file is to specify default values for all services in the file. These default values may be overridden or modified by each individual service entry. The following is an example of defaults entry as it might appear in /etc/xinetd.conf.

 <nowiki>
defaults
{
    log_type = SYSLOG local4 info
    log_on_success = PID HOST EXIT DURATION
    log_on_failure = HOST
    instances = 8
    disabled = in.tftdp in.rexecd
}</nowiki>

The above defaults specify that for all services, log messages will be sent to the syslogd daemon via the local4. info selector. Successful connections to services will cause the PID, client IP address, termination status, and time of connection to be logged. Unsuccessful connection attempts will have the client IP address logged. The maximum number of instances for any one service is set to eight. Two services, in.tftpd and in.rexecd are disabled. The Red Hat distribution puts a link in the xinetd.conf file to a directory where all the individual service files are configured and kept

If the xinetd.conf file's default section is configured with an <code> includeidr</code> tag then individual service files will be configured within those directories. Example:
 <nowiki>
defaults
{
    instances = 60
    log_type = SYSLOG authpriv
    log_on_success = HOST PID
    log_on_failure = HOST RECORD
}
includedir /etc/xinetd.d</nowiki>
This example would store indicidual configuration files for each service in <code>/etc/xinetd.d</code>. Note that configurations in these files will override the defaults if they differ

==Notes==

==Sources==
