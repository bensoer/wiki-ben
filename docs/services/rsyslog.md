rsyslog is the linux logging tool for all system activity. There are a number of useful things you can do with it to evaluate and debug problems with your server with the logs

==Directories==
* Main system logs are located: <code>/var/log/messages</code>  This is a catchall log for any logging on the system
* rsyslog configuration file: <code>/etc/rsyslog.conf</code>
* rsyslog daemon: <code>/sbin/rsyslogd</code>

* Root logins, user logins, su attmpts are located: <code>/var/log/secure</code>
* Mail traffic is logged to: <code>/var/log/maillog</code>

* Error Message from uuvp and news server (innd) daemons: <code>/var/log/spooler</code>  (This one is not used by most systems anymore)

Some versions of Fedora don't have any of these log files aswell and instead use the <code>journalctl</code> command to access and filter through the logs. rsyslog though can be installed aswell to work side by side with journalctl and will not interfere

==Configure Remote Logging==
Remote logging is an important security functionality so that hackers can not alter the log files and remove their presence on the machine. By sending copies of your logs to a machine completely dedicated and locked down to only archive logs will ensure that logs will not be tampered with

You can configure a system to send its logs to a remote server by editing the <code>/etc/syslog.conf</code> file. Add an entry to the bottom of the conf file that looks something like this
 <nowiki>
<facility>.<priority> @<ip-of-log-server>:514</nowiki>

Facility and prioiryt are used to specify and filter what log information is sent to the log server. See the man pages for full documentation on the different facilites and priorities are available to filter the send logs. Note that priority is used as a threshhold and logs from that threshold and up are sent to the server.

To send all logs from each facility and each priority level you can just use <code>*.*</code>

Example:
 <nowiki>
*.* @144.272.38.38:514</nowiki>
Note also the importance of port 514. This is the typical default port number most loggings servers are configured to use. You can though of course configure this on your logging server.

==Configure Logging Server==
With rsyslog you can easily configure your logging server to recieve external log files with a few commands to rsyslog on startup

# Open <code>/etc/rsyslog.conf</code>
# Uncomment or add the following lines:
 <nowiki>
$ModLoad imudp
$UDPServerRun 514

$ModLoad imtcp
$InputTCPServerRun 514</nowiki>
If you know what transportation protocol your server uses you only need to add/uncomment the corresponding lines.

If you wanted to change the port number to receive log files, change the value assigned to the <code>$UDPServerRun</code> variable

For the changes to take effect, restart rsyslog
 <nowiki>
sudo systemctl restart rsyslog #Fedora/CentOS
sudo service rsyslog restart #Ubunt</nowiki>

==Configure RSysLog To Use A Different Conf Directory==
RSysLog can be configured to load its configuration details form a different directory. Unfortunately this implementation so far has to be done manually and does not persist past a restart.

1) copy the rsyslog.conf file (by default in /etc/rsyslog.conf) to the new location you would like it to be referenced by<br>
2) Shutdown the currently running rsyslog service with the following command
 <nowiki>
sudo service rsyslog stop</nowiki>
3) Startup the rsyslog daemon manualy with the following command
 <nowiki>
rsyslogd -n -f /new/conf/directory/rsyslog.conf & </nowiki>
4)Press Enter Twice. Once Loads the program, Second releases terminal from the process
Note the '&' at the end is important to ensure the daemon executes in its own process

This setup may appear a bit obvious as someone may login and find rsyslog turned off, attempt to turn it on, and recieve errors. To resolve this issue execute step 3 with the following command instead
 <nowiki>
rsyslogd -n -i 1447 -f /new/conf/directory/rsyslog.conf & </nowiki>
By default rsyslogd uses PID 1446 to loadup. If you are to run multiple instances of rsyslog on your system, you will need to provide each new instance with their own PID. This can be done simply with the -i parameter when loading rsyslogd

==Facilities==
{| class="wikitable"
! Facility
! Message Category
|-
| auth or security || Security / Authorization
|-
| authpriv || Private Security / Authorization
|-
| cron || Cron Daemon Messages
|-
| daemon || System Daemon-Generated Messages
|-
| ftp || FTP Server Messages
|-
| kern || Kernel Messages
|-
| lpr || Printer SubSystem
|-
| news || Network News Subsystem
|-
| syslog || Syslog-Generated Messages
|-
| user || User Program-Generated Messages
|-
| UUCP || UUCP SubSystem
|-
| mail || Mail SybSystem
|}
==Priorities==
{| class="wikitable"
! Priority
! Message type
|-
| debug || Debug messages
|-
| info || Informational status messages
|-
| notice || Normal but important conditions
|-
| warning or warn || Warning Messages
|-
| err or error || Error Messages
|-
| crit || Critical Conditions
|-
| alert || Immediate Attention Required
|-
| emerg or panic || System is unusable
|}

