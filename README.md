<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Tutorial: Using Azure to Configure Network Security Groups (NSGs) and Inspect Traffic Between Virtual Machines</h1>
In this tutorial, we'll use Wireshark to observe traffic between Azure Virtual Machines and experiment with Network Security Groups. <br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- Various Network Protocols (SSH, RDH, DNS, HTTP/S, ICMP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (21H2)
- Ubuntu Server 20.04

<h2>High-Level Steps</h2>

- Connect to a Windows 10 Virtual Machine using Remote Desktop
- Observe ICMP traffic
- Observe SSH Traffic
- Observe DHCP Traffic
- Observe DNS Traffic
- Observe RDP Traffic
  
<body>
  <h3>Testing Connectivity Between Virtual Machines</h3>
  <ul>
    <li>Using Windows Remote Desktop, connect to the Windows 10 Virtual Machine</li>
  </ul>
  <ul>
    <li>Open the Windows command prompt and initiate a ping to the Ubuntu Virtual Machine</li>
  </ul>
  <ul>
    <li>Observe echo request and reply packets in the command line.</li>
  </ul>
  <ul>
    <li>On Wireshark, filter for ICMP traffic and observe requests and replies from VM1 (Windows 10) and VM 2 (Ubuntu).</li>
  </ul>
  <ul>
    <li>Initiate a perpetual ping between VM1 and VM2 with -t</li>
  </ul>
  <h3>Implementing and Testing and Inbound Security Rules: Blocking Inbound ICMP Traffic</h3>
  <ul>
    <li>Navigate to Network Security Groups in the Azure portal.</li>
  </ul>
  <ul>
    <li>Open the Network Security Group for VM-2, the Ubuntu Virtual Machine.</li>
  </ul>
  <ul>
    <li>Under “Inbound Security Rules,” add a new inbound security rule with the following fields filled out:
      <ul>
        <li>Source —&gt; Any</li>
      </ul>
      <ul>
        <li>Destination —&gt; Any</li>
      </ul>
      <ul>
        <li>Protocol —&gt; ICMP</li>
      </ul>
      <ul>
        <li>Action — Deny</li>
      </ul>
      <ul>
        <li>Priority —&gt; 200 (Prioritized before all other inbound security rules to screen all incoming traffic)</li>
      </ul>
    </li>
  </ul>
  <ul>
    <li>Back in VM-1, observe “request timed out” and “no response” messages from the command line and Wireshark, respectively.</li>
  </ul>
  <ul>
    <li>In the Azure portal, return to the Network Security Group for VM-2 and re-enabled inbound ICMP traffic by changing Action in security rule to “Allow.”</li>
  </ul>
  <ul>
    <li>Returned VM1 and observed resumed flow of ICMP traffic between both virtual machines.</li>
  </ul>
  <h3>Observing SSH Traffic</h3>
  <ul>
    <li>Return to VM-1 on Remote Desktop and open the command prompt. Initiate an SSH session with VM-2 by entering the following command. Remember to use VM-2’s private IP address!
      <ul>
        <li>ssh &lt;username&gt;@&lt;ip_address&gt;</li>
      </ul>
    </li>
  </ul>
  <ul>
    <li>You’ll be prompted to enter the password that was created during initial setup of the Ubuntu VM.</li>
  </ul>
  <ul>
    <li>After connecting to VM’s server, experiment by entering simple Linux commands (pwd, <strong>mkdir, rmdir)</strong></li>
  </ul>
  <ul>
    <li>On Wireshark, filter for SSH activity and observe flow of SSH traffic between the virtual machines.</li>
  </ul>
  <h3>Observing DHCP Traffic</h3>
  <ul>
    <li>In the Windows 10 VM’s command line, use ipconfig /renew to attempt to issue the VM a new IP address</li>
  </ul>
  <ul>
    <li>On Wireshark, filter for DHCP and observe the traffic between our virtual network’s DHCP server and VM-1.</li>
  </ul>
  <h3>Observing DNS traffic</h3>
  <ul>
    <li>In VM-1’s command line, use the nslookup command to find the IP address of a well-known website. In this case, I’ve looked up www.amazon.com.</li>
  </ul>
  <ul>
    <li>We can see here that the DNS server has returned Ipv6 addresses. We can force the DNS server to return Amazon’s Ipv4 addresses only by querying its A records</li>
  </ul>
  <ul>
    <li>Return to Wireshark and observe the flow of traffic between our network’s DNS server, VM-1, and Amazon.</li>
  </ul>
</body>
</html>
