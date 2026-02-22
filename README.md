# Static IP Configuration & Internal Network Validation (Hyper-V Lab)

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

## Result
All VMs successfully communicate over a manually configured static subnet: 192.168.100.0/24 <br />
This establishes a stable foundation for:
- Active Directory deployment
- Lateral movement simulation
- SIEM log generation
- Controlled attack surface testing



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

## Step 1 — Configure Static IP on Windows 11
Path: <br />
Control Panel → Network and Sharing Center →
Change Adapter Settings → Ethernet → Properties →
Internet Protocol Version 4 (TCP/IPv4)
<br />
<br />

<img width="975" height="620" alt="image" src="https://github.com/user-attachments/assets/7fb72a78-cbde-422c-8b39-809d3fef278e" />

<br />
<br />

## Step 2 — Configure Static IP on Kali Linux
### Identify Kali interface: <br />
- ip a


### Bind Network Manager profile to eth0: <br />
- sudo nmcli con mod "Wired connection 1" connection.interface-name eth0


### Set static IP:<br />
- sudo nmcli con mod "Wired connection 1" ipv4.method manual ipv4.addresses 192.168.100.20/24
- sudo nmcli con up "Wired connection 1"

### Expected:<br />
- inet 192.168.100.20/24
<br />
<br />

<img width="973" height="788" alt="image" src="https://github.com/user-attachments/assets/8dc6e803-81fa-4e9e-a30e-0947dda72760" />

<br />
<br />

## Step 3 — Validate Connectivity
From Windows → Kali
ping 192.168.100.20
Expected: Replies received.
<br />
<br />
<img width="974" height="618" alt="image" src="https://github.com/user-attachments/assets/0666b0c1-2807-42a9-9c00-767f61084b89" />
<br />
<br />

## Step 3.5 — Enable ICMP (Ping) in Windows Firewall

Because the Hyper-V Internal switch is categorized as Public, Windows blocks inbound ICMP by default. So the initial ping from Kali to Windows was not received. In order to fix the issue, I needed to manage the firewall rules.

### Open: 
- wf.msc

### Create New Inbound Rule:
- Rule Type: Custom
- Protocol: ICMPv4
- Specific ICMP types: Echo Request
- Action: Allow
- Profile: Public

<img width="974" height="569" alt="image" src="https://github.com/user-attachments/assets/b9a0cb82-66ec-4bea-9c2b-3b47a4630053" />
<img width="974" height="594" alt="image" src="https://github.com/user-attachments/assets/1aa89fd7-e130-4a94-a629-910e32606b67" />


## Now pinging from Kali → Windows
- ping 192.168.100.10
- Expected: Replies received.
<br />
<br />


 <img width="974" height="371" alt="image" src="https://github.com/user-attachments/assets/8151a1c0-839d-443e-b73e-8e35af630679" />

<br />
<br />



## Step 4: Windows Server 2022 Deployment (Active Directory)
# VM Creation (Hyper-V)
- Generation: 2
- Memory: 4096 MB
- Network: Lab_internal
- VHDX: Dynamically expanding
- ISO: Windows Server 2022 Evaluation
After installation, the server was assigned a static IP.

## 4.5 Static IPv4 Configuration Windows Server (Domain Controller)
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
<br />
<br />
<img width="974" height="627" alt="image" src="https://github.com/user-attachments/assets/8300c1de-2191-493c-879a-5af42309ac23" />
<br />
<br />
 


## Step 5: Ubuntu 24.04 Splunk Server Deployment
VM Creation
- OS: Ubuntu Server 24.04 LTS
- Network: Lab_internal
- Memory: 4 GB
- Static IP assigned post-install


### Failed Ubuntu IP a 

<img width="975" height="832" alt="image" src="https://github.com/user-attachments/assets/b3d6acd9-65ec-4678-b172-15d5267d9e24" />

<br />
<br />
 
### Initial Network Configuration
Bringing the linux network online on network layer 2
sudo ip link set eth0 up ”  since the Ubuntu VM network interface is not active.
Then, configure the netplan file to set up static IPs
Edited:
/etc/netplan/*.yaml
Configuration:
<br />
<br />

<img width="974" height="797" alt="image" src="https://github.com/user-attachments/assets/7f51c0e5-3113-43d3-9acd-34a4659884ec" />

<br />
<br />

### Applied with:
- sudo chmod 600 /etc/netplan/*.yaml
- sudo netplan apply
### Verified:
- ip a

 <br />
 <br />
<img width="974" height="811" alt="image" src="https://github.com/user-attachments/assets/9cdedc87-a043-44da-956c-baaa432eddf0" />
<br />
 <br />
 
### This ensures:
- Static addressing
- DNS resolution via Domain Controller
- No dependency on DHCP
 <br />
 <br />
 
## Step 6 Network Validation Across Infrastructure
### After configuring:
- Windows 11 → 192.168.100.10
- Kali → 192.168.100.20
- AD Server → 192.168.100.5
- Splunk Server → 192.168.100.30
I validated full subnet communication.

 
### Ping Tested:
- Windows → Kali
- Kali → Windows
- Splunk → AD
- AD → Splunk
All nodes responded to ICMP Echo after firewall rule configuration. 
 <br />
<br />
<img width="974" height="352" alt="image" src="https://github.com/user-attachments/assets/7577054d-63b3-491b-9ecb-d67f25e4e79e" />


### This confirms:
- Layer 2 switching via Hyper-V Internal switch
- Layer 3 routing within subnet
- No subnet misconfiguration
- No gateway requirement (isolated lab)





## Technical Concepts Reinforced
- Private IPv4 addressing (RFC 1918)
- CIDR notation (/24)
- Subnet mask interpretation (255.255.255.0)
- Layer 2 vs Layer 3 communication
- Broadcast domain boundaries
- Windows Firewall profile behavior (Public vs Private)
- ICMP filtering and inbound rule creation
- Hyper-V Internal Switch behavior (non-routed)





## Architectural Decisions: 
| Decision                       | Rational                   
| ------------------------------ | -----------------------------------------------------| 
| **AD Windows Server**            | Active Directory (AD) on a Windows Server for production environments, since managing Windows workstations, due to its native compatibility, ease of use, and full support for Group Policies. | 
| **Splunk Ubuntu Server**       | Splunk on an Ubuntu server (Linux) due to its superior performance, lower resource usage, and better support for advanced features like SmartStore. Linux is generally preferred for indexing, searching, and overall system stability. | 


## Security Considerations
-  Kali connected to Internal switch by default
-  External access used only when required
-  No attack traffic exposed to production/home LAN
-  Hash validation performed prior to deployment
