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
  Step Two: <br />
  <img src="" height="80%" width="80%" alt="image two"/>
  <br />
  <br />
  Step Three: <br />
  <img src="" height="80%" width="80%" alt="image three"/>
   <br />
  <br />
  Step Four: <br />
  <img src="" height="80%" width="80%" alt="image four"/>
</p>
<br />
<br />
- <b>Section Name</b>
<p>Description</p>
<br>
<p align="center">Step One: <br/>
  <img src="" height="80%" width="80%" alt="image one"/>
  <br />
  <br />
  Step Two: <br />
  <img src="" height="80%" width="80%" alt="image two"/>
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

