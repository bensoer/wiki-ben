==What is SFTP==
SFTP is the SSH File Transfer Protocol which is an extension of the Secure Shell protocol (SSH) to provide secure file transfer capabilities. It is not to be mistaken for FTP or FTPS. Unlike FTP, SFTP encrypts both commands and data, preventing passwords and sensitive information from being transmitted openly over the network. It cannot interoperate with FTP software. FTPS is an extension to the FTP standard that allows clients to request FTP sessions to be encrypted. This is done by sending the "AUTH TLS" command.

==Setting Up a Simple Server==
The goal is to setup a SFTP server where the users are chrooted to their home directory, and have limited system powers.
===Setting up the Server===
Assuming you already have SSH installed, we need to edit the SSH server config file.
#Open the config file<br> <code>sudo nano /etc/shh/sshd_config</code>
#Comment out the following line with a <tt>#</tt> at the beginning<br> <code>#Subsystem sftp /usr/lib/openssh/sftp-server</code>
#Add the following at the end of the file:<br> <pre>Subsystem sftp internal-sftp&#10;&#10;Match Group sftpusers&#10;    ChrootDirectory %h&#10;    ForceCommand internal-sftp&#10;    X11Forwarding no&#10;    AllowTCPForwarding no&#10;    PasswordAuthentication yes</pre>
#Restart SSH<br> <code>sudo service ssh restart</code>

===Creating the sftpusers Group===
<code>sudo groupadd sftpusers</code>

===Create SFTP Users===
#Create user<br> <code>sudo adduser username</code>
#Prevent SSH login & assign user to SFTP group<br> <code>sudo usermod -G sftpusers username</code><br><code>sudo usermod -s /usr/sbin/nologin username</code>
#Chroot user (limit them to their home directory)<br> <code>sudo chown root:root /home/username</code><br><code>sudo chmod 755 /home/username</code>
#Give the user a folder to upload to<br> <code>sudo mkdir /home/username/share</code><br><code>sudo chown username:sftpusers /home/username/share</code>

==Notes==

==Sources==
