## Install MySQL
There is a bug where resetting the root password at the wrong point during the installation will cause MySQL to be installed on Ubuntu in a bad state. The following steps circumvent this issue.

### Ubuntu
1. Run `sudo apt-get install mysql-server` and let the install execute. During this install you will be prompted to reset the root password. <b>Do Not Reset The Password At This Point</b>. Leave the field blank and carry on with the installation. MySQL will use its default password at this point.
2. Run `sudo mysql_install_db`. If the previous step worked, this will run regardless of what directory you are located in
3. Run `sudo mysql_secure_installation`. This will prompt you through a number of steps to secure your MySQL database. One of these steps includes resetting the root password. <b>Reset the Root Password When Prompted During This Command Execution</b>

## Create A User For A Database With Full Access
The following allows you to create a new user and give them full permissions to either a specific table in a database or a whole database. This assumed you have already installed MySQL

1. Login to MySQL through terminal: 
    ```bash
    mysql -u root -h localhost -p
    ``` 
    Press enter and mysql will prompt for your password
2. Execute 
    ```sql
    CREATE USER <username> IDENTIFIED BY <password>;
    ```
    This will create your user
3. Execute one of the following lines depending on what permissions you wish to give:

    | Command | Description |
    | ------- | ----------- |
    | `GRANT ALL PRIVILEGES ON <database>.* TO <username>;` | Give a user full privileges to any table of a specific database |
    | `GRANT ALL PRIVILEGES ON <database>.<tablename> TO <username>;` | Give a user full privileges to a specific table in a specific database |
    | `GRANT ALL PRIVILEGES ON *.* TO <username>;` | Give a user full privileges to any table of any database. This essentially gives the user the equivalent permissions as the root user |

Some additional calls that may be of use when creating these users:

| Command | Description |
| ------- | ----------- |
| `SELECT User, Host, Password FROM mysql.user` | List all the known users and their host access. '%' Host means only over localhost |
| `SHOW GRANTS FOR <username>` | List all grant permissions given, this is useful to double check the privileges assigned have gone through |

## Resources

* Digital Ocean LAMP tutorial: [https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-14-04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-14-04)
* MySQL CREATE USER Docs: [https://dev.mysql.com/doc/refman/5.1/en/create-user.html](https://dev.mysql.com/doc/refman/5.1/en/create-user.html)
* MySQL GRANT Docs: [https://dev.mysql.com/doc/refman/5.1/en/grant.html](https://dev.mysql.com/doc/refman/5.1/en/grant.html)
