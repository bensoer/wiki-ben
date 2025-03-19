The open-source OS.

==Linux Monitoring Tools==
See [[Linux_Monitoring_Tools]]

== SELinux ==
See [[SELinux]]

== RSysLog ==
See [[RSysLog]]

== User Management ==
See [[Linux_User_Management]].

== Package Installer == 
See [[apt-get]].

== Favourite Commands ==

=== <code>find</code> and <code>grep</code> - search and sort for a string ===
*will recursively search <tt>/</tt> for <tt>''string''</tt> printing the results to the terminal; the <tt>-i</tt> flag will ignore case sensitivity
:<code>sudo find / | grep -i 'string'</code>

=== <code>tail</code> - live view of file ===
*will give you a scrolling view of the logfile; as new lines are added to the file, they will show up in your console screen
:<code>tail -f 'path to file'</code>

=== <code>ls</code> - list directory contents ===
*long list format with owner and permission details
:<code>ls -l</code>
*show hidden folders and files as well
:<code>ls -al</code>

=== <code>ps</code> - report a snapshot of the current processes ===
*for current user only
:<code>ps -u</code>
*for all users
:<code>ps -au</code>
*for all processes even those not associated with a terminal
:<code>ps -aux</code>

See Also [[Linux_Monitoring_Tools]] for details and analysis on using ps

=== <code>rm</code> - remove files or directories ===
*for a single file
:<code>rm "''file''"</code>
*removes directory and all contents <span style="color:red">'''USE CAUTION'''</span>
:<code>rm -r "''directory''"</code>
*forces the operation to delete unwriteable files <span style="color:red">'''USE EXTREME CAUTION'''</span>
:<code>rm -rf "''directory''"</code>

=== <code>chown</code> - change the user and/or group ownership ===
*change the file owner
:<code>chown "''Owner''" "''file''"</code>
*change the file owner and group
:<code>chown "''Owner''":"''Group''" "''file''"</code>
*change a folder and contents  <span style="color:red">'''USE CAUTION'''</span>
:<code>chown -R "''Owner''":"''Group''" "''directory''"</code>

=== <code>chmod</code> - change access permissions ===
*readable, writable, and executable by owner, group, and world
:<code>chmod 777 "''file''"</code>
*make a file executable
:<code>chmod +x "''file''"</code>
*change a folder and recursively apply to subfolders and subfiles <span style="color:red">'''USE CAUTION'''</span>
:<code>chmod -R 744 "''directory''"</code>

==How to Determine chmod Code==
chmod is a tricky function in it allows you to alter file and folder permissions based on numbers, but these numbers are actually not just nonsense and are based on a pattern that can be easily recognized to produce all file permissions desired by the implementer.

chmod uses 3 numbers to set its permissions (eg. 777). Each number is an octal value that maps  to read,write and execute permissions for 3 user groups. Each of the 3 numbers is the permission for each group.

* The first number sets permissions for the owner/creator of the file or folder. This may be root, but if user implemented is typically that user.<br>
* The second number sets permissions for the group that the file or folder is part of. For example files created in the /etc/apache or /var/www folders are typicaly part of the www-data or apache usergroup
* The third number sets permissions for all other users. These can be anyone who is not the owner or part of the group that the folder or file belongs to. Typicaly for most user created files this is the field you will be editing because usually it is for other programs needing access to the file you have created that is not part of any groups

So each number then is an octal number (0-7) but this can be represented in binary needing 3 bits (0-7 in base10 = 000-111 in base2). These 3 bits that make up the number for the group then determine he permission of the group. Each bit represents the following permission: 
* First bit is Read permission. 1 means it is enabled 0 means disabled
* Second bit is Write permission. 1 means it is enabled 0 means disabled
* Third bit is Execute permission. 1 means it is enabled 0 means disabled

===Examples===
Lets give a file  Read/Write/Execute to itself, Read/Execute to the group and Read to all others.

* Read/Write/Execute = 111 = 7. First number is 7
* Read/Execute for the group = 101 = 5. Second number is 5
* Read for all others = 100 = 4. Third number is 4

Therefor to set this permission we need <code>chmod 754 <name of file></code>
