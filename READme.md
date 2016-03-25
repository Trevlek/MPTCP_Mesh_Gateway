# MPTCP_Mesh_Gateway
MPTCP enable Openwrt built with TCP to MPTCP gateway proxy for Mesh gateway WMN

This build is based on OpenWRT 14.07 - Barrier Breaker github by Mario Krüger.  Build instructions can be found on the OpenWRT-wiki(http://wiki.openwrt.org/doc/uci/mptcp), and is based on MPTCP v0.89.x. 

This software/firmware is develop and prototyped at the Wireless Research Lab at Algonquin College in Ottawa ON, Canada. The gold of the project was to create gateway software that will effectively build a self-forming Wireless Mesh Network(WMN) between neighboring gateways of the same ISP provider.  It will requiring little to no configuration from users of the network when deployed, with no requirement for modifcation of user devices or webservers on the Internet. The gateway software will manage the task of IP address assignment and translation, building MPTCP compatible routing tables for bandwidth aggregation from transparent multipath sub-flows. The result is a gateway software topology that is stacked using publicly available open source software implementations. When deployed gateway routes and client devices will be able to achieve greater bandwidth and network resilience, with the ability to achieve fast failover from logical and physical link failure.

The Mario Krüge OpenWRT 14.07 - Barrier Breaker which is available on github has been modified and stacked with software packages and custom scripting that will achieve the gold of a firmware that will create a ready to use selfforming MPTCP enabled WMN that increase network throughputs and resiliency.

This software creates a network topology where the routers build a WMN between in range neighboring gateway routers.  It is assumed that each gateway router belong to an Internet service subscribers of an ISP with full route to the Internet through a DSL connection. Because each router has a direct connection.  Each router will gain additional indirect routes to the Internet via the WMN mesh interface. There will be not need for modification on user devices and webservers, however, network traffic is generate on user device and for MPTCP subflows to be created both client device and server needs to be MPTCP enabled.  This will not be practical, we get around this through a method by which TCP traffic generated on end user device and is intercepted and proxyed in gateway routers to create MPTCP subflows.  

The proxy server in the gateway router acts as a man in the middle between the end user device and end servers.  when user device initiate a TCP connection, the gateway router intercept and initiate the sessions on behave of the user device.   The gateway router uses MPTCP syn packets to establish subflows over direct and indirect routes to the Internet via home and neighboring routers DSL link.  In this package we implement light weight HAproxy as our gateway proxy servers.  However, because most webservers are not MPTCP capable MPTCP packets will have to be terminated before the reach their final destination.  In our Algonquin College Test bed we choose to terminate MPTCP subflows on ISP routers before they are directed to their final destination.  If you are running your own MPTCP enabled server then termination is not necessary on ISP routers. If traffic is going through multiple ISP's then network traffic will have to be redirected to an alternate MPTCP gateway that will terminate MPTCP subflows and establish a regular TCP session between gateway routers and webservers.  This way TCP traffic is proxy to MPTCP and vice versa in gateway routers.

The software is designed for dual band AC routers, a 2.4 GHz and for creating the WMN backhaul and a 5 GHz band that is use as a client access Point for user devices. For the WMN interface we utilize open source packages.   we use 802.11s to create the mesh network and finding peers, OLSRD is our IP routing protocol and Avahi-autoipd is used for IPv4 address autoconfiguration on the mesh interfaces which is crated as mesh0. More information on this package and the Algonquin College testbed can be found the published papers: 

'Autoconfiguration for Faster WiFi Community Networks ' by  Kelvert Ballantyne ; Fac. of Technol. & Trades, Algonquin Coll., Ottawa, ON, Canada ; Wahab Almuhtadi ; Jordan Melzer. 
 
'Bandwidth Aggregation using MPTCP and WMN Gateways'  by  Kelvert Ballantyne ; Fac. of Technol. & Trades, Algonquin Coll., Ottawa, ON, Canada ; Wahab Almuhtadi ; Jordan Melzer. 

And also,  'Aggregating Internet Access in a Mesh-Backhauled Network through MPTCP Proxying' by Thanh-Hieu Nong ; Fac. of Technol. & Trades, Algonquin Coll., Ottawa, ON, Canada ; Rita Wong ; Wahab Almuhtadi ; Jordan Melzer. 

To achieve our gole of a softawre that create a self-forming Wireless Mesh Network, assign IP addresses, and build MPTCP routing tables, as well as a network manager that always points the default default route to a connected interface with routes to the Internet, we have created soome scripts to achieve this.  These scripts can be found in the file builder in the build root directory. below is a list of these scripts and what they do.

/etc/avahi/avahi-autoipd.action		##this is so addressing sets setup on interface name mesh0 and not avahi:mesh0.
/etc/config/dhcp			###sets up DHCPv6 for prefix deligation
/etc/conn_mngr				###This is our IPv4 connecction manager
/etc/conn_mngr6				###This is our IPv6 connecction manager
/etc/haproxy.cfg			###This is our proxy server listening on port 8888
/etc/init.d/gateway 			###The software default config is in an MPTCP gateway mode, how you can use this 						to turn it on or off. Off will 	bring the mesh interface down and stop connection 						managers and routing table scripts.
/etc/mptcp-r6				###sets up IPv6 MPTCP routing tables
/etc/setup-gwy				###configures the router into MPTCP mesh gateway mode
/etc/setup_mesh				###Sets up mesh interface
/etc/mesh_down				###takes down mesh interfaces
/etc/setup-olsrd			###starts olsrd
/etc/setup-rt				###IPv4 MPTCP routing table script
/etc/init.d/conn_mngr			###IPv4 connection manager init.d script to stop/start service
/etc/init.d/connn_mngr6			###IPv6 connection manager init.d script to stop/start service
/etc/init.d/mesh			####IPv6 routing table init.d script to stop/start service
/etc/init.d/olsrd/			###olsrd init.d script to stop/start service	
/etc/init.d/mptcp-rt			###IPv4 routing table init.d script to stop/start service
/files/usr/share/udhcpc/default.script  ###This script is a modified dhcp default script to add direct default global 						interface and unique table for this interface  when dhcp acquire and ip from 						upstream

