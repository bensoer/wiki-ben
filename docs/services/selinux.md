# SELinux

SELinux is a firewall program on Fedora and a number of REHL linux distributions. It can be a bit of a pain with permissions and giving programs on your local machine external access. Here are some commands to help you out

==Directories==
SELinux Root Directory: <code> /etc/selinux </code>

==Temporary Enable/Disable SELinux==
Good for Testing is SELinux is the problem

 <nowiki>
sudo setenforce 0 #puts SELinux in permissive mode. It prompts issues but does not stop them
sudo setenforce 1 #puts SELinux in enforce mode. Will prompt and will block</nowiki>

You can always check what the status of SELinux is with this command:
 <nowiki>
sudo sestatus</nowiki>

==SELinux Is Interfering with Apache and PHP==
This is a common issue with the Laraval framework

To disable SELinux from interfering in a directory use the following command:

 <nowiki>
su -c "chcon -R -h -t httpd_sys_script_rw_t <dir-to-folder-to-stay-out-of>"</nowiki>

Note that it will do this recursively, so any files and folders below the passed in folder will be ignored aswell. Should note aswell that this should only be used on local development environments. Production configuration please do more googling

==Notes==

==Sources==
http://stackoverflow.com/questions/17954625/services-json-failed-to-open-stream-permission-denied-in-laravel-4/27377624#27377624
