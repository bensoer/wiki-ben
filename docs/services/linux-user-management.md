
==Summary==
Basic user creation and account manipulation is relatively easy with a number of simple commands


{| class="wikitable"
!Command
!Purpose
!Syntax Use
|-
|adduser||Creates a user account|| adduser <username>
|-
|passwd||Assigns a password to a user account || passwd <username>
|-
|userdel||Deletes user account. Does not delete content || userdel <username>
|-
|}

==File Locations==
Linux stores its user account and password hashes in two seperate files located on the file system.

The <code>/etc/passwd</code> file stores all of the user accounts on the system <br>
The <code>/etc/shadow</code> file stores all of the hashes of the passwords to each account on the system

==Commands==
===adduser===
<code>adduser</code> adds a user to the system. This user account though is unaccessible as no password has been assigned to it, but can be identified as a valid user through ssh login or virtual login when logging into linux in non-gui mode. Note this step will create the user accounts home directory under <code>/home/<username></code>

Create a user by entering:
 <nowiki>
adduser <username> </nowiki>

Creating a user who also has sudo capabilities can be done simply by entering:
 <nowiki>
adduser <username> sudo</nowiki>
Adding users to any group can be done in the same way. To remove a user from a group, see userdel

===passwd===
After adding a user, you should immediately add a password, thus giving access to the account

Add a user by entering:
 <nowiki>
passwd <username></nowiki>

You will then be prompted to enter a password and possibly warned if it is not strong enough, you can choose to ignore this warning if you would like

===userdel===
<code>userdel</code> will delete the user's account but does not delete any of the user's data or content stored in their home folder. This function acts as a method of disabling the user account. 

To disable/delete a user account enter:
 <nowiki>
userdel <username></nowiki>
Sometimes if the user is currently active or a program is using that user, userdel will prompt an error from the disable/delete. You can confirm and check this by running this command
 <nowiki>
w <username></nowiki>
This will list all processes that are being used by the user or using the user for executing their commands. By either killing all processes using the user, or using -f flag to force the user to be deleted, userdel will then disable the account

To fully delete an account you will need to delete the user's data folder in <code>/home/<username</code>

This can be done cautiously with the following command
 <nowiki>
cd /home
rm -rf <username></nowiki>

Note we are using the remove function but also have navigated to the <code>/home</code> folder. All we are doing here is forcing and recursively deleting the user's data folder from the <code>/home</code> folder

Alternatively, userdel can also be used to remove users from groups. You can list all groups a user is in with the following command
 <nowiki>
groups <username></nowiki>
Simply, remove the user from a group with the following command
 <nowiki>
userdel <username> <usergroup></nowiki>

==Notes==

==Sources==
