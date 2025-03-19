# Server Hardening

Server hardening is the process of locking down a server so that the minimum amount of necessary components are allowed in and out of the server. This also includes applications operating within the server as well. In some cases, this procedure is simply creating a minimum version of the operating system and its tools for the servers specific purpose.

Below is a continually evolving outline on an ideal practice of hardening different operating systems and different software components within each operating system.

==Packages==
To harden the server, removing as many packages as possible ensures that no surprising software is operating on the system. Additionally this reduces the chances of an exploit being discovered
in a library you are not regularly keeping up to date. In some cases, installing a minimal OS build is easiest in this situation

===Debian===
 <nowiki>
sudo apt-get --purge remove python3 python perl ruby 
sudo apt-get install deborphan
sudo apt-get remove --purge `deborphan`
sudo apt-get install localepurge
sudo localepurge </nowiki>

====Debian SMTP====
Debian Also comes preconfigured with a lightweight SMTP daemon. It can be quite a pain to remove it. It can be uninstalled though with the following commands
 <nowiki>
apt-get remove exim4 exim4-base exim4-config exim4-daemon-light </nowiki>
You can then delete all logging belonging to the daemon with the following command
 <nowiki>
rm -r /var/log/exim4/ </nowiki>
This removal of course does not remove any configuration files. You can remove those by including the <tt>--purge</tt> flag in the remove call. See posting for details: http://stackoverflow.com/questions/12061358/how-to-cleanly-remove-exim4-mail-server-on-ubuntu

===Raspbian===
 <nowiki>
sudo apt-get remove --purge libx11-6 xserver-xorg xserver-xorg-core xserver-common luajit java-common oracle-java8-jdk gdb gdbserver python-pifacecommon python-pifacedigitalio python-numpy python3 
python3-minimal python3.2 python3.2-minimal debian-reference-common raspi-copies-and-fills hicolor-icon-theme raspberrypi-artwork omxplayer penguinspuzzle gnome-themes-standard-data lxde-icon-theme 
desktop-file-utils libfreetype6-dev libept-dev smbclient aptitude aptitude-common libsysfs2 libident libboost-iostreams1.46.1 libept1.4.12 libboost-iostreams1.50.0 libboost-iostreams1.48.0 
libboost-iostreams1.49.0 libjbig0 libcwidget3 libsigc++-1.2-5c2 libsigc++-2.0-0c2a libapt-pkg-dev libtagcoll2-dev libraspberrypi-dev libraspberrypi-doc gcc-4.5-base libxapian-dev libwibble-dev 
zlib1g-dev libxapian22 gstreamer1.0-omx fbset menu-xdg hardlink udisks python-rpi.gpio gstreamer1.0-alsa python-serial xdg-utils cgroup-bin v4l-utils dphys-swapfile pkg-config gstreamer1.0-libav 
wireless-tools cifs-utils ncdu

sudo apt-get autoremove --purge

sudo apt-get install deborphan
sudo apt-get remove --purge `deborphan`

sudo apt-get install localepurge
sudo localepurge </nowiki>

==Users==
User creation and credential management is important in securing your server. Always use long passwords for regular users, and even longer ones for the root user or sudo privileged users. An ideal
strategy is either use a password generator or create sentences instead of words for your passwords. Note that length is more important then unique characters. Adding unique characters (eg: @#$.?), numbers, lowercase and uppercase will reduce the chances of the password being available in a dictionary or hybrid password attack. Length will though increase time during a brute-force attack. In any situation you want attackers to resort to a brute force attack, this way being the least efficient and most time consuming manner. A nice overview of this can be found here: https://www.youtube.com/watch?v=RtUvMJFP_IE

On a fresh install
# Change the root password with <tt>sudo passwd root</tt>
# Create a new regular user with <tt> adduser <username></tt>. DO NOT give this user sudo privileges
# Remove sudo privileges from all users who can login to the machine

==OpenSSH==
* Configure the ssh daemon (typicaly in <tt>/etc/sshd/sshd_config</tt>) to not allow connections with root
* Configure RSA key based authentication. Once configured DISABLE PASSWORD AUTHENTICATION
* Configure Allow, AllowGroup, Deny, DenyGroup settings in ssh (http://www.cyberciti.biz/tips/openssh-deny-or-restrict-access-to-users-and-groups.html)

Scroll to the bottom of the sshd_config and update or add the following:
 <nowiki>
KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256

Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes256-ctr

MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-512 </nowiki>
This will give the strongest Key Exchange, Ciphers and MAC Algorithms

==IpTables/Firewall==
* See IpTables page for details - Create Firewall which only allows required services through
** DHCP
** DNS
** SSH only in if it is being used
* Ensure Kernel Forwarding (allows packets to be forwarded) is Disabled Unless this server is a Gateway Server
* Ensure IPTABLES default policy is DROP on INPUT, OUTPUT, and FORWARD
* Ensure IP6TABLES default policy is DROP on INPUT, OUTPUT, and FORWARD
** Unless you are explicitly supporting IPv6 for a specific application, there should be no ACCEPTing rules in this firewall
* Install and configure <tt>iptables-persistent</tt> to ensure firewall persists. See IpTables page again for details

==RSysLog==
Always Configure RSysLog to do remote logging!. At minimum use a cloud service such as Papertrail to store extra copies of system logging. Ideally have a separate hardened server for this purpose.

Configure as many components as possible (apache, lightppd, mysql) to log their data through RSysLog, thus allowing copies all to be sent to a remote logging service. Details on RSysLog can
be found on the RSysLog page

Tools such as Splunk, Sawmill, Loggly support mix of features to also retrieve RSyslogs and analyse them in real-time. These are also options that can be setup with a remote logging server

==Intrusion Detection / Intrusion Prevention Systems==
Depending on your configuration an IDS or IPS may be ideal to have within the network. Systems such as Snort, Bro, Suricata can listen and run analysis on various traffic running through your network or
system, providing useful insight along with RSysLog. See the Snort and Bro pages for documentation on those products

==Port Knocking==
Depending on configuration, port knocking is an ideal method of hiding open ports on the server and to make connections more discrete. Numerous tools exist, most notably doorman is a tool that supports this
functionality, and manipulates the IPtables rules to implement its functionality
==Notes==

==Sources==
Raspbian Minimal Install: https://www.raspberrypi.org/forums/viewtopic.php?t=109262<br>
Configure Accept and Deny for OpenSSH: http://www.cyberciti.biz/tips/openssh-deny-or-restrict-access-to-users-and-groups.html

===SSH===
https://security.stackexchange.com/questions/179114/what-are-the-toughest-ssh-daemon-settings-in-terms-of-encryption-handshake-or<br>
https://www.linuxjournal.com/content/cipher-security-how-harden-tls-and-ssh<br>
https://infosec.mozilla.org/guidelines/openssh<br>
https://stribika.github.io/2015/01/04/secure-secure-shell.html
