<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Tutorial: Using Azure to Configure Network Security Groups (NSGs) and Inspect Traffic Between Virtual Machines</h1>
In this tutorial we'll create and deploy two Windows Virtual Machines, use Wireshark to observe TCP/IP traffic between them, and experiment with Network Security Groups to block inbound ICMP traffic. <br />

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
  
 <h2>Setting up VMs in Azure</h2>
  <h3>Before we get started, let’s clarify what our objectives are. In this lab we’ll create two virtual machines (or VMs), then we’ll try to get them to talk to each other within the same virtual network or VNet. To accomplish this, we first need to create a resource group. This resource group will be a home for both our VMs, their components, and the virtual network that will tie them all together.</h3>
  <img src="https://github.com/amaraphi/azure-network-protocols/assets/144752187/da3daa8d-c707-46d0-8ee5-58d15d28a373"/>
  <h3>Start by setting up your resource group</h3>
  <ul>
    <li>On the Azure portal homepage, click "Resource Groups." Alternatively, you can use the search bar to search for resource groups.</li>
  </ul>
  <ul>
    <li>Click "create" to create a new resource group.</li>
  </ul>
  <ul>
    <li>Name this resource group "RG-1," which will be contained inside of your current Azure subscription.</li>
  </ul>
  <img src="https://github.com/amaraphi/azure-network-protocols/assets/144752187/a8426ecf-ee78-4099-b87b-9714d31c1dc9"/>
  <h3><strong>Next, let's create our first virtual machine.</strong></h3>
  <ul>
    <li>Create a VM with a Windows 10 OS: In the Azure portal search for "virtual machines."</li>
  </ul>
  <ul>
    <li>Once in the virtual machines default directory, click "Create --&gt; Azure Virtual Machine."</li>
  </ul>
  <ul>
    <li>Select the resource group that you’ve just created, RG-1. (Important: remember that our virtual network environment and both VMs should be within this resource group.)</li>
  </ul>
  <ul>
    <li>Scrolling down to "Instance Details," select your appropriate region. Keep note of the region selected -- both your VMs will need to be within the same region.</li>
  </ul>
  <ul>
    <li>Though optional, you can select a redundancy option to protect against data loss.</li>
  </ul>
  <img src="https://github.com/amaraphi/azure-network-protocols/assets/144752187/d8c54a26-664a-4947-bdbf-14bc0066f128"/>
  <ul>
    <li>Under "Image," select "Windows 10 Pro."</li>
  </ul>
  <ul>
    <li>Under "Size," select a Standard machine with 2-4 vCPUs (virtual CPUs).</li>
  </ul>
  <img src="https://github.com/amaraphi/azure-network-protocols/assets/144752187/9beb90ea-d35c-4417-a552-70271bf7b0cf"/>
  <ul>
    <li>Under "Administrator Account" create a username and password. For the sake of this lab we'll have a generic username, lab_user. This username and password will be used to remotely log in to our VM.
      <ul>
        <li>Note that under "Inbound Port Rules,” only inbound RDP traffic is accessible to our VM from the internet.</li>
      </ul>
    </li>
  </ul>
  <ul>
    <li>Click "Review and Create."</li>
  </ul>
  <ul>
    <li>If we scroll down to "Networking” on the Validation page, we can observe that the following network resources have been automatically generated for our VM:
      <ul>
        <li>virtual network and subnet</li>
      </ul>
      <ul>
        <li>public IP address</li>
      </ul>
    </li>
  </ul>
  <img src="https://github.com/amaraphi/azure-network-protocols/assets/144752187/7a6544c0-92fd-4d9a-98d8-5f5c32f5fa07"/>
  <ul>
    <li>Click “Create.”</li>
  </ul>
  <h3>Create another virtual machine, this time with a Linux Ubuntu Server.</h3>
  <ul>
    <li>Repeating the same steps above, create another VM, this time with the name “VM-2.”</li>
  </ul>
  <ul>
    <li>Again, this VM should be inside of the same resource group, RG-1, as your Windows 10 VM.</li>
  </ul>
  <ul>
    <li>Select the same region as VM-1.</li>
  </ul>
  <ul>
    <li>Under “Image,” select “Ubuntu Server.”</li>
  </ul>
  <ul>
    <li>Select "Password" under "Authentication Type." This will prompt us to create a username and password that we’ll use to connect to the Ubuntu server in a later tutorial.</li>
  </ul>
  <img src="https://github.com/amaraphi/azure-network-protocols/assets/144752187/42eb0a12-78e9-4a4a-8306-438de3e8ce3d"/>
  <ul>
    <li>Observe that under "inbound port rules," our newly-created VM's SSH port 22 will be open and accessible from the Internet.</li>
  </ul>
  <ul>
    <li>Click "Review and Create."
      <ul>
        <li>Observe that our Ubuntu VM, VM-2, has been automatically placed within VM-1's virtual network.</li>
      </ul>
      <ul>
        <li>VM-2 has also been assigned its own public IP address. It is also part of a 10.0.0.0/24 subnet, identical to VM-1's. This means that VM-2's private IP address will be located within the same subnet, enabling both machines to communicate and exchange data over the private network.</li>
      </ul>
    </li>
  </ul>
 <img src="https://github.com/amaraphi/azure-network-protocols/assets/144752187/d2281b17-bc67-44d4-9199-54ec3c34f6db"/>
  <h3>Now that we have created both virtual machines, we can see all of our newly generated resources in the RG-1 resource group.</h3>
  <ul>
    <li>On the Azure home portal, click "Resource Groups" and open "RG-1."</li>
  </ul>
  <ul>
    <li>For both VM-1 and VM-2 we can see one of the following:
      <ul>
        <li>a virtual machine</li>
      </ul>
      <ul>
        <li>a network interface card, or NIC</li>
      </ul>
      <ul>
        <li>a virtual disk or hard drive</li>
      </ul>
      <ul>
        <li>one public IP address</li>
      </ul>
      <ul>
        <li>one Network Security Group</li>
      </ul>
    </li>
  </ul>
  <p><strong>Important: There should only be one virtual network or VNet, titled "VM-1"!</strong></p>
  <img src="https://github.com/amaraphi/azure-network-protocols/assets/144752187/13256a3a-6e39-4f17-84e1-87d2a2692360"/>
  <h3><strong>Alternatively, we can take a look at our network topology with Azure’s Network Watcher.</strong></h3>
  <ul>
    <li>Search for "Network Watcher" in the search bar and click on the first result.</li>
  </ul>
  <ul>
    <li>On the left, select "Topology."</li>
  </ul>
  <ul>
    <li>Here we can see the layout of all of our network components which include one VNet, one subnet, and our two virtual machines and their resources.</li>
  </ul>
  <img src="https://github.com/amaraphi/azure-network-protocols/assets/144752187/d28abb17-5a6b-4639-b78b-a2a6a6fb7dfb"/>
  <h3>Testing Connectivity Between Virtual Machines</h3>
  <ul>
    <li>Using Windows Remote Desktop, connect to the Windows 10 Virtual Machine</li>
  </ul>
  <ul>
    <li>Open the Windows command prompt and initiate a <code>ping</code> to the Ubuntu Virtual Machine</li>
  </ul>
  <ul>
    <li>Observe echo request and reply packets in the command line.</li>
  </ul>
  <ul>
    <li>On Wireshark, filter for ICMP traffic and observe requests and replies from VM-1 (Windows 10) and VM-2 (Ubuntu).</li>
  </ul>
  <ul>
    <li>Initiate a perpetual ping between VM1 and VM2 with <code>ping -t <ip_address></code></li>
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
    <li>Return to VM-1 via Remote Desktop and observe the resumed flow of ICMP traffic between both virtual machines.</li>
  </ul>
  <h3>Observing SSH Traffic</h3>
  <ul>
    <li>Return to VM-1 on Remote Desktop and open the command prompt. Initiate an SSH session with VM-2 by entering the following command. Remember to use VM-2’s private IP address!
      <ul>
        <li><code> ssh username@ip_address</code></li>
      </ul>
    </li>
  </ul>
  <ul>
    <li>You’ll be prompted to enter the password that was created during initial setup of the Ubuntu VM.</li>
  </ul>
  <ul>
    <li>After connecting to VM’s server, experiment by entering simple Linux commands (e.g.,<code>pwd</code>, <code>mkdir</code>, <code>rmdir</code>).</li>
  </ul>
  <ul>
    <li>On Wireshark, filter for SSH activity and observe flow of SSH traffic between the virtual machines.</li> 
  </ul>
  <h3>Observing DHCP Traffic</h3>
  <ul>
    <li>In the Windows 10 VM’s command line, use <code>ipconfig /renew</code> to attempt to issue the VM a new IP address</li>
  </ul>
  <ul>
    <li>On Wireshark, filter for DHCP and observe the traffic between our virtual network’s DHCP server and VM-1.</li>
  </ul>
  <h3>Observing DNS traffic</h3>
  <ul>
    <li>In VM-1’s command line, use the <code>nslookup</code> command to look up the IP address of a well-known website. In this case, I’ve looked up google.com.</li>
  </ul>
  <ul>
    <li>We can see here that the DNS server has returned IPv6 and IPv4 addresses. We can force the DNS server to return only Google’s associated IPv4 addresses by querying its A records:
      <ul>
        <li><code>nslookup -q=A google.com</code></li>
      </ul>
    </li>
  </ul>
  <ul>
    <li>Return to Wireshark and observe the flow of traffic between our network’s DNS server, VM-1, and Amazon.</li>
  </ul>
</body>
</html>
