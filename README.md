# DNS-and-DHCP
In this Qwiklab from the System Administration and IT Infrastructure course I used dnsmasq to set up and manage DNS and DHCP for a network.

<h2>Scenario</h2>
The company that you are working for has dnsmasq set up to manage the DNS and DHCP needs of the network.

Currently, almost all of the DHCP range is being used to serve dynamic IPs. A number of servers will be added to the network and those should be configured with known IPs.

My task in this lab was to modify the configuration of this dnsmasq setup so that those servers always have the same IPs. To do this, I gave those servers the requested IPs, and also reduced the range for the dynamic IPs.

<h2>Utilities Used</h2>

- <b>dnsmasq </b> 

<h2>Environments Used </h2>

- <b>Linux </b>

<h2>Project walk-through:</h2>

- <b>Network Setup and Current Configuration</b>
<p>Before I could start making changes to the configuartions, I first needed to explore the current setup.</p>
<br>
<p align="center">I used the ip command to check the virtual network interface, eth_srv, which the DNS and DHCP server will listen on. The interface is configured to have the ip address 192.168.1.1 and the netmask is 255.255.255.0 as indicated by the /24 after the ip address. There is also a virtual network interface configured to simulate a client talking to the server, eth_cli, and by using the same command, ip, I see that the client interface does not have an ip address yet.<br/>
  <img src="https://github.com/user-attachments/assets/8f90834f-e302-44cd-8c94-305bc7831da5" height="80%" width="80%" alt="a linux terminal window with black background and white text. The top command reads ip address show eth_srv and the response over the next 5 lines shows information about the network interface, including the ip address. The next command reads ip address show eth_cli and the following 3 lines show information about the client interface, including a mac address but no IPv4 address."/>
  <br />
  <br />
Next, I used the cat command to print the text of the current dnsmasq configuration file, located at /etc/dnsmasq.d/mycompany.conf, to the terminal so that I could learn what the current settings were. I can see that the interface is defined as eht_srv, and that the bind-interface setting means that dnsmasq will only operate on that interface and ignore any others. I can see that the domain for the network is mycompany.local and that there's additional information in the dhcp-option settings providing clients with the router and the dns server. Lastly, I can see the current range that is being dynamically configured by dhcp as well as the lease time, which is currenlty set to 24 hours.<br />
  <img src="https://github.com/user-attachments/assets/8b29a386-7595-42ce-8430-32bc0e113f77" height="80%" width="80%" alt="a linux terminal window with black background and white text. The text reads: cat /etc/dnsmasq.d/mycompany.conf This is the interface on which the DCHP server will be listening to. Interface equals eth underscore srv. This tells this dnsmasq to only operate on that interface and not operate on any other interfaces, so that it doesn't interfere with other running dnsmasq processes. bind hyphen interfaces. Domain name that will be sent to the DHCP clients. domain equals my company dot local. Default gateway that will be sent to the DHCP clients. dhcp hyphen option equals option colon router comma 192 dot 168 dot 1 dot 1. DNS servers to announce to the DHCP clients. dhcp hyphen option equals option colon dns hyphen server comma 192 dot 168 dot 1 dot 1. Dynamic range of IPs to use for DHCP and the lease time. dhcp hyphen range equals 192 dot 168 dot 1 dot 2 comma 192 dot 168 dot 1 dot 254 comma 24 h."/>
</p>
<br />
<br />
- <b>Enabling Debug Logging</b>
<p>In order to better understand what is going on and why as I make changes to the configuration, I enabled debug logging.</p>
<br>
<p align="center">First, I queried the status of the dnsmasq service using the command sudo service dnsmasq status. I can see that the service is currently running.<br/>
  <img src="https://github.com/user-attachments/assets/6209e2e9-4362-42b3-92b2-7cc7065d41dc" height="80%" width="80%" alt="a linux terminal window with black background and white text. The first line has the command sudo service dnsmasq status. The response reads checking DNS forwarder and DHCP server colon dnsmasq running. Running is in parentheses."/>
  <br />
  <br />
