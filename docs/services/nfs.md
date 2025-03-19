NFS is essentially the predecessor of SAMBA. It is strictly limited to UNIX and LINUX systems, and has some major security flaws. It is still popularity used though due to its integration with mount and ability to treat network drives like local mounted drives. Also its simplistic yet powerful configurations make it a much simpler alternative to SAMBA

==Install NFS==
===Fedora/CentOS===
dnf install nfs-utils

Note: That this is to install nfs on the server whose folders will be shared after setup

==Setup NFS==
BEFORE DOING ANYTHING, after installing NFS create an exports file under <code>/etc</code> so as to not to accidentaly give full access to the file system.
 <nowiki>
sudo touch /etc/exports</nowiki>
The exports file is where we place our configuration information for nfs

On each line enter
 <nowiki>
<directory-to-share> <ip/ip-range with access><permissions></nowiki>
An Example:
 <nowiki>
/home/shareFolder 192.168.0.0/24(rw,no_root_squash)</nowiki>

Note you can specify an explicit address as the second parameter, or specify a range using decimal subnet mask notation (ip number \ valid bits). The above written example would give anybody with a prefix of 192.168.0 access to the /home/shareFolder and is allows read/write access and if accessed with root, the user permissions of the client will not be squashed while in the drive

See permissions chart for common permission options and settings. Man is also your friend here

==Permissions==
Referre to man documents for all options available
{| class="wikitable"
!Permission
!Value
|-
|ro||Give Read-Only Access
|-
|rw||Give Read/Write Access
|-
|no_root_squash|| Allows the root user on the client to have root access on the share folder mounted. By default this is disabled and the client user has rights equal to the nobody user
|-
|no_subtree_check|| NFS will validate that the requested directory the client is within the correct directory. This is useful if only part of the volume is shared, If all of the volume is shared this will slow down transfers
|-
|sync|| This effects the exports command. Changes to run synchronizations synchronously vs. the default asynchronous. Warning though that this may cause data corruption if the server reboots or turns off
|}

==Start NFS Server==
===Fedora/CentOS===
Enabled the nfs service with:
 <nowiki>
systemctl enable nfs-server.service</nowiki>
Then start the nfs server:
 <nowiki>
systemctl start nfs-server</nowiki>
You can also then stop the server at any time with
 <nowiki>
systemctl stop nfs-server</nowiki>

Note that there is a bug in nfs with the restart command as it does not always fully restart nfs. It is in this case best practice to stop and then start the server instead of restarting

Alternatively you can also use this command while the server is running to update your changes...but I have never had it work correctly. To gaurantee the NFS server to update with new changes. stop and start the server
 <nowiki>
exportsfs -v</nowiki>

==Connecting to an NFS Share==
Using NFS as a client is very easy, just use the mount command as if you were mounting a local drive
 <nowiki>
mount -t nfs <ip-of-server>:<dir-of-share> <local-system-dir></nowiki>

The <code>ip-of-server</code> is the IP of the server with nfs installed and sharing files. <code>dir-of-share</code>is the directory on the server that is being shared. This is the exact same dir as specified in the <code>/etc/exports</code> file configured on the server side. The <code>local-system-dir</code> is then the location on your local computer you would like to mount the location. This will be the directory you will access the share folders on your computer. A common location is <code>/mnt</code>

An Example:
 <nowiki>
mount -t nfs 192.168.0.123:/home/sharefolder /mnt/shareserver</nowiki>

==Notes==

==Sources==
