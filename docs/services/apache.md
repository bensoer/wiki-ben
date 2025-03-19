==File Locations==

===Fedora/CentOS===
* HTML Document Root : /var/www/html
* Main httpd config file: /etc/httpd/conf/httpd.conf
* Apache Daemon Directory: /etc/rc.d/init.d/httpd
===Ubuntu===
* HTML Document Root : /var/www/html
* Main httpd config file: /etc/apache2/apache.conf
* Virtual Host Configuration file: /etc/apache2/sites-available/000-default.conf

==Basic Install Apache==

===Fedora/CentOS===
sudo dnf install httpd OR sudo yum install httpd depending on Fedora version 

====Apache Status====
sudo systemctl httpd status<br>
sudo apachectl status

====Start Apache====
sudo systemctl start httpd.service<br>
sudo apachectl start

====Stop Apache====
sudo systemctl httpd stop<br>
sudo apachectl stop

====Set Apache To Start At Boot====
sudo systemctl enable httpd.service<br>
sudo apachectl enable

===Ubuntu===
sudo apt-get install apache2

====Apache Status====


====Start Apache====
sudo service apache2 start<br>

====Stop Apache====
sudo service apache2 stop<br>

====Restart Apache====
sudo service apache2 restart<br>

==Create User Account Site==
===Fedora/CentOS===
#Edit File: /etc/httpd/conf.d/userdir.conf
#Comment out the “UserDir disable” macro and uncomment the “UserDir public_html” macro in userdir.conf
#Create the useraccount (adduser, passwd)
#Login to the account (avoids some known errors)
#LOGOUT AND BACK IN AS ROOT
#Create the public_html folder inside the users account folder (if you logged in the folder will have Documents, Music, Downloads, etc.)

==Add Password Access To Site==
This is configuration guide in the appropriate conf file to cause a Basic Auth prompt to appear in the user's browser when viewing a page. Typically you will also want to generate a password file which will store the username and password to log into this area of your website. See the Create Password File section for steps on how to do that
===Fedora/CentOS===
Add this to /etc/httpd/conf.d/userdir.conf
 <nowiki>
<Directory /home/<username>
    AllowOverride None
    AuthUserFile <dirtopasswordfile>
    # Group authentication is disabled
    AuthGroupFile /dev/null
    AuthName test
    AuthType Basic
    <Limit GET>
        require valid-user
        order deny,allow
        deny from all
        allow from all
    </Limit>
</Directory></nowiki>

If the basic auth does not appear, check in the userdir.conf the stanza at the bottom the page blocking access to userdir is commented out

==Create A Password File==

htpasswrd [-c] <passwordfilename> <username>

Eg:<br>
htpasswd [-c ] passwordfile username

-c to create a new password file

do not include '-c' to update a password file


Note when generating this file, if using it for the above user profile site and with basic auth security, you may need to place this created file in the <code>public_html</code> folder of the user's profile site you are trying to access. This doesn't seem safe, but it works. The internet gives notions that the file should be named with a '.' so that it is hidden from the file system

==Create A Self-Signed Certificate For Apache==
===Ubuntu===
1. Create a folder create all of your self-signed certs in. cd into it<br>
2. Execute: 
 <nowiki>
sudo openssl req -new > new.ssl.csr</nowiki>

First your going to be prompted to create a password. Remember this password, you will need it in a later command

At this point you will be prompted to enter some information looking something like this:
 <nowiki>
Generating a 1024 bit RSA private key
................++++++
........................++++++
writing new private key to 'privkey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]: <countrycode (optional)>
State or Province Name (full name) [Some-State]: <state/province (optional)>
Locality Name (eg, city) []: <city (optional)>
Organization Name (eg, company) [Internet Widgits Pty Ltd]: <companyname (optional)>
Organizational Unit Name (eg, section) []: <orgunit (optional)>
Common Name (eg, YOUR name) []: <fullyqualifieddomainname (mandatory with virtual hosting)>
Email Address []: <email (optional)

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []: <leave blank>
An optional company name []: <companyname (optional)></nowiki>

Basicaly for any setup most of the fields are optional except the <b>Common Name</b> field. If you are setting up apache with VirtualHost then you will need to make sure this is the fully qualified domain name (eg. wiki.bensoer.com) OR whatever value you are placing in the <code>ServerAlias</code> field of your VirtualHost entry for apache. This is important as apache will deny access if the value in the ServerAlias and the Common Name in the Self-Signed Certificate don't match

Additionaly you should leave the challenge password blank, otherwise you will need to enter the password you put there everytime you reboot apache

3. Execute the Following commands:
 <nowiki>
sudo openssl rsa -in privkey.pem -out new.cert.key
sudo openssl x509 -in new.ssl.csr -out new.cert.cert -req -signkey new.cert.key -days 3600

sudo mkdir /etc/ssl/self-signed
sudo mkdir /etc/ssl/self-signed/certs
sudo mkdir /etc/ssl/self-signed/private

sudo cp new.cert.cert /etc/ssl/self-signed/certs/server.crt
sudo cp new.cert.key /etc/ssl/self-signed/private/server.key</nowiki>

After entering the first command in step 3 you will be prompted for a password. This is the password you created earlier in step 1

Note the <code>-days</code> command used in the second command of step 3 is setting the number of days the self-signed certificate is valid. If you would like to renew your self-sgined certificate more often then change this number to something smaller

After completing all commands in step 3 your self-signed certificates will be available in the <code>/etc/ssl/self-signed</code> folder. You can reference these directories in your VirtualHost configurations when setting up SSL for a VirtualHost

==Notes==
#Check Folder/File Permissions (sudo chmod 777 with caution)
#* Check index.html
#Check the Account was created correctly (login/logout)

==Sources==
Create Self-Signed Certificates
https://www.linux.com/learn/tutorials/392099-creating-self-signed-ssl-certificates-for-apache-on-linux
