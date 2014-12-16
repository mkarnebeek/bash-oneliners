Fix for supporting maximum three nameservers
==============

libc supports only three entires in /etc/resolv.conf. When opening multiple OpenVPN connections, this causes some DNS servers to not be queried.

Idea
-----
The idea is to have openvpn, like any other linux program, report its nameservers to resolvconf, which keeps track of all dns servers and writes out configuration for various resolvers. I will use bind9 as a resolver, since it supports more than three dns servers. 

The bind9 file in this repository is the resolvconf hook script passing the dns servers to bind9

Steps
----
1. Append the following line to all openvpn configuration files. This makes openvpn communicate the dns servers to resolvconf.
<pre>
script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
</pre>
2. Install bind9 and configure it as a forwarding server (https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-caching-or-forwarding-dns-server-on-ubuntu-14-04)
3. Place the following includes. These files get written by the resolvconf hook script.
  - in /etc/bind/named.conf.options <code>include "/etc/bind/named.conf.options";</code>
  - in /etc/bind/named.conf.local <code>include "/etc/bind/named.conf.local.forwardzones";</code>
4. Remove the resolv.conf link and point it to bind9
<pre>
rm /etc/resolv.conf
echo nameserver 127.0.0.1 > /etc/resolv.conf
</pre>
5. Put the bind9 file in this repository in /etc/resolvconf/update.d/ and mark it as executable. 
