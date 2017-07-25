# SOCKS 105 prototype based on shadowsocks-libev

This is an early SOCKS 6 prototype (SOCKS 105 is our internal working name) based on shadowsocks-libev. Please see README.original.md to familiarize yourself with shadowsocks.

The following apps have been ported:
 * ss-redir (transparent proxifier)
 * ss-local (SOCKS 5 to SOCKS 105 translator)
 * ss-server (proxy)

The other apps (ss-nat etc.) still speak the shadowsocks protocol and WILL NOT interoperate with ss-server.

## Differences from the SOCKSv6 draft

 * Authentication replies are called initial replies
 * Operation replies are called final replies
 * Messages do not carry options. Instead, requests and initial (authentication) replies carry authentication data blobs.

## Extra compilation steps

Before running make, the socks105 library must be compiled:
cd socks105
qmake
make

## Running SOCKS 105

The server can be deployed as follows:

./ss-server -p 1080 -m plain --fast-open 


The transparent proxifier (ss-redir) requires some iptables rules. The following example proxies all TCP traffic from the user proxyme:

iptables -t nat -N SHADOWSOCKS
iptables -t mangle -N SHADOWSOCKS
iptables -t mangle -N SHADOWSOCKS_MARK

iptables -t nat -A SHADOWSOCKS -p tcp -m owner --uid-owner proxyme -j REDIRECT --to-ports 12345

iptables -t nat -A OUTPUT -p tcp -j SHADOWSOCKS
iptables -t mangle -A PREROUTING -j SHADOWSOCKS
iptables -t mangle -A OUTPUT -j SHADOWSOCKS_MARK


./ss-redir -s 127.0.0.1 -p 1080 -l 12345 -m plain --fast-open [--force-tfo]

## TFO on the proxy-server leg

As per the draft, the SOCKS proxy does not attempt to use TFO to connect to the server unless the proxy client requests it.

Newer Linux kernel versions (4.4+) allow userspace applications to inspect the SYN of an accepted conection. By deffault, ss-redir will ask for TFO only if it detects that the original client attempted TFO. You can override this behavior with the "--force-tfo" argument.
