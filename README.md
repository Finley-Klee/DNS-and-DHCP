# DNS-and-DHCP
In this Qwiklab from the System Administration and IT Infrastructure course I used dnsmasq to set up and manage DNS and DHCP for a network.

<h2>Scenario</h2>
The company that you are working for has dnsmasq set up to manage the DNS and DHCP needs of the network.

Currently, almost all of the DHCP range is being used to serve dynamic IPs. A number of servers will be added to the network and those should be configured with known IPs.

My task in this lab was to modify the configuration of this dnsmasq setup so that those servers always have the same IPs. To do this, I gave those servers the requested IPs, and also reduced the range for the dynamic IPs.

<h2>Utilities Used</h2>

- <b>dnsmasq </b>
- <b>dhclient </b>
- <b>nano </b>

<h2>Environments Used </h2>

- <b>Linux </b>

<h2>Project walk-through:</h2>

- <b>Network Setup and Current Configuration</b>
<p>Before I could start making changes to the configuartions, I first needed to explore the current setup.</p>
<br>
<p align="center">I used the ip command to check the virtual network interface, eth_srv, which the DNS and DHCP server will listen on. The interface is configured to have the ip address 192.168.1.1 and the netmask is 255.255.255.0 as indicated by the /24 after the ip address. There is also a virtual network interface configured to simulate a client talking to the server, eth_cli, and by using the same command, ip, I see that the client interface does not have an ip address yet.<br/>
  <img src="https://github.com/user-attachments/assets/8f90834f-e302-44cd-8c94-305bc7831da5" width="80%" alt="a linux terminal window with black background and white text. The top command reads ip address show eth_srv and the response over the next 5 lines shows information about the network interface, including the ip address. The next command reads ip address show eth_cli and the following 3 lines show information about the client interface, including a mac address but no IPv4 address."/>
  <br />
  <br />
Next, I used the cat command to print the text of the current dnsmasq configuration file, located at /etc/dnsmasq.d/mycompany.conf, to the terminal so that I could learn what the current settings were. I can see that the interface is defined as eht_srv, and that the bind-interface setting means that dnsmasq will only operate on that interface and ignore any others. I can see that the domain for the network is mycompany.local and that there's additional information in the dhcp-option settings providing clients with the router and the dns server. Lastly, I can see the current range that is being dynamically configured by dhcp as well as the lease time, which is currenlty set to 24 hours.<br />
  <img src="https://github.com/user-attachments/assets/8b29a386-7595-42ce-8430-32bc0e113f77" width="80%" alt="a linux terminal window with black background and white text. The text reads: cat /etc/dnsmasq.d/mycompany.conf This is the interface on which the DCHP server will be listening to. Interface equals eth underscore srv. This tells this dnsmasq to only operate on that interface and not operate on any other interfaces, so that it doesn't interfere with other running dnsmasq processes. bind hyphen interfaces. Domain name that will be sent to the DHCP clients. domain equals my company dot local. Default gateway that will be sent to the DHCP clients. dhcp hyphen option equals option colon router comma 192 dot 168 dot 1 dot 1. DNS servers to announce to the DHCP clients. dhcp hyphen option equals option colon dns hyphen server comma 192 dot 168 dot 1 dot 1. Dynamic range of IPs to use for DHCP and the lease time. dhcp hyphen range equals 192 dot 168 dot 1 dot 2 comma 192 dot 168 dot 1 dot 254 comma 24 h."/>
</p>
<br />
<br />

- <b>Enabling Debug Logging</b>
<p>In order to better understand what is going on and why as I make changes to the configuration, I enabled debug logging.</p>
<br>
<p align="center">First, I queried the status of the dnsmasq service using the command sudo service dnsmasq status. I can see that the service is currently running.<br/>
  <img src="https://github.com/user-attachments/assets/6209e2e9-4362-42b3-92b2-7cc7065d41dc"" width="80%" alt="a linux terminal window with black background and white text. The first line has the command sudo service dnsmasq status. The response reads checking DNS forwarder and DHCP server colon dnsmasq running. Running is in parentheses."/>
  <br />
  <br />
