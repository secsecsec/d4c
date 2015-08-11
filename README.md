D4C
===

DNS DDoS Defence and Countermeasure.

d4c works as non-learning bridge and filter specified DNS packets that 
are suspected as DDoS traffic.


	 % ./d4c 
	 Usage of d4c
	 	 * required options.
	 	-r : Right interface name
	 	-l : Left interface name
	 
	 	 * DNS filtering options.
	 	-m : Filter suffix of DNS Query Name
	 	-d : Destination prefixes of filtered DNS responses
	 	-s : Source prefixes of NOT filtered DNS responses
	 
	 	 * misc.
	 	-q : Max number of threads for interface
	 	-e : Number of Rings of a vale port
	 	-f : Daemon mode
	 	-v : Verbose mode
	 	-h : Print this help


1. Drop DNS response packet from outer networks
-----------------------------------------------

DNS DDoS traffic is generated by open resolvers around the world.
When assuming that "correct DNS responses are sent from correct resolver
servers", DNS responses from resolver servers in outer own networks are fully
incorrect.

So, d4c drop incorrect DNS response packets that match following conditions.

- The packet is DNS response (QR flag is 1).
- The packet is response from a resolver server (Authoritative flag is 0).
- The destination address of packets is in the prefixes specified `-d` options.
- The source address of packets is not in the prefixes specified `-s` options.

-d option helps that you can apply this filter for specified hosts and networks.
Moreover, -s option allows that using resolver servers in outer own network
such as Google Public DNS.

Example)

	 sudo ./d4c -l p2p1 -r p2p2 -d 10.10.0.0/16 -d 172.16.0.0/16 -s 8.8.8.8/32

The filter is applied for the networks 10.10.0.0/16 and 172.16.0.0/16,
and DNS response packets from Google DNS are allowed.


2. Drop DNS query packet with specified QNAME
---------------------------------------------

Query to generate DDoS traffic often contains striking QNAME.
Dropping these query packets with striking QNAME helps reducing DDoS traffic.
d4c provides a query filter function with QNAME _suffix_ match.
-m options specify QNAME as suffix.

Example)

	 sudo ./d4c -l p2p1 -r p2p2 -m hogehoge.com -m hugahuga.com

All queries for `.*hogehoge.com` and `.*hugahuga.com` are dropped.


Filter rule 1 and 2 works independently. Specifying source/destination prefixes
works only for rule 1.

Compile
-------

d4c is a netmap application. netmap.ko and netmap driver for interfaces used
by d4c must be installed in advance. In order to compile, `netmap.h` and	`netmap_user.h` must be installed on `/usr/include/net/`.
