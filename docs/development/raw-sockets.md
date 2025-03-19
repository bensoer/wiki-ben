# Raw Sockets

The C++ Raw Sockets library is powerful, complicated and largely poorly documented. Numerous examples show inconsistent methods of solving a single solution and not many of these solutions seem to be able to resolve all functionality of Raw Sockets in a relatively clean and consistent way. This is what this article intends to try and clear up by describing a consistent method of developing, exploring and learning how to use all areas of Raw Sockets.

This documentation will cover largely the C libraries and will display examples using the C/C++ library. Much of Raw Sockets from here is the same in any language, and after studying Raw Sockets in C/C++ it is highly transferable to other languages and their libraries such as Python, Ruby or Java

==Create A Raw Socket==

Creating a standard socket is typicaly done like this:

    int socket = socket(AF_INET, SOCK_STREAM, 0);

This would create a IPv4 TCP Socket. The 3 parameters passed to the socket function all have special purposes and values. From the linux documentation the socket methods parameters are listed as:

    int socket(int domain, int type, int protocol);

The domain parameter handles the version, the type handles the type of protocol and the protocol parameter largely depends on the previous two parameters. This third parameter becomes of great use when using Raw Sockets as it dictates how the stack should react when sending or receiving packets. Below is a list of the various options that can be passed to each of these parameters. Note that this list is not comprehensive

<b>Domains</b>
<ul>
<li>AF_INET - Creates an IPv4 Socket</li>
<li>AF_INET6 - Creates an IPv6 Socket<li>
</ul>
<b>Type</b>
<ul>
<li>SOCK_STREAM - Creates A TCP Socket</li>
<li>SOCK_DGRAM - Creates A UDP Socket</li>
<li>SOCK_RAW - Creates A Raw Socket</li>
</ul>
<b>Protocols</b>
<ul>
<li>0 - This seems to create default behavior, if SOCK_DGRAM or SOCK_STREAM is used, the network stack will handle and generate an appropriate IP Header for the underlying protocol</li>
<li>IPPROTO_RAW - When used with SOCK_RAW this tells the stack by default that this is a Raw Socket and that the IP packet data will be supplied by the client<li>
<li>IPPROTO_TCP - When used with SOCK_RAW this tells the stack the data inputed will be TCP data. The network stack will create the IP Header data itself and will set the Protocol section of the IP Header to TCP</li>
<li>IPPROTO_UDP - When used with SOCK_RAW this tells the stack the data inputed will be UDP data. The network stack will create the IP Header data itself and will set the Protocol seciton of the IP Header to UDP</li>
</ul>

==Sources==
* Linux Raw Socket Man Page: http://man7.org/linux/man-pages/man7/raw.7.html
* Linux Socket Man Page: http://man7.org/linux/man-pages/man2/socket.2.html
* Linux IPv4 Implementation Man Page: http://man7.org/linux/man-pages/man7/ip.7.html

* Raw Sockets in C Code: http://www.binarytides.com/raw-sockets-c-code-linux/
==Notes==