Then, I opened the configuration file for dnsmasq in the nano text editor using the command sudo nano followed by the file path for the configuration file. I edited the file to add the option to log queries, and directed the service where to store the log file.<br />
  <img src="https://github.com/user-attachments/assets/0f177d52-74c2-4a93-94f9-d79b1d1bef78" width="80%" alt="A nano text editor window with black background and white text. The text of the configuration file is the same as was described in the network set up and current configuration section with the exception of the last two lines which read: log hyphen queries and log hyphen facility equals /var/log/dnsmasq.log"/>
  <br />
  <br />
 Finally, I tested that the syntax of the configuration file was correct, using the --test parameter, and after confirming that it was OK, I started the dnsmasq daemon so that it would read the configuration file again.<br />
  <img src="https://github.com/user-attachments/assets/5cee2a53-6e52-4dd9-a532-40b46902c6d1" width="80%" alt="a linux terminal with black background and white text. The first command reads sudo dnsmasq double hyphen test hyphen C /etc/dnsmasq.d/mycompany.conf and the response reads dnsmasq colon syntax check OK. The second command reads sudo service dnsmasq start and the response reads starting DNS forwarder and DHCP server colon dnsmasq."/>
</p>

- <b>Experimenting with DNS queries</b>
<p>I observed how dnsmasq handles DNS queries by using the dig command to request the IP address of certain hostnames.</p>
<br>
<p align="center">First, I queried dnsmasq for the IP address of example.com and specified that I want to use the running machine as the DNS server by using the @localhost parameter. In the answer section I actually got 6 different responses for A records for the domain name.<br/>
  <img src="https://github.com/user-attachments/assets/22116447-cdf4-4ace-be23-21a5ec0dd23c" width="80%" alt="a linux terminal with black background and white text. The command in the top line reads dig example dot com at localhost. The response shows no errors, one query with six answers and then in the bottom answer section lists the A records for example dot com which has 6 IPv4 addresses."/>
  <br />
  <br />
 Then I used the debug logs from dnsmasq to see how dnsmasq handled the query. I used the tail command to print the last few lines of the debug log file to the terminal, and I can see that the local host checked its host file at /etc/hosts for the IP address for example.com and didn't find anything, so it forwarded the request to an external DNS server and received the responses listed in the dnsmasq response. This is normal behavior for a caching DNS service.<br />
  <img src="https://github.com/user-attachments/assets/f18a5689-aeb8-445c-9a67-7d5424ee53df" width="80%" alt="a linux terminal with black background and white text. The command on the first line reads sudo tail /var/log/dnsmasq.log followed by the last 10 lines of the log file. The log entries have the date, May 2, and time, 18:15:18 to 18:16:52, on the left hand side, then the process, dnsmasq, and the process id, 1270, followed by a colon and then the responses. From top to bottom the responses read using nameserver 169 dot 254 dot 169 dot 254 # 53, read /etc/hosts hyphen 7 addresses, query A example dot com from 127 dot 0 dot 0 dot 1, forwarded example dot com to 169 dot 254 dot 169 dot 254, reply example dot com is 23 dot 192 dot 228 dot 80, reply example dot com is 23 dot 192 dot 228 dot 84, reply example dot com is 23 dot 215 dot 0 dot 136, reply example dot com is 23 dot 215 dot 0 dot 138, reply example dot com is 96 dot 7 dot 128 dot 175, reply example dot com is 96 dot 7 dot 128 dot 198"/>
  <br />
  <br />
 I checked the caching ability by running the same query again, and here you can see from the tail of the debug log that this time dnsmasq returned cached addresses for example.com instead of forwarding the request.<br />
  <img src="https://github.com/user-attachments/assets/8410e0a2-f9e8-420b-a0c4-eb2028bf7920" width="80%" alt="a linux terminal with black background and white text. The command in the top line reads dig example dot com at localhost. The response shows no errors, one query with six answers and then in the bottom answer section lists the A records for example dot com which has 6 IPv4 addresses."/>
   <br />
  <img src="https://github.com/user-attachments/assets/a82f0eff-c1ea-4b7c-9c04-c7aa7da655d8" width="80%" alt="a linux terminal with black background and white text. The command on the first line reads sudo tail /var/log/dnsmasq.log followed by the last 10 lines of the log file. In the last 7 lines the query from the dig command can be seen and this time the 6 responses are appended with cached indicating that the response came from the host file."/>
   <br />
  <br />
  I also explored what would happen if I queried for a domain name that doesn't exist. I used the dig command again, this time with the domain example.local. I can see from the response that I get a warning that .local is reserved for multicast DNS and I see that my 1 query received 0 answers. I can further explore this scenario by once again examining the debug log with tail. I see in the last two lines that the query was received and forwarded to the external DNS server and no response was received.<br />
  <img src="https://github.com/user-attachments/assets/866aa6cb-6fd2-434b-aa44-39f0cc961071" width="80%" alt="a linux terminal with black background and white text. The command in the top line reads dig example dot local at localhost. The response contains a warning which reads dot local is reserved for multicast DNS you are currently testing what happens when an mDNS query is leaked to DNS. Bellow the dig response is the command sudo tail /var/log/dnsmasq.log and the last two lines of the log show the query of the A record for example dot local and then the forwarding of that response to the external DNS server."/>
