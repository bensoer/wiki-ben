# Crack Windows Passwords

There is no such thing as forgetting your Windows Password or being locked out of your computer...
==Pre-Requisites==
* Create a Kali Linux Bootable USB
==Process==
#plug in USB. Boot into Kali
#mount partition
#cd to /mnt/partition
#cd to /windows/System32/config
#Run <code>chntpw -l SAM*</code> to list all usernames on Windows
#Run <code>chntpw -u “username” SAM</code> to view all options available to do with user's account.
#Choose 1 to remove password in prompt when executing previous steps command

==Notes==

==Sources==