==Configure Remote Logging to A Database==
RSysLog can log to a database. Documentation online is poor and there are a number of careful steps that must be met. But the procedure is possible. This procedure will install and configure remote logging to a MySQL database

===Configure MySQL Database===
# Find the version of your RSysLog on your Logging Server. This can be done by restarting RSyslog and then checking the /var/log/syslog log file. When RSysLog initializes it first prints its version
# Then go to the RSysLog website and search through the archives for a Download of the same version as your RSysLog server. Download it onto the server where you MySQL database is hosted
# Extract the folder and get the 'createDB.sql' script roughly located in /rsyslog-8.4.2/plugins/ommysql/createDB.sql. Run this script on MySQL (can either from console or by import in MySQL Toolbox). This script typicaly includes code to build the database, so nothing needs to exist before running it. Make sure to create a user account that can access the database.

===Configure Logging Server===

# apt-get install rsyslog-mysql on the Server that will be logging to your MySQL database. Skip any prompted automated setup procedure.
# cd to <tt>/etc/rsyslog.d/</tt> within this folder during the install will be a 'mysql.conf' file. Open it with nano
# Update or copy into the file the following information, substituting your own information where necessary
 <nowiki>
$ModLoad ommysql
*.* :ommysql:<host>,<database>,<username>,<password> </nowiki>
For my setup this looks something like this. Note that the host could either be localhost, in which case the logging server also is hosting the MySQL database, or it can be a domain or IP of another system hosting the MySQL database. RSysLog will resolve hostnames so a domain is acceptable aswell
 <nowiki>
$ModLoad ommysql
*.* :ommysql:192.168.20.100,Syslog,username,supersecret </nowiki>
<ol start="7">
<li>Once configured restart rsyslog on the logging server with service <tt>rsyslog restart -r</tt></li>
<li>Login to your MySQL server and you should see events now arriving</li>
</ol>

==Raspberry Pi Example==
My router has the ability to log remotely to a Syslog server on its network so I decided to set one up on my Pi. It took a total of 15 minutes.
<ol>
<li>Edit <code>/etc/rsyslog.conf</code> to listen for a connection on the default port of 514 (see above)</li>
<li>Create your log file:</li>
<code>sudo touch /var/log/router.log</code>
<li>Configure rsyslog to use the log file you jut created</li>
*Under <code>/etc/rsyslog.d/</code> create a file with the extension <code>.conf</code>
:<code>sudo nano /etc/rsyslog.d/router.conf</code>
*Add the following lines:
 <nowiki>
$template NetworkLog, "/var/log/router.log"
:fromhost-ip, isequal, "192.168.0.1" -?NetworkLog
& ~</nowiki>
<li>Restart rsyslog (see above)</li>
<li>Enable remote logging on your web interface of your router</li>
<li>Configure logrotate</li>
It is a good idea to configure the new log file you just created in <code>logrotate</code> to compress and remove the log file when it gets too big.
*Create a file in <code>/etc/logrotate.d/</code>
:<code>sudo nano router</code>
*Add the following lines:
 <nowiki>
/var/log/router.log {
        rotate 7
        size 500k
        notifempty
        compress
        postrotate
                invoke-rc.d rsyslog rotate > /dev/null
        endscript
}</nowiki>
</ol>

==Notes==
If RSyslog is not recieivng logs try the following:
* On the recieving host check and entry for <code>udp/514</code> exists under <code>/etc/services</code>
* RSysLog may need to be started with the <code>-r</code> flag to allow remote logging. This may need to be done on the sender and/or reciever
** Example: <code>systemctl restart rsyslog -r</code>
* Clear the Firewall with:
*#<code>iptables -F</code>
*#<code>iptables -X</code>

==Sources==
https://linux.die.net/man/8/rsyslogd <br>
https://linux.die.net/man/5/rsyslog.conf <br>
https://linux.die.net/man/8/syslogd <br>
http://serverfault.com/questions/542379/how-to-change-rsyslog-configuration-file-directory<br>
http://opensourceforu.com/2015/10/remote-logging-using-rsyslog-and-mysql/<br>
http://www.rsyslog.com/doc/v8-stable/tutorials/recording_pri.html
