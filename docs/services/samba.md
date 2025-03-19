# SAMBA

==Install SAMBA==
dnf install samba
==Configure SAMBA==
For the most basic configuration and security setup, append the following to the bottom of the file
 <nowiki>
[NFSHARE]
	comment= <share-folder-comment>
	path = <dir-to-share>
	public = <yes|no>
	writable = <yes|no>
	guest ok = <yes|no>
	printable = <yes|no></nowiki>
<b>Comment</b> is just a comment that can be used to help identify the share. <br>
<b>Path</b> is the directory that is being shared through SAMBA <br>
<b>Public</b> is to enable public access to the share folder <br>
<b>Writable</b> is to enable writing to the share folder <br>
<b>Guest Ok</b> enabled anonymous access to the share folder <br>
<b>Printable</b> is whether SAMBA will share printer access through this share folder <br>

==Start SAMBA==
===Fedora/CentOS===
#Start the SMB service: <code>systemctl start smb.service</code>
#Then start the SAMBA server: <code>systemctl start smb</code>
* You can check if your mount has worked with the following command: <code>smbclient -L localhost</code>

Note that at this point no user accounts have been configured for SAMBA so the only authetnication available is anonymous access. When you run the above command it will prompt to enter a password. Do not enter a password and hit enter. This will attempt to then connect as if you are anonymous.

==User Permissions On SAMBA==
By default SAMBA is configured with 'user' security. For this report were going to configure further this default setup. To add security, were going to add a user and password. With how SAMBA is built, these credentials <b>must</b> match a user and password on the linux system.

Add a SAMBA user with this command:
 <nowiki>
smbpasswd -a <username-of-user-on-linux-system> </nowiki>
Press enter and you may be prompted to enter a password for the account. Add the same password as the account password is to login to the linux system. Note: If the user account has its password changed at a later time, this password will have to be updated for the account to work
To update a SAMABA user's pasword use this command:
 <nowiki>
smbpasswd <username-of-user-on-linux-system> </nowiki>

==SAMBA with Xinetd==
You can configure Xinetd to manage SAMBA aswell for additional security.

The following entries in the <code>/etc/xinetd.conf</code> will add it:

 <nowiki>
server netbios-ns
{
    socket_type = dgram
    protocol = udp
    wait = yes
    user = root
    only_from = 192.168.0.1 192.168.0.2
    group = root
    server = /urs/local/sama/bin/smbd
}

server netbios-ssn
{
    socket_type = stream
    protocol = tcp
    wait = no
    user = root
    only_from = 192.168.0.1 192.168.0.2
    group = root
    server = /usr/local/samba/bin/smbd
}</nowiki>

To see details on configuring Xinetd see the appropriate page

==Notes==

==Sources==
