
<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Observing Network Traffic and Network Security Group Effects</h1>
  
<p>In this walkthrough, we will observe various network traffic to and from Azure Virtual Machines using PowerShell and Wireshark as well as see how traffic is affected with the use of Network Security Groups.</p>


<h2>Pre-requisite: Creating Azure Virtual Machines</h2>

- ### https://github.com/anakamura1/VM-creation

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop (Windows) / Windows App (Mac)
- PowerShell
- Various Network Protocols (ICMP, SSH, DHCP, DNS, RDP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (21H2)
- Ubuntu Server 20.04

<h2>High-Level Steps</h2>

- Step 1: Observe ICMP Traffic
- Step 2: Use NSG (Firewall) to Block Ping
- Step 3: SSH and DHCP Traffic
- Step 4: DNS and RDP Traffic

<h2>Actions and Observations</h2>
<h3>Step 1: Observe ICMP Traffic</h3>

<table>
  <tr>
    <td style="width: 67%;"><img width="100%" alt="Screenshot 2025-01-08 at 3 20 21 PM" src="https://github.com/user-attachments/assets/c653d1d7-37ca-446a-9b11-4b6bbf912d1d" /></td>
    <td style="width: 26%;"><img width="100%" alt="Screenshot 2025-01-08 at 3 23 08 PM" src="https://github.com/user-attachments/assets/f789a2c3-cc16-4a26-a3f1-1ab061ed8b3e" /></td>
  </tr>
</table>

<p>In Azure, click into the Windows VM and copy the public IP address of the Windows VM and add a new PC in Remote Desktop or Windows App. This will allow us to log in to the Windows VM.</p>
<br /><br>

<table>
  <tr>
    <td style="width: 62%;"><img width="100%" alt="Screenshot 2025-01-08 at 3 38 52 PM" src="https://github.com/user-attachments/assets/ef7ece98-6bd2-4f65-8a7d-f413cb0df0cb" /></td>
    <td style="width: 31%;"><img width="100%" alt="Screenshot 2025-01-08 at 3 39 23 PM" src="https://github.com/user-attachments/assets/3f96cb86-ac44-4764-83df-455c3c79bede" /></td>
  </tr>
</table>

<p>After logging into the Windows VM with the username and password we assigned it at creation, go to the browser to install Wireshark. Be sure to download the Windows x64 version and run the Setup installer to finish installing Wireshark.</p>
<br /><br>

<table>
  <tr>
    <td style="width: 45%;"><img width="100%" alt="Screenshot 2025-01-08 at 3 45 54 PM" src="https://github.com/user-attachments/assets/e1541369-cf1d-4ff4-89ac-44c3b1662fe1" /></td>
    <td style="width: 45%;"><img width="100%" alt="Screenshot 2025-01-08 at 3 48 52 PM" src="https://github.com/user-attachments/assets/7c3159f6-6aee-4ef2-a878-0d305ae4e0a9" /></td>
  </tr>
</table>

<p>Once Wireshark has been installed, open it and begin observing the packet capture by clicking on "Ethernet". In the top search bar, type in icmp and hit enter to begin filtering for ICMP traffic. ICMP traffic is the network protocol that ping uses. A ping is simply a connection test between two devices, in our case it is between our Windows VM and our Linux VM. It sends a small packet of data over the internet to see if we can establish a connection.</p>
<br /> <br>

<p><img width="60%%" alt="Screenshot 2025-01-08 at 3 55 45 PM" src="https://github.com/user-attachments/assets/8584006e-756a-4945-851b-de7b665b772f" /></p>

<p>Next, go back to the Linx VM we created in Azure and find the PRIVATE IP address within the configurations. Open up PowerShell and type in "ping 10.0.0.5". Since 10.0.0.5 is the private IP address of our Linux VM, we will be using it to ping our Linux VM from our Windows VM. Observe that after we hit enter in PowerShell, Wireshark shows us the network traffic that occurred when we pinged our Linux VM. We can see from the traffic that appears a "request" from our Windows VM and "reply" from the Linux VM occurs. Hooray for network traffic!</p>
<br>

<p><strong>BONUS:</strong> From PowerShell, ping a public website like Google.com. Enter ping google.com and observe the network traffic!</p>
<br>
<h3> Step 2: Use NSG (Firewall) to Block Ping</h3>

<table>
  <tr>
    <td style="width: 100%;"><img width="100%" alt="Screenshot 2025-01-08 at 5 14 28 PM" src="https://github.com/user-attachments/assets/7e823b95-f8e3-4341-a430-f5989846b48b" /></td>
    <td style="width: 100%;"><img width="100%" alt="Screenshot 2025-01-08 at 5 04 26 PM" src="https://github.com/user-attachments/assets/5f5d2342-47eb-40b8-af44-64d167daf97d" /></td>
  </tr>
</table>

<p>In Powershell, initiate a perpetual ping by entering "ping 10.0.0.5 -t". Next, go to the Linux VM Network Settings and click "Create port rule". Select "Inbound Port Rule".</p>
<br><br>

<p><img width="40% height="40%" alt="Screenshot 2025-01-08 at 5 08 04 PM" src="https://github.com/user-attachments/assets/ef7c84b4-9953-44d9-b670-a7d7a3a5b583" /></p>
 

<p>For the Inbound Port Rule settings, be sure to select "ICMPv4", select "Deny", and put 290 for the priority. ICMPv4 is the protocol for ping traffic, so we need to select this for our Linux VM to deny inbound ping. 290 priority will in short cause the Linux VM to prioritize it over the other protocols that have a higher number value.</p>
<br><br>

<p> <img width="100%" alt="Screenshot 2025-01-08 at 5 18 18 PM" src="https://github.com/user-attachments/assets/0a87618e-3729-4ba1-814e-4bdfda56c216"/> </p>
 

<p>Once you add the rule, give it few seconds to deploy and observe the ping timeout in Powershell. Notice that the bottom number of requests in Wireshark do not show a reply. This means that our Windows VM is sending a ping (request) but the NSG rule is blocking it, causing a timeout, resulting in no reply from our Linux VM. The timeout shows us that our NSG rule was successful in blocking inbound ping!</p>
<br><br>

<table>
  <tr>
    <td style="width: 100%;"><img width="100%" alt="Screenshot 2025-01-08 at 5 25 02 PM" src="https://github.com/user-attachments/assets/0be33a90-ce04-4f54-9001-71a39be45610" /></td>
  </tr>
</table>

<p>Go back into Network Settings in Azure and delete the security rule we just made. Then after a few seconds, observe the ping traffic resume in PowerShell/Wireshark. Finally, to stop the perpetual ping, enter "Control C".</p>
<br>

<h3>Step 3: SSH and DHCP Traffic </h3>


  <p> <img width="60%" alt="Screenshot 2025-01-08 at 5 34 14 PM" src="https://github.com/user-attachments/assets/3d4a3010-ddf3-4de9-bdb7-e39ee7e20586"/> </p>


<p>Continuing on to observe Secure Shell (SSH) traffic, let's start a new packet capture by pressing the green button at the top and pressing "continue without saving". In short, SSH (Secure Shell) is a protocol used to securely access and manage remote computers over a network.</p>

<p>In the top search bar, search for ssh to begin filtering for ssh traffic.</p>
<br><br>

<table>
  <tr>
    <td style="width: 100%;"><img width="100%" alt="Screenshot 2025-01-08 at 5 42 25 PM" src="https://github.com/user-attachments/assets/784fd256-f383-4005-a389-2270fed33f69" /></td>
  </tr>
</table>

<p>We can then enter "ssh username@IPaddress" into PowerShell to SSH into our Linux VM. The username is whatever username was given to the Linux VM when we created it, in this case it is anakamura1. The IP address is the private IP address of the Linux VM which in this case is 10.0.0.5.</p>

<p>After pressing enter, enter "yes", and type in the password for the LinuxVM. No text will appear as we type the password but simply hit enter after typing the password and we will successfully SSH into the Linux VM. Notice in PowerShell we are now "anakamura1@LinuxVM". This shows us that we now have secure access and can manage the LinuxVM over the SSH network protocol.</p>

<p><strong>BONUS:</strong> Press any key and notice in Wireshark traffic generated by each key stroke. How cool! When finished, simply enter "exit" to exit SSH.</p>
<br>

<h3>DHCP</h3>

<table>
  <tr>
    <td style="width: 47%;"><img width="100%" alt="Screenshot 2025-01-08 at 6 20 59 PM" src="https://github.com/user-attachments/assets/fe7f0481-8d4e-47a9-bce8-748ed563fa3c" /></td>
    <td style="width: 41%;"><img width="100%" alt="Screenshot 2025-01-08 at 6 44 16 PM" src="https://github.com/user-attachments/assets/5aba0186-b82c-4532-a98a-76c503775eef" /></td>
  </tr>
</table>

<p>Now in Wireshark lets go to the search bar and search for "udp.port == 67 || udp.port == 68". This will filter DHCP traffic for us. DHCP (Dynamic Host Configuration Protocol) automatically assigns IP addresses to devices on a network.</p>

<p>We will "release" our VM's IP address and request a new one from the DHCP.</p>

<p>In order to do this, lets create a batch file by opening up Notepad and typing "ipconfig /release" and "ipconfig /renew". This is a script that we will run from PowerShell that will command our VM to release its IP address and receive a new one from the DHCP.</p>

<p>On Notepad, click File -> Save As. Be sure to save it in the C drive. We can check this by typing "C:\" into the top bar. We want to name the file dhcp.bat and change "Save as type" to "All Files".</p>


  <p><img width="50%" alt="Screenshot 2025-01-08 at 6 47 33 PM" src="https://github.com/user-attachments/assets/02ef7a08-82df-4074-805b-13d45b3e7926" /></p>


<p>Let's verify in Powershell that our script was saved. Enter "cd c:\" and then enter "ls". Notice that we have the dhcp.bat file saved.</p>

<p>Enter ".\dhcp.bat" to run the script and observe the traffic in Wireshark.</p>
<br><br>

<table>
  <tr>
    <td style="width: 100%;"><img width="100%" alt="Screenshot 2025-01-08 at 6 53 30 PM" src="https://github.com/user-attachments/assets/3e4dc478-8560-42ae-b8bf-b6998253d7e8" /></td>
  </tr>
</table>

<p>The screen will freeze as we will momentarily drop connection to our VM but afterwards we see in Wireshark the release of our IP address and the receiving of a new address. The "Release Discover Offer Request Acknowledge" tells us that our VM released the IP address, sent out a discovery call to the DHCP server for an IP address, it received an offer from the server, requested the IP address, and the DHCP server acknowledged the request by assigning us the IP address.</p>
<br><br>

<h3>Step 4: DNS and RDP Traffic</h3>

<table>
  <tr>
    <td style="width: 100%;"><img width="100%" alt="Screenshot 2025-01-08 at 7 02 59 PM" src="https://github.com/user-attachments/assets/0ae49aa8-4b87-4f57-9d1f-028585fa011c" /></td>
  </tr>
</table>

<p>DNS (Domain Name System) translates domain names (like www.google.com) into IP addresses that computers can understand. We can observe DNS traffic by seeing how our VM asks the DNS server to resolve "google.com" into an IP address.</p>

<p>In Wireshark filter for "dns" and enter "nslookup google.com" into PowerShell. This will generate traffic for us to observe.</p>

<p>On Wireshark we can see traffic from us (10.0.0.4) to the DNS server (168.63.129.16) and on the PowerShell side we see the various IP addresses given to us by the DNS server for google.com.</p>
<br>

<h3>RDP</h3>

<table>
  <tr>
    <td style="width: 100%;"><img width="100%" alt="Screenshot 2025-01-08 at 7 12 25 PM" src="https://github.com/user-attachments/assets/e99cc2b5-9c2b-4133-95f9-dc16bed6cf26" /></td>
  </tr>
</table>

<p>Remote Desktop Traffic can be observed in Wireshark by filtering for "tcp.port == 3389". We will notice that this is a lot of traffic being generated and this is because by using a VM we are using an RDP. All of this traffic represents the VM sending our physical computer each image and activity going on. The RDP is constantly streaming a picture to us as we use the VM.</p>

**If you are done with using the virtual machines, please remember to stop the VM's or delete the resource groups in Azure to prevent unnecessarily spending funds.**

