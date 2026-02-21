<h1>Static IP Configuration & Internal Network Validation (Hyper-V Lab)</h1>

## Lab Objective
In this lab, I set up the lab topography, configured static IPv4 addressing between a Windows 11 VM , a Kali Linux VM, a Ubuntu Server VM and a Windows Server VM using a Hyper-V Internal Virtual Switch, then validated Layer 3 connectivity (ping) and documented Windows Firewall behavior on a “Public” network profile. The goal was to establish a controlled, isolated lab network that I can reuse for future Active Directory and attack simulation exercises.



## Skills Demonstrated
### Infrastructure & Virtualization
- Designed and deployed a multi-VM enterprise lab using Microsoft Hyper-V (Generation 2)
- Implemented a fully isolated Internal Virtual Switch (Layer 2, non-routed architecture)
- Configured resource allocation (memory, VHDX, network adapters)
- Built repeatable lab topology for enterprise simulation and detection testing

### Network Architecture & IPv4 Configuration
- Designed and implemented private subnet: 192.168.100.0/24
- Configured static IPv4 addressing across multiple VMs
- Applied CIDR notation (/24) and subnet mask logic (255.255.255.0)
- Validated Layer 3 connectivity via ICMP testing
- Troubleshot Linux interface state using ip link, ip a, and nmcli
- Configured Netplan for persistent static IP deployment




## Network Architecture
Hyper-V Internal Switch (Layer 2 only)
### Lab Topology
Subnet: 192.168.100.0/24
| System | Role | IP | 
|-------|----------|------------|
| Windows Server | 2022	Domain Controller + DNS	| 192.168.100.5 |
| Ubuntu 24.04 |	Splunk Server |	192.168.100.30 |
| Windows 11 |	Domain Client |	192.168.100.10 |
| Kali Linux |	Attack Simulation |	192.168.100.20 |

<br />
<br />

<img width="591" height="601" alt="image" src="https://github.com/user-attachments/assets/bce85353-5b9a-47bf-91c3-a35968569823" />

<br />
<br />

Step 1 — Configure Static IP on Windows 11
Path:
Control Panel → Network and Sharing Center →
Change Adapter Settings → Ethernet → Properties →
Internet Protocol Version 4 (TCP/IPv4)
 
________________________________________
Step 2 — Configure Static IP on Kali Linux
Identify interface:
ip a
Bind Network Manager profile to eth0:
sudo nmcli con mod "Wired connection 1" connection.interface-name eth0
Set static IP:
sudo nmcli con mod "Wired connection 1" ipv4.method manual ipv4.addresses 192.168.100.20/24
sudo nmcli con up "Wired connection 1"
Verify:
ip a
Expected:
inet 192.168.100.20/24
 
________________________________________
Step 3 — Validate Connectivity
From Windows → Kali
ping 192.168.100.20
Expected: Replies received.

 ________________________________________
Step 3.5 — Enable ICMP (Ping) in Windows Firewall
Because the Hyper-V Internal switch is categorized as Public, Windows blocks inbound ICMP by default. So initial ping from Kali to windows was not received. In order to troubleshoot the issue I needed to manage the firewall rules.
Open:
wf.msc
Create New Inbound Rule:
•	Rule Type: Custom
•	Protocol: ICMPv4
•	Specific ICMP types: Echo Request
•	Action: Allow
•	Profile: Public
  

________________________________________
Now pinging From Kali → Windows
ping 192.168.100.10
Expected: Replies received.


 



Windows Server 2022 Deployment (Active Directory)
VM Creation (Hyper-V)
•	Generation: 2
•	Memory: 4096 MB
•	Network: Lab_internal
•	VHDX: Dynamically expanding
•	ISO: Windows Server 2022 Evaluation
After installation, the server was assigned a static IP.

 
________________________________________
Static IPv4 Configuration (Domain Controller)
Setting	Value
IP Address	192.168.100.5
Subnet Mask	255.255.255.0
Default Gateway	(Blank – internal only)
DNS Server	192.168.100.5 (Self)
This configuration is critical because:
•	A Domain Controller must point to itself for DNS.
•	Active Directory relies entirely on DNS resolution.
Verified with:
Ipconfig

 
________________________________________

2️⃣ Ubuntu 24.04 Splunk Server Deployment
VM Creation
•	OS: Ubuntu Server 24.04 LTS
•	Network: Lab_internal
•	Memory: 4 GB
•	Static IP assigned post-install
________________________________________
Failed Ubuntu IP a 
 

Initial Network Configuration 
Entered: “ sudo ip link set eth0 up ”  since the Ubuntu VM network interface is not active.
Then Configuring netplan file to set up static IPs
Edited:
/etc/netplan/*.yaml
Configuration:
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.100.30/24
      nameservers:
        addresses:
          - 192.168.100.5

 

Applied with:
sudo chmod 600 /etc/netplan/*.yaml
sudo netplan apply
Verified:
ip a

 
This ensures:
•	Static addressing
•	DNS resolution via Domain Controller
•	No dependency on DHCP
________________________________________

3️⃣ Network Validation Across Infrastructure
After configuring:
•	Windows 11 → 192.168.100.10
•	Kali → 192.168.100.20
•	AD Server → 192.168.100.5
•	Splunk Server → 192.168.100.30
I validated full subnet communication.
Tested:
•	Windows → Kali
•	Kali → Windows
•	Splunk → AD
•	AD → Splunk
All nodes responded to ICMP Echo after firewall rule configuration. 
 
This confirms:
•	Layer 2 switching via Hyper-V Internal switch
•	Layer 3 routing within subnet
•	No subnet misconfiguration
•	No gateway requirement (isolated lab)
________________________________________













________________________________________
Technical Concepts Reinforced
•	Private IPv4 addressing (RFC 1918)
•	CIDR notation (/24)
•	Subnet mask interpretation (255.255.255.0)
•	Layer 2 vs Layer 3 communication
•	Broadcast domain boundaries
•	Windows Firewall profile behavior (Public vs Private)
•	ICMP filtering and inbound rule creation
•	Hyper-V Internal Switch behavior (non-routed)
________________________________________
Result
All VMs successfully communicate over a manually configured static subnet:
192.168.100.0/24
This establishes a stable foundation for:
•	Active Directory deployment
•	Lateral movement simulation
•	SIEM log generation
•	Controlled attack surface testing



Why Windows Server? 
install Active Directory (AD) on a Windows Server for production environments, especially if managing Windows workstations, due to its native compatibility, ease of use, and full support for Group Policies. 

Why Ubuntu server? 
install Splunk on an Ubuntu server (Linux) due to its superior performance, lower resource usage, and better support for advanced features like SmartStore. Linux is generally preferred for indexing, searching, and overall system stability.



## Architectural Decisions: 
| Decision                       | Rational                   
| ------------------------------ | -----------------------------------------------------| 
| **Internal Switch**            | Protect home network from attack traffic             | 
| **8GB RAM**                    | RAM	Supports offensive tools without starving host  | 
| **Kali Linux**                 | Go to OS for all cyber needs                         | 
| **NVMe storage**               | High I/O performance during scans                    | 
| **SHA256 verification**        | Supply chain integrity validation                    | 

## Security Considerations
-  Kali connected to Internal switch by default
-  External access used only when required
-  No attack traffic exposed to production/home LAN
-  Hash validation performed prior to deployment
