# Cron

"cron" from the Greek word "chronos" meaning time.

cron is an automation tool that periodicaly wakes up and executes "cron jobs" that can be stored in specific locations to do specific tasks. cron jobs are generaly written in shell, but are free to execute any command thus being expandable aswell to execute any other scripting language script

==File Locations==
Main cron file (crontab): <code>/etc/crontab</code> <br>
Additional cron directory: <code>/etc/cron.d/</code> <br>


Default Cron Logs: <code>/var/log/cron</code> <br>
NOTE: On some Fedora 22 this log may not exist. If you can not find it, check if you have <code>syslog</code> installed. If that is not installed, then your logs are being stored in journalctl. Lookup journalctl for more documentation. Alternatively you can also install syslog along with journalctl and it will have no effect on logging


<b>These directories may or may not exists depending on distro</b> <br>
Hourly cron directory: <code>/etc/cron.hourly</code> <br>
Daily cron directory: <code>/etc/cron.daily</code> <br>
Weekly cron directory: <code>/etc/cron.weekly</code> <br>
Monthly cron directory: <code>/etc/cron.monthly</code> <br>

==Setup==
When the daeomon wakes up every minute it first checks and executes the crontab file, then it checks and executes all files in the cron.d folder. When executing these scripts, any output is mailed to the owner of the crontab or the user name in the MAILTO environment variable that can be set in the crontab - if it exists.

Fedora treats the files located in <code>/etc/cron.d/</code> as an extension of the <code>/etc/crontab</code> file and therefor inherits any settings as thier defaults from the crontab file. The intended purpose of this feature is to allow packages that require finer control of their scheduling then that available in <code>/etc/crontab</code>

For more general cron usage, some Unix/Linux distributions come with hourly, daily, weekly and monthy cron job folders. These folders contents are executed at such appropriate times. To use these prebuild cron jobs, add your shell script to the appropriate folder

You should note you can only use Bourne Shell Command for Cron. Also Cron requires absolute paths for all commands (eg. "/bin/ls"  to use the list command)

==Create A Cron Entry==
An example of an entry in the <code>/etc/crontab</code> file looks like this:
 <nowiki>
*  *  *  *  *  root /bin/full-path/script.sh</nowiki>
Note that SPACING is VERY IMPORTANT. There are 2 spaces between each star including between the last star and root, and then a single space between root and the script directory

A template of what each item stands for:
 <nowiki>
<minute>  <hour>  <dayofmonth>  <month>  <dayofweek>  <user-permission> <dir-to-script></nowiki>

Each of the stars represent a time entry point. A "*" by itself means "every" of that category. It's easiest to read this as a sentence to decipher what is happening. Ranges for each entry is as follows

{| class="wikitable"
! Entry
! Range
! Additional Functionality
|-
| <minute> || Value from 0-59. Entry means at what minute the cron job will run. || To have it run every increment of a certain amount of minutes use */<minutes-before-next-run>
|-
| <hour> || Value from 0-23. Entry means at what hour the cron job will run. || To have it run every increment of certain amount of hours use */<hours-before-next-run>
|-
| <dayofmonth> || Value from 1-31. Entry means at what day of the month to execute the cron job ||
|-
| <month> || Value from 1-12. Entry means at what month to execute the cron job. || To have it run every increment of a certain amount of months use */<month-before-next-run>
|-
| <dayofweek> || Value from 0-6. Sunday being 0. Entry means at what day of the week to execute the cron job ||
|-
| <user-permission> || What user to use when executing the cronjob. Typicaly this is root
|-
| <dir-to-script> || The absolute path to the shell script to be executed || By adding "run-parts" after the user-permission section this can be an absolute path to a directory containing multiple scripts which will all be executed based on this cron entry
|}

Whenever you make changes Cron does not need to be restarted. Everytime the crontab command executes it updates the modtime of the spool directory whenever it changes a crontab. By doing this the crontab checks if the spool directory's modtime (or modtime in /etc/crontab) has changed and if it has, cron will then examine the modtime on all crontabs and reload those which have changed.

==Examples==
===Crontab Examples===
 <nowiki>
01 * * * * root run-parts /etc/cron.hourly
02 4 * * * root run-parts /etc/cron.daily
22 4 * * 0 root run-parts /etc/cron.weekly
42 4 1 * * root run-parts /etc/cron.monthly</nowiki>
* The first line specifies that during the first minute, of every hour, of every day, of every week, and every month, all the scripts in the /etc/cron.hourly directory will be run.
* The second line specifies that all of the scripts in the /etc/cron.daily will be executed daily at 0402 hrs (using the 24 hour clock).
* The third line specifies that all of the scripts in the /etc/cron.weekly directory will be executed every week, on day 0 (Sundays - the days are numbered from 0 to 6, with 0 being Sunday) at 0422 hrs.
* The last line specifies that all of the scripts in the /etc/cron.monthly will be executed on the first of very month at 0442 hrs.

===Script Examples===
A script is an sh file that could contain something as simple as:
 <nowiki>
mail -c foo@domain.ca -s "attachment" bar@domain.ca < /root/file.txt</nowiki>
This is a script that sends an email to bar@domain.ca and CC's it to foo@domain.ca. The subject is "attachment" and the mail contents is injected via input redirection from a file stored in /root/file.txt


==Storing Crontab Output==
By defualt cron saves the output of a script in the user's mailbox (in most cases that is root)

For user and custom application scripts, it is much better to save in a seperate logfile. Alter your cron job to look something liek this to reroute its output:
 <nowiki>
*/10 * * * * /home/foo/ex1 2>&1 >> /var/log/script_output.log</nowiki>
What is happening here is both <code>STDOUT</code> and <code>STDERR</code> are being output redirected and appended to var/log/script_output.log. The combination <code>2>&1</code> tells linux to merge both of these output streams and then <code>>></code> is appending the output redirect to the specfied file.

==Mailing Crontab Output==
Additionaly modiifications can be made to actualy mail the output of the cron script. We can do this by changing the <code>MAILTO</code> environment variable defined in the <code>/etc/crontab</code> file or in your crontab scripts in <code>/etc/cron.d/</code> with an actual valid email.

To set an email specificaly for 1 cronjob. This can be done by appending the email to the cron call like this:
 <nowiki>
*/10 * * * * /home/foo/script.sh 2>&1 | mail -s "cronjob ouput" validemail@domain.com</nowiki>
You can see the STDOUT and STDERR redirects here are being piped ( "|" ) into the mail command as mail content

==Notes==
If your crontab is not working check:
* file permissions on the file being executed by cron
* what user is being used by cron to execute the script
* whether SELinux is interfering with the scripts execution
* spacing may or may not matter depending on the machine


* Note cron requires absolute paths to run any system tool (/bin/ls for ls)
** This also means absolute paths to other files or resources the cron job script may need to access
* cron also only can use bourne shell commands (commands only available through the bourne shell, NOT Bourne-Again Shell)

==Sources==