</p>

- <b>Experimenting with a DHCP Client</b>
<p>To better understand how the DHCP configuration is working, I ran dhclient on the eht_cli network interface.</p>
<br>
<p align="center">I ran the DHCP client using verbose mode so that I could see the response and using a debugging script (the -sf flag) so that I could see what information was received from the server instead of modifying the network settings of the machine. I can see in the verbose response how the DHCP server discovered the client, offered the IP address of 192.168.1.148, which was then requested and acknowledged. So now that IP address is bound to the client for 33331 seconds (approximately 9 hours and 26 minutes)<br/>
  <img src="https://github.com/user-attachments/assets/f08ec04a-0c33-44db-8015-fd2b40440968" width="80%" alt="linux terminal with a black background and white text. The command in the first line reads sudo dhclient hyphen i eth underscore cli hyphen v hyphen sf /root/debug_dhcp.sh and the response starts with the copyright from internet systems consortium, then follows with the back and forth of the client and server exchanging the ip address information and then the variables received including the host name and ip address, and lastly a line showing that the ip address is bound to the client and when the address renewal will happen."/>
  <br />
  <br />
  I also checked the debug file dnsmasq and was able to see that the options set in the configuration file were correctly sent to the client. <br />
  <img src="https://github.com/user-attachments/assets/2f8723a2-a47c-4b15-a29e-53e705e68845" width="80%" alt="linux terminal with a black background and white text. The last 5 lines of the debug log file show the dhcp discover on the eth serv followed by the client's mac address, then the offer of the ip address from the server to the client mac address, followed by the request to the server and then the acknowledgement."/>
</p>
<br />
<br />

