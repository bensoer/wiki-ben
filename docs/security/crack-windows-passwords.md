# Crack Windows Passwords

There is no such thing as forgetting your Windows Password or being locked out of your computer...

## Pre-Requisites
* Create a Kali Linux Bootable USB

## Process
1. plug in USB. Boot into Kali
2. mount partition
3. `cd` to `/mnt/partition`
4. `cd` to `/windows/System32/config`
5. Run `chntpw -l SAM*` to list all usernames on Windows
6. Run `chntpw -u “username” SAM` to view all options available to do with user's account.
7. Choose 1 to remove password in prompt when executing previous steps command