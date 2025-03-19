Bind9 is one of the most popular unix dns handling systems available. This documentation is primarily a reiteration of the included digital ocean source listed below with some personalised extra information to improve clarity

==Debian 9==
The debian 9 configuration instructions are setup for a single server hosted setup. Thus there is only one server with bind9 installed. This single server both operates and the name server and records server. With only a single server there is also no backup / secondary server as described in the digital ocean documentation

===Installation===
To install Bind9 on Debian9 execute the following commands
 <nowiki>
sudo apt install bind9 bind9utils bind9-doc</nowiki>

===Configure IPv4 Mode===
Configuring IPv4 Mode means bind9 will only handle requests over IPv4. This essentially just reduced configuration work as the IPv6 requires additional settings in bind9's config files.

Edite the file <code>/etc/default/bind9</code> and add to the top of the file the following line:
 <nowiki>
OPTIONS="-u bind -4"</nowiki>
This will force bind9 to boot in IPv4 mode. You will need to restart bind9 for the change to take effect

===Configure The DNS Server===
Open the file <code>/etc/bind/named.conf.options</code> and enter the following:
 <nowiki>
options {
        directory "/var/cache/bind";

        recursion yes;                 # enables resursive queries
        allow-recursion { any; };      # allows recursive queries from any clients
        listen-on { <privateip>; };    # dns servers public IP address
        allow-transfer { none; };      # disable zone transfers by default

        forwarders {
                8.8.8.8;
                8.8.4.4;
        };
};</nowiki>
Note that the above configuration has set <code>allow-recursion</code> to any which means any IP can make DNS look-ups requiring recursion. This is generally a security hazard but for an internal private network this may not be a huge deal. You can always change this value to <code>trusted</code>. 

IF you change the <code>allow-recursion</code> value to <code>trusted</code>, add the following section also to the top of the configuration file:
 <nowiki>
acl "trusted" {
        <privateip>;     # dns servers public ip address
        <clientusingdnsserver1>;  # host1
        <clientusingdnsserver2>;  # host2
};</nowiki>
By doing this you are only allowing hosts with the IPs listed to make recursive queries. You will need to fill this list with the ip of the dns server and all clients that will be using this dns server!

Also, this configuration has been setup with forwarders to <code>8.8.8.8</code> and <code>8.8.4.4</code>. These are the IPs of Google's DNS servers and are used when our dns server does not have the record
requested listed. This is useful if this DNS server will be referred to to resolve all domains - including those outside of the network. You can change this to a DNS server of your preference or remove the
section if you do not want any forwarding of DNS requests to occur.

===Configuring Zone Data===
Next edit the <code>/etc/bind/named.conf.local</code> file to add zone information. This file stores the name of the domains that will have their records stored on this server and where to find the zone file
information.

Copy the following into the <code>/etc/bind/named.conf.local</code> file:
 <nowiki>
zone "<yourfulldomain>" {
    type master;
    file "/etc/bind/zones/db.<yourfulldomain>"; # zone file path
};</nowiki>
Replace <code><yourfulledomain></code> with your full domain. This could be myprivatedomain.local or bensoer.com if you wanted this dns server to resolve those domains

===Configuring Zone Files===
You now need to create the zone file which you have configured in the previous section to refer to for zone data. The folder and path listed above may not exist, so run the following commands:
 <nowiki>
sudo mkdir /etc/bind/zones</nowiki>
Then create an open a file named <code>db.<yourfulldomain></code>. Copy the following into it
 <nowiki>
$TTL    604800
@       IN      SOA     <yourfulldomain>. root.<yourfulldomain>. (
                      YYYYMMDDV         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

; name servers - NS records
    IN      NS      ns1.<yourfulldomain>.

; name servers - A records
ns1.<yourfulldomain>.          IN      A       <dnsserverpublicip>
root.<yourfulldomain>.         IN      A       <dnsserverpublicip>

; A records</nowiki>
Replace all locations of <code><yourfulldomain></code> with your full domain. This could be myprivatedomain.local or bensoer.com if you wanted this dns server to resolve it. Note also to replace the <code>YYYYMMDDV</code> in the Serial value with the current Year Month Date and Version. As of this writing this should be 201811201. Note that keeping this number up to date is crucial with every update
as bind9 is only able to determine if changes have happened if this serial number is updated. Simply update it by updating the date OR increment the Version value if there are multiple updates within
the same day. Reset version back to 1 if the date has changed. This system not only allows for easy number generation but gives a helpful reminder to other administrators of when the last change was
made to the bind9 dns server.

All required configuration is now in place to resolve your domain. Now simply add A records in the zones file configured above. Add records under the A records comment in the same format as the nameserver
records specified above. You can use the following as a template:
 <nowiki>
<subdomain>.<yourfulldomain>.         IN      A       <iptoresolveto></nowiki>

Save your changes and restart the bind9 service

==Notes==

==Sources==
* https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-debian-9#testing-clients