- <b>Changing the Configuration</b>
<p>For the last step of this lab, after exploring how the DNS and DHCP services are currently set up, I made changes to the configuration file to assign fixed IP addresses to the new servers, leave space to add more servers in the future, and update the dynamic range accordingly.</p>
<br>
<p align="center">Before editing the configuration file, I first stopped the dnsmasq service, then opened the file with the nano text editor. The company wants to add 3 new servers, so I added new dhcp hosts (boxed in green) with their mac address and the fixed IP address intended for them to the configuration file between the assignment of the dynamic range and the debug log options. I also reduced the dynamic IP range to begin at 20 instead of 2 and shortened the lease time from 24 hours to 6 hours (boxed in blue).<br/>
  <img src="https://github.com/user-attachments/assets/3a6efb93-793b-4048-a684-bad8c2da0cad" width="80%" alt="linux terminal with black background and white text. The top command reads sudo service dnsmasq stop. The response reads stopping DNS forwarder and DHCP server colon dnsmasq. The second command reads sudo nano /etc/dnsmasq.d/mycompany.conf"/>
  <br />
  <img src="https://github.com/user-attachments/assets/173a5ad3-5adf-453e-b655-942e79076ea3" width="80%" alt="a nano text editor window with a black background and white text. The configuration file is shown and has changes highlighted by a blue rectangle and a green rectangle. The blue rectangle surrounds the line which reads dhcp hyphen range equals 192 dot 168 dot 1 dot 20 comma 192 dot 168 dot 1 dot 254 comma 6 H. The green rectangle surrounds three lines, each of which follow the format dhcp hyphen host equals then the mac address, each of which start with aa colon bb colon cc colon dd colon ee and differ only in the last two places, which are, from top to bottom, b2, c3, and d4. then there is a comma and the ip address for each server. the ip addresses all start with 192 dot 168 dot 1 and then from top to bottom the last digits are 2, 3, and 4."/>
  <br />
  <br />
Then I verified the configuration file syntax by running the test, and since I got no errors I started the dnsmasq service again.<br />
  <img src="https://github.com/user-attachments/assets/31b8834a-33d2-4f89-a662-99ac548f3aae" width="80%" alt="linux terminal with black background and white text. The first command reads sudo dnsmasq hyphen hyphen test hyphen c /etc/dnsmasq.d/mycompany.conf. The response reads dnsmasq colon syntax check OK. The second command reads sudo service dnsmasq start and the response reads starting DNS forwarder and DHCP server colon dnsmasq."/>
   <br />
  <br />
To test the change of the configuration, I changed the MAC address for the virtual network interface to one of the addresses specified in the configuration file, then I used dhclient again to get the IP address for the client, and I can see that it has the IP address specified in the configuration file for the matching MAC address.<br />
  <img src="https://github.com/user-attachments/assets/7655dbcf-a3c1-473d-a6e0-ae6ed4c0a1f4" width="80%" alt="linux terminal with black background and white text. The top command reads sudo ip link set eth underscore cli address aa colon bb colon cc colon dd colon ee colon c3. The next command reads sudo ip address show eth underscore cli. The response reads eth underscore clie at eth underscore srv colon broadcast comma multicast comma up comma lower underscore up mtu 1500 qdisc noqueue state up group default qlen 1000 link/ether aa colon bb colon cc colon dd colon ee colon c3 brd ff colon ff colon ff colon ff colon ff colon ff"/>
   <br />
  <img src="https://github.com/user-attachments/assets/2e072afe-afc5-4ffc-b947-b9c33e68a958" width="80%" alt="linux terminal with black background and white text. The command at the top reads sudo dhclient hyphen i eth underscore cli hyphen v hyphen sf /root/debug_dhcp.sh. The response starts with a copyright from internet systems consortium followed by the listening and sending interfaces with the MAC address for the client aa colon bb colon cc colon dd colon ee colon c3, then two DHCP requests for the ip address from the previous assignment, followed by a dhcp discover of the client, which is then offered the ip address 192 dot 168 dot 1 dot 3, that exchange is documented and then the variables exchanged are listed ending with the ip address bound and renewal in 8244 seconds."/>
   <br />
  <br />
Finally, I checked the dnsmasq debug logs to see that the client tried to request the previous IP address, but the DHCP server checked the configuration file and the MAC address had a fixed IP address in the configuration, it did not assign the old IP address, then through a DHCP discover the process of binding the fixed IP address took place.<br />
  <img src="https://github.com/user-attachments/assets/354d8046-1315-4a6f-9fd3-3722d8b7b469" width="80%" alt="linux terminal with black background and white text. The top line has the command sudo tail /var/log/dnsmasq.log and the dhcp requests are shown in the lines of the log."/>
</p>
