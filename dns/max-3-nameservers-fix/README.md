Fix for supporting maximum 3 nameservers
==============

libc supports only 3 entires in /etc/resolv.conf. When opening multiple OpenVPN connections, this causes some DNS servers to not be queried.

Idea
-----
The idea is to have openvpn, like any other linux program, report its nameservers to resolvconf, which keeps track of all 
dns servers and writes out configuration for various resolvers. I will use bind9 as a resolver, since it supports more than three dns servers. 

This solution uses an ugly solution to get the dns nameservers from resolvconf and to pass them to bind9, and should be replaces with a proper resolvconf hook script

Steps
----
1. Append the following line to all openvpn configuration files
<pre>
script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
</pre>
2. Install bind9 and configure it as a forwarding server (https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-caching-or-forwarding-dns-server-on-ubuntu-14-04)
3. Create a seperate file for the forwarding options (/etc/bind/named.conf.options.forwarders for example)
4. Remove the resolv.conf link and point it to bind9
<pre>
rm /etc/resolv.conf
echo nameserver 127.0.0.1 > /etc/resolv.conf
</pre>
5. Now comes the ugly part: Run the following script every time resolvconf gets updated. This scripts reads the resolvconf directly and fills in the forwarders options of bind9
<pre>
cat /run/resolvconf/interface/* | awk 'BEGIN{print"forwarders {"}$1=="nameserver"&&$2!~/^127/{print $2";"}END{print "};"}' > /etc/bind/named.conf.options.forwarders
</pre>
