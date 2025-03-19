==Install MySQL==
There is a bug where resetting the root password at the wrong point during the installation will cause MySQL to be installed on Ubuntu in a bad state. The following steps circumvent this issue.
===Ubuntu===
# Run <code>sudo apt-get install mysql-server</code> and let the install execute. During this install you will be prompted to reset the root password. <b>Do Not Reset The Password At This Point</b>. Leave the field blank and carry on with the installation. MySQL will use its default password at this point.
# Run <code>sudo mysql_install_db</code>. If the previous step worked, this will run regardless of what directory you are located in
# Run <code>sudo mysql_secure_installation</code>. This will prompt you through a number of steps to secure your MySQL database. One of these steps includes resetting the root password. <b>Reset the Root Password When Prompted During This Command Execution</b>

==Create A User For A Database With Full Access==
The following allows you to create a new user and give them full permissions to either a specific table in a database or a whole database. This assumed you have already installed MySQL

# Login to MySQL through terminal: <code>mysql -u root -h localhost -p</code> . Press enter and mysql will prompt for your password
# Execute <code>CREATE USER <username> IDENTIFIED BY <password>;</code> . This will create your user
# Execute one of the following lines depending on what permissions you wish to give:
#* <code>GRANT ALL PRIVILEGES ON <database>.* TO <username>;</code> To give a user full privileges to any table of a specific database
#* <code>GRANT ALL PRIVILEGES ON <database>.<tablename> TO <username>; </code> To give a user full privileges to a specific table in a specific database
#* <code>GRANT ALL PRIVILEGES ON *.* TO <username>;</code> To give a user full privileges to any table of any database. This essentially gives the user the equivalent permissions as the root user

Some additional calls that may be of use when creating these users:

* <code>SELECT User, Host, Password FROM mysql.user</code> To view all the known users and their host access. '%' Host means only over localhost
* <code>SHOW GRANTS FOR <username></code> To view all grant permissions given, this is useful to double check the privileges assigned have gone through

==Notes==

==Sources==

Digital Ocean LAMP tutorial<br>
https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-14-04<br>
MySQL CREATE USER Docs<br>
https://dev.mysql.com/doc/refman/5.1/en/create-user.html<br>
MySQL GRANT Docs <br>
https://dev.mysql.com/doc/refman/5.1/en/grant.html<br>
