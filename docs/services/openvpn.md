OpenVPN is a open source VPN application that uses a unique security protocol to implement virtual private networks.
Read more about it at:
*[https://en.wikipedia.org/wiki/OpenVPN Wikipedia]
*[https://openvpn.net/index.php/open-source/documentation.html OpenVPN Documentation]

== Install OpenVPN ==
OpenVPN comes with Easy-RSA, a lightweight package for using the RSA encryption method.<br>
The install needs to be run as root so execute <code>sudo su</code>
===Adding apt repositiories===
ONLY DO THIS IF YOU ARE RUNNING A i386 OR AMD64 ARCHITECTURE!<br>
Lets add OpenVPN's apt repository to our own list of repositories so that we can get a more up-to-date release than what is in your distribution's repositories.<br>
Run the following commands:
:<code><nowiki>wget -O - https://swupdate.openvpn.net/repos/repo-public.gpg|apt-key add -</nowiki></code><br>
:<code><nowiki>echo "deb http://build.openvpn.net/debian/openvpn/stable <osrelease> main" > /etc/apt/sources.list.d/openvpn-aptrepo.list</nowiki></code><br>
Where <osrelease> depends your distribution:
*squeeze (Debian 6.x)
*wheezy (Debian 7.x)
*jessie (Debian 8.x)
*lucid (Ubuntu 10.04)
*precise (Ubuntu 12.04)
*trusty (Ubuntu 14.04)

===Install=== 
Now run <code>apt-get install openvpn</code> and let the install execute. Easy-rsa should be installed along with it, otherwise [https://wiki.bensoer.com/index.php/Apt-get#Usage update and install your dependencies].

OpenVPN installs to <tt>/etc/openvpn</tt>.

== Setting Up SSL Certificates ==
Lets get cryptographic.
=== Preparing Easy-RSA ===
#Create an Easy-RSA folder in your OpenVPN install directory<br> <code>mkdir /etc/openvpn/easy-rsa</code>
#Move your Easy-RSA install to your OpenVPN install directory<br> <code>cp -r /usr/share/easy-rsa /etc/openvpn/</code>
#Navigate to the Easy-RSA folder we just created<br> <code>cd /etc/openvpn/easy-rsa</code>
#Edit the Easy-RSA variables file to generate keys in our new <tt>/etc/openvpn/easy-rsa</tt> directory<br> <code>nano vars</code><br> change EASY_RSA variable to: <code>export EASY_RSA="/etc/openvpn/easy-rsa"</code><br> and change KEY_SIZE variable to: <code>export KEY_SIZE=4096</code><br> also change the default values for <tt>KEY_COUNTRY="US"</tt>, <tt>KEY_PROVINCE="CA"</tt>, <tt>KEY_CITY="SanFrancisco"</tt>, <tt>KEY_ORG="Fort-Funston"</tt>, <tt>KEY_EMAIL="me@myhost.mydomain"</tt>, and <tt>KEY_OU="MyOrganizationalUnit"</tt> so you don't have to do it multiple times later in the key generation process.

=== Building and Signing you Certificate Authority ===
#Build your certificate authority(CA):<br> <code>source ./vars</code><br><code>./clean-all</code><br><code>./build-ca</code>
#During the build process it will prompt you for some name values for the CA. If you modified and sourced the vars file correctly these values should default to the ones you entered earlier.
#Sign your CA:<br> <code>./build-key-server "''Server_Name''"</code><br> It will prompt you for those same values. <tt>Common Name</tt> must be the same as "''Server_Name''" you just picked. You MUST leave the <tt>Challenge Password</tt> blank.

=== Creating and Signing Clients ===
*For desktop clients use: <code>./build-key-pass "''UserName''"</code>
*For mobile clients use: <code>./build-key-pkcs12 "''UserName''"</code>
When prompted to <tt>Enter PEM pass phrase</tt> enter a password that will be used to encrypt your private key. Make sure to remember it as you will need it each time you want to connect to your VPN. Again you MUST leave the <tt>Challenge Password</tt> blank.<br> You could only generate one client key and use the same certificate for every device you wanted to connect to the VPN, however each device connected to the VPN concurrently needs to have a unique certificate and username.

=== Build the DH parameters ===
[https://wiki.openssl.org/index.php/Diffie_Hellman Diffie Hellman] parameters must be generated for the OpenVPN server. Run the following commands:<br>
:<code>cd /etc/openvpn/easy-rsa/</code><br>
:<code>./build-dh</code><br>
This process will take the longest time to compute as it is using random numbers and looking for some specific relationships.

=== OpenVPN's DoS Protection ===
OpenVPN has build in protection against a hacker finding out your server’s address, and generating such a large number of access requests that your server crashes. By generating a [https://en.wikipedia.org/wiki/Hash-based_message_authentication_code HMAC] key, your server wont attempt to authenticate unless it sees this static key first. Generate one by:<br>
:<code>openvpn –-genkey –-secret keys/ta.key</code>

=== Understanding your Keys ===
Now we will find our newly-generated keys and certificates in the keys subdirectory. Here is an explanation of the relevant files:
{| class="wikitable"
!Filename
!Needed By
!Purpose
!Secret
|-
|ca.crt||server + all clients||Root CA certificate||NO
|-
|ca.key||key signing machine only||Root CA key||YES
|-
|dh{n}.pem||server only||Diffie Hellman parameters||NO
|-
|server.crt||server only||Server Certificate||NO
|-
|server.key||server only||Server Key||YES
|-
|client1.crt||client1 only||Client1 Certificate||NO
|-
|client1.key||client1 only||Client1 Key||YES
|-
|client2.crt||client2 only||Client2 Certificate||NO
|-
|client2.key||client2 only||Client2 Key||YES
|-
|client3.crt||client3 only||Client3 Certificate||NO
|-
|client3.key||client3 only||Client3 Key||YES
|-
|ta.key||server + all clients||TA Key||YES
|}

The final step in the key generation process is to copy all files to the machines which need them, taking care to copy secret files over a secure channel like <code>scp</code>.<br><br>CONGRATULATIONS! You have now created all required SSL certificates needed for your VPN.

==Config Files==
It would take too long to go into details about everything in these files. You can find details in the resources mentioned above.
===Server Config===
Edit <tt>/etc/openvpn/server.conf</tt> to contain the following:
 <nowiki>
port 1194
proto udp
dev tun

ca ca.crt
cert server.crt
key server.key

dh dh4096.pem

tls-auth ta.key 0

topology subnet
server 10.8.0.0 255.255.255.0

# Add route for local subnet
push "route 192.168.0.0 255.255.255.0"

# all clients redirect their default network gateway through the VPN
push "redirect-gateway def1"

# DNS servers provided by opendns.com.
;push "dhcp-option DNS 208.67.222.222"
;push "dhcp-option DNS 208.67.220.220"
# Or Google Developers Public DNS
;push "dhcp-option DNS 8.8.8.8"
;push "dhcp-option DNS 8.8.4.4"
push "dhcp-option DNS 192.168.0.1"

client-to-client

keepalive 10 120

cipher AES-256-CBC

comp-lzo

max-clients 5

user nobody
group nogroup

persist-key
persist-tun

status /var/log/openvpn/openvpn-status.log
log /var/log/openvpn/openvpn.log
verb 3</nowiki>

===Client Config===
The following is used in a <tt>client.conf</tt> by the openVPN application on a client computer.
 <nowiki>
client
proto udp
dev tun

remote my-openVPN-server-IP 1194
resolv-retry infinite

nobind

user nobody
group nobody

persist-key
persist-tun

ca ca.crt
cert client.crt
key client.key

remote-cert-tls server

tls-auth ta.key 1

cipher AES-256-CBC

auth-nocache

comp-lzo

verb 3</nowiki>

===Server Firewall Setup===
Choose between a simple or more secure firewall script.
====Basic====
Since in this setup we are routing all of the clients internet traffic through the VPN we need to be configured to deal with this traffic somehow, such as by NATing it to the internet. This can be accomplished by:<br>
:<code><nowiki>iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE</nowiki></code><br>
Note that this solution will only get things up and running. It provides no security whatsoever to your VPN Server.

====Advanced====
Below is a link to a sample IPTables firewall that can be tweaked to control traffic passing through the VPN. This configuration includes the masquerading rules mentioned above along with additional security measures for your VPN firewall. This script includes rules to only allow systems located in the local network of the VPN can access the VPN via SSH. Additionally, VPN users cannot access the VPN Server or the Gateway Router, but do have access to all other machines in the local network, and the internet outside of the network. A commented section has been highlighted in the script for users to add additional rules to control what data is allowed through the network.

https://gist.github.com/bensoer/9fec3386a9460c42511bf8fd49b4a6d8

To use this script:
<ol>
<li>Download and copy it to your VPN server with IPTables installed.</li>
<li>Open the script using a text editor and update the variables listed in the USER CONFIGURATION section to match that of your network the VPN is being hosted in (see script comments for details).</li>
<li>Execute the script with the following command: <code>sudo ./openvpnfirewall.sh</code></li>
</ol>

Status messages will print out as each section of the firewall is applied. To change the firewall, simply update the script and rerun it. The script contains logic to clear the previous firewall settings
before applying the newly listed ones. If there is an error, follow the prompts given in the console with their locations in the script (a printout in the shell script is preceded with the command 'echo')
and adjust the code as needed.

====Script Execution and Firewall Persistance====
=====Using The Network Interface=====
In order for this iptables rule to be applied every time you turn on the VPN, you can turn it into a shell script and modify <tt>/etc/network/interfaces</tt> so it will execute the script every time it initializes the network interface eth0.
 <nowiki>
...
iface eth0 inet dhcp
        pre-up script.sh
...</nowiki>
=====Using iptables-save=====
You can also use <tt>iptables-save</tt> a tool available for Debian, Ubuntu, CentOS and RHEL based distros. After executing your script run the following command to export it:
 <nowiki>
#Debian / Ubuntu
iptables-save > /etc/iptables/rules.v4
ip6tables-save > /etc/iptables/rules.v6
#RHEL / CentOS
iptables-save > /etc/sysconfig/iptables
iptables-save > /etc/sysconfig/ip6tables</nowiki>
You may need to mkdir the directory if this is the first use. Ensure also that these directories only have read and write permissions for sudo, and only read for others (<tt>sudo chmod 777</tt>)

Then install <tt>iptables-persistent</tt>
 <nowiki>
sudo apt-get install iptables-persistent</nowiki>
The installation will prompt to re-export the current iptables rules again. You can say no at this step since we just did it. The iptables-persistance will then ensure that everytime the system
boots up the the iptables rules we exported above are impported and set. To update the persistence rules at a later time, simply re-export them as done initialy

To explore the <tt>iptables-save</tt> its co-tool <tt>iptables-restore</tt> and <tt>iptables-persistent</tt>. See the [[Iptables]] page for details

====Enabling Forwarding====
If you are running a Raspbian VPN you have to modify <tt>/etc/sysctl.conf</tt> to allow packet forwarding. Uncomment the line <code>net.ipv4.ip_forward=1</code><br>
Apply the changes we just made by executing: <code>sysctl -p</code>

==Revoking Certificates==
===Using Easy-RSA===
Easy-RSA handles revoking client certificates. Navigate to your Easy-RSA directory in your OpenVPN folder:<br>
:<code>cd /etc/openvpn/easy-rsa/</code><br>
Run the following to revoke the certificate for the client called 'unwanted-client':<br>
:<code>./revoke-full unwanted-client</code><br>
The index.txt file in you keys directory will be updated. You’ll see an ‘R’ (for Revoked) on the first column from the left for unwanted-client. View this with:<br>
:<code>cat keys/index.txt</code><br>
You can also examine the CRL (certificate revocation list) file with:<br>
:<code>openssl crl -in keys/crl.pem -text</code>
===Configure OpenVPN===
Now we need to configure our OpenVPN server to enable CRL verification. Add the following line to your <tt>server.conf</tt> file:<br>
:<code>crl-verify /etc/openvpn/easy-rsa/keys/crl.pem</code><br>
Reload the OpenVPN config file by:<br>
:<code>service openvpn reload</code><br>
When the crl-verify option is used in OpenVPN, the CRL file will be re-read any time a new client connects or an existing client renegotiates the SSL/TLS connection (by default once per hour). This means that you can update the CRL file while the OpenVPN server daemon is running, and have the new CRL take effect immediately for newly connecting clients.

==Adding LDAP for Active Directory Authentication==
LDAP is the underlying protocol of Active Directory. In an organization context where there may be multiple servers, controlling authentication in a single place may be desirable. This can also be applied to OpenVPN. OpenVPN can be configured to authenticate with both Certificates and LDAP or LDAP alone. The following instructions will configure for both, as it is the more secure option

===Installation===
On your linux machine install
:<code>apt-get install openvpn-auth-ldap</code><br>
Depending on whether you are RPM or Debian based will change where the module is installed. On Debian the module will be installed in /etc/openvpn/openvpn-auth-ldap.so. On RPM it is installed in /etc/openvpn/plugin/lib/openvpn-auth-ldap.so

You will also need to create the following directory to store your ldap configuration
:<code>mkdir /etc/openvpn/auth</code><br>

===Configuration===
Copy the following into ldap.conf inside your newly created directory:
 <nowiki>
<LDAP>
# LDAP server URL
URL ldap://dc-test-1.test.com:389
# Bind DN (If your LDAP server doesn't support anonymous binds)
#BindDN uid=admin,ou=Users,dc=test,dc=com
BindDN admin@test.com
# - for domain users <domain>\<username> also works

# Bind Password
Password humus
# - this is the password to login to the LDAP server to authenticate connecting users

# Network timeout (in seconds)
Timeout 15

# Enable Start TLS
TLSEnable no

# Follow LDAP Referrals (anonymously)
FollowReferrals yes

# - Only Add The Following if you are connecting securely to the AD/LDAP server
# TLS CA Certificate File
TLSCACertFile /usr/local/etc/ssl/ca.pem

# TLS CA Certificate Directory
TLSCACertDir /etc/ssl/certs

# Client Certificate and key
# If TLS client authentication is required
TLSCertFile /usr/local/etc/ssl/client-cert.pem
TLSKeyFile /usr/local/etc/ssl/client-key.pem

# Cipher Suite
# The defaults are usually fine here
# TLSCipherSuite ALL:!ADH:@STRENGTH
</LDAP>

<Authorization>
# Base DN
#BaseDN "CN=Users,DC=test,DC=com"
BaseDN "CN=Users,DC=test,DC=com"
# - BaseDN sets the base point to start searching for users. To search everywhere in the domain, remove the CN portion. Each DC is a part of the domain split by the dots

# User Search Filter
#SearchFilter "(&(uid=%u)(accountStatus=active))"
#SearchFilter "(&(sAMAccountName=%u)(msNPAllowDialin=TRUE))"
SearchFilter "(&(sAMAccountName=%u))"
# - SearchFilter is the search query made within the BaseDN location - the above one searchs for matching usernames

# Require Group Membership
# - Add this if you want to control the AD users needing to belong to a group. Comment out if not
RequireGroup true

# Add non-group members to a PF table (disabled)
#PFTable ips_vpn_users

# - This section is only required if RequireGroup is true, however it can still be here even if it is removed
<Group>
BaseDN "CN=Users,DC=test,DC=com"
# - BaseDN sets the base point to start searching for users. To search everywhere in the domain, remove the CN portion and replace each DC with each part of the domain
SearchFilter "(cn=vpn-users)"
# - vpn-users is the name of the group that users must be part of to join
MemberAttribute "member"
# Add group members to a PF table (disabled)
#PFTable ips_vpn_eng
</Group>
</Authorization></nowiki>

After saving and configurating the above contents in ldap.conf, add the following line to the bottom of the main server.conf
 <nowiki>
plugins /etc/openvpn/openvpn-auth-ldap.so "/etc/openvpn/auth/ldap.conf" </nowiki>
This tells openvpn to use the ldap plugin and load the just created plugin

Finaly, include the following line in the server.conf depending on what you would like to do:
* To force Certificate and LDAP authentication add <code>auth-user-pass</code> below the plugin line
* To only require LDAP authentication add <code>client-cert-not-required</code> below the plugin line

Finally give openvpn a restart to load the plugin. Check the logs to ensure configuration is correct (Authentication for LDAP typically is an issue if it is not the proper format)

For full details on this configuration see: https://www.allcloud.io/how-to/configure-openvpn-authentication-using-active-directory/ for details

== Notes ==
*<tt>easy-rsa</tt> might not have installed to <tt>/usr/share/easy-rsa</tt>. It should be somewhere in <tt>/usr/share</tt>, but you might have to search to find the exact path.
*The mathematical relation between a 4096 bit SSL key pair takes too long to generate on a small computer like a RaspberryPi, use 2048.
*the n in dh{n}.pem is the size of your SSL keys (2048 or 4096)

==Sources==

https://openvpn.net/index.php/open-source/documentation/howto.html<br>
http://readwrite.com/2014/04/10/raspberry-pi-vpn-tutorial-server-secure-web-browsing<br>
https://www.thomas-krenn.com/en/wiki/Saving_Iptables_Firewall_Rules_Permanently
<br><br>
https://openvpn.net/index.php/open-source/documentation/miscellaneous/77-rsa-key-management.html<br>
https://jamielinux.com/docs/openssl-certificate-authority/introduction.html
<br><br>
https://www.allcloud.io/how-to/configure-openvpn-authentication-using-active-directory/<br>
https://github.com/threerings/openvpn-auth-ldap/blob/master/auth-ldap.conf