Then, I opened the configuration file for dnsmasq in the nano text editor using the command sudo nano followed by the file path for the configuration file. I edited the file to add the option to log queries, and directed the service where to store the log file.<br />
  <img src="https://github.com/user-attachments/assets/0f177d52-74c2-4a93-94f9-d79b1d1bef78" height="80%" width="80%" alt="A nano text editor window with black background and white text. The text of the configuration file is the same as was described in the network set up and current configuration section with the exception of the last two lines which read: log hyphen queries and log hyphen facility equals /var/log/dnsmasq.log"/>
  <br />
  <br />
 Finally, I tested that the syntax of the configuration file was correct, using the --test parameter, and after confirming that it was OK, I started the dnsmasq daemon so that it would read the configuration file again.<br />
  <img src="https://github.com/user-attachments/assets/5cee2a53-6e52-4dd9-a532-40b46902c6d1" height="80%" width="80%" alt="a linux terminal with black background and white text. The first command reads sudo dnsmasq double hyphen test hyphen C /etc/dnsmasq.d/mycompany.conf and the response reads dnsmasq colon syntax check OK. The second command reads sudo service dnsmasq start and the response reads starting DNS forwarder and DHCP server colon dnsmasq."/>
</p>
- <b>Experimenting with DNS queries</b>
<p>I observed how dnsmasq handles DNS queries by using the dig command to request the IP address of certain hostnames.</p>
<br>
<p align="center">First, I queried dnsmasq for the IP address of example.com and specified that I want to use the running machine as the DNS server by using the @localhost parameter. In the answer section I actually got 6 different responses for A records for the domain name.<br/>
  <img src="https://github.com/user-attachments/assets/22116447-cdf4-4ace-be23-21a5ec0dd23c" height="80%" width="80%" alt="a linux terminal with black background and white text. The command in the top line reads dig example dot com at localhost. The response shows no errors, one query with six answers and then in the bottom answer section lists the A records for example dot com which has 6 IPv4 addresses."/>
  <br />
  <br />
 Then I used the debug logs from dnsmasq to see how dnsmasq handled the query. I used the tail command to print the last few lines of the debug log file to the terminal, and I can see that the local host checked its host file at /etc/hosts for the IP address for example.com and didn't find anything, so it forwarded the request to an external DNS server and received the responses listed in the dnsmasq response.<br />
  <img src="https://github.com/user-attachments/assets/f18a5689-aeb8-445c-9a67-7d5424ee53df" height="80%" width="80%" alt="a linux terminal with black background and white text. The command on the first line reads sudo tail /var/log/dnsmasq.log followed by the last 10 lines of the log file. The log entries have the date, May 2, and time, 18:15:18 to 18:16:52, on the left hand side, then the process, dnsmasq, and the process id, 1270, followed by a colon and then the responses. From top to bottom the responses read using nameserver 169 dot 254 dot 169 dot 254 # 53, read /etc/hosts hyphen 7 addresses, query A example dot com from 127 dot 0 dot 0 dot 1, forwarded example dot com to 169 dot 254 dot 169 dot 254, reply example dot com is 23 dot 192 dot 228 dot 80, reply example dot com is 23 dot 192 dot 228 dot 84, reply example dot com is 23 dot 215 dot 0 dot 136, reply example dot com is 23 dot 215 dot 0 dot 138, reply example dot com is 96 dot 7 dot 128 dot 175, reply example dot com is 96 dot 7 dot 128 dot 198"/>
  <br />
  <br />
  Step Three: <br />
  <img src="" height="80%" width="80%" alt="image three"/>
   <br />
  <br />
  Step Four: <br />
  <img src="" height="80%" width="80%" alt="image four"/>
   <br />
  <br />
  Step Five: <br />
  <img src="" height="80%" width="80%" alt="image five"/>
</p>

