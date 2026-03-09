#WEEK 5 - PHASE 4#

##THE GREAT MIGRATION
The time has finally come to migrate my homelab from a flat to a VLAN-segmented network.

**2/3/2026**

-	Installing PfSense – downloaded from the official website -> VM configured in Proxmox: 2 core CPU, 2GB RAM, 20GB Hard Drive, 2 network interface cards (firewall unchecked, we don’t want Proxmox firewall to be working), start at boot configured.
-	PfSense installed succesfully
-	Accessed the WebGUI by disabling the inital block -> pfsense console -> `pfSsh.php playback enableallowallwan` -> accessed through the local IP.
-	Configuring PfSense -> Setup Wizard -> Hostname:FW01 -> Domain:leapcorp.local (matches my Domain) -> primary DNS 1.1.1.1, secondary DNS 8.8.8.8 -> override DNS unchecked -> reload -> Congratulations! pfSense is now configured.
-	Proxmox configuration -> pve node -> System -> Network -> bridge interface -> VLAN aware checked -> OK -> Apply Configuration
-	TP-Link ES208G config -> VLAN -> 802.1Q VLAN -> enabled -> added the following VLANs – VLAN 10 (servers) port 2 tagged, VLAN 20 (employees) port 2 tagged, VLAN 40 (cctv) port 2 tagged, VLAN 666 (security) port 2 tagged. (all on port 2 as it’s all virtualized on a Dell Optiplex)
-	Back to pfSense -> Interfaces -> Assignments -> VLANS -> added all 4 VLANs configured via LAN network interface card
-	Interface assignments -> added all 4 -> next step is configuring each
-	OPT1 -> renamed to **Servers** -> Ipv4 Configuration type -> Static IPv4 -> Ipv4 Address: 192.168.10.1/24
-	OPT2 -> renamed to **Employees** -> Ipv4 Configuration type -> Static IPv4 -> Ipv4 Address: 192.168.20.1/24
-	OPT3 -> renamed to **CCTV** -> Ipv4 Configuration type -> Static IPv4 -> Ipv4 Address: 192.168.40.1/24
-	OPT4 -> renamed to **Security** -> Ipv4 Configuration type -> Static IPv4 -> Ipv4 Address: 192.168.66.1/24
-	Configuring DHCP on pfSense -> Services -> DHCP server -> got a notice - *ISC DHCP has reached end-of-life and will be removed in a future version of pfSense*. Visit System > Advanced > Networking to switch DHCP backend. 
-	switching to the new Kea DHCP -> System -> Advanced -> Networking -> Server Backedn -> Kea DHCP -> Save.
-	Assigning DHCP IP ranges -> Emplyoees: 192.168.20.10 to 192.168.20.245, CCTV: 192.168.40.10 to 192.168.40.245, Security: 192.168.66.10 to 192.168.66.245

Things are ready for the **Great Migration**. Before the cutout, I am setting up Management Override Rules – need to be able to access everything from my main machine during the configuration process.
-	pfSense -> Firewall -> Rules -> WAN -> add -> action: PASS -> Interface -> WAN -> Address Family -> IPv4 -> Protool: ANY (in an enterprise environment we would only enable RDP ports but for my lab management I opt for ANY for convenience) -> Source: Network – 192.168.50.0/24 -> Destination: ANY -> Description: IT Admin Management Override -> Save -> Apply Changes.
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/pfsense%20install.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/interfaces.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/TP-Link%20VLANs.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/Firewall%20Override.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/VLANs%20configured.png)

---
**VLAN Tagging and IP assignment**

-	Proxmox dashboard -> DC01 -> Hardware -> Network Device -> VLAN Tag 10
-	Proxmox dashboard -> FS01 -> Hardware -> Network Device -> VLAN Tag 10
-	DC01 Console -> Network Connections -> right click adapter -> Properties -> IPv4 properties -> IP: 192.168.10.5, Subnet Mask: 255.255.255.0, Default Gateway: 192.168.10.1, DNS primary – 127.0.0.1 secondary – 1.1.1.1
-	FS01 console – tried changing IP addresses via sconfig but got an error – failed to release DHCP error code 83. Googled it up and learned it’s a bug in the OS that sometimes happens in sconfig, so used PowerShell commands to bypass it:
-	`Get-NetAdapter`
-	`Remove-NetIPAddress -InterfaceAlias "Ethernet" -Confirm:$False`
-	`New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.6 -PrefixLength 24 -DefaultGateway 192.168.10.1`
-	`Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses "192.168.10.5"`
-	Ran ipconfig and confirmed the changes.
-	Both servers migrated to the correct VLAN, IT Admin Management Override implemented, pinged the servers -> **REQUEST TIMED OUT**
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/FS01%20IP%20powershell.png)


---
**Configuring VM 101 (virtual machine on the proxmox hypervisor), getting back to the servers later** 

-	Dashbaord -> Hardware -> Network device -> VLAN Tag 20 -> ok.
-	Logged into the machine -> `ipconfig /release` -> `ipconfig /renew` -> ip config – IP 192.168.20.10, Default Gateway – 192.168.20.1 -> **SUCCESS**
-	`Ping 192.168.10.5` -> **REQUEST TIMED OUT** -> checking if the VLANs are configured and working properly -> pfSense -> Diagnostics -> Ping -> hostname 192.168.10.5 -> source address SERVERS -> ping -> **SUCCESS**
-	time to configure Firewall rules -> Employees -> add -> Action: PASS -> Interface: Employees -> Address family IPv4 -> Protocol: Any -> Source: Emplyoees network -> Destination: Any -> Decription: Temporary Allow ALL – Emplyoees
-	Repeat for Servers tab.
-	Back to VM101 -> `ping 8.8.8.8` – **SUCCESS**
-	`Ping 192.168.10.5` -> **SUCCESS**
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/pfsense%20VLAN%20ping.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/Employee%20VM%20IP.png)


---
**Configuring AD DNS**

-	pfSense -> Services -> DHCP Server -> Employees -> Server Options -> DNS 1 – 192.168.10.5, DNS 2 – 1.1.1.1 -> Save -> Apply Changes
-	ran `ipconfig /release` -> `ipconfig /renew` on DC01 -> `ipconfig/all` – DNS 192.168.50.5 (old DNS address)
-	restaring service in pfSense -> Status -> Services -> kea-dhcp4 -> restart service -> ran `ipconfig /release` -> `ipconfig /renew` -> `ipconfig/all` – issue persisted DNS 192.168.50.5 (old DNS address)
-	at this point I start suspecting that manual DNS setting is overriding the firewall configuration so I opened Network Connections -> properties -> IPv4 properties -> **CULPRIT FOUND**
-	Switched to Obtain Automatically
-	-ran ran `ipconfig /release` -> `ipconfig /renew` -> `ipconfig/all` – DNS 192.168.10.5 -> **SUCCESS!**
-	Fixing the main laptop now by adding:
`route add 192.168.10.0 mask 255.255.255.0 192.168.50.164 -p`
`route add 192.168.20.0 mask 255.255.255.0 192.168.50.164 –p` (making it permanently send packets for the two VLANs to pfSense instead of the router.
-	`Ping 192.168.10.5` -> **SUCCESS**
-	Logged into IT Admin (VM machine on the main laptop) – the machine can ping the server (because of the routing rules it probably inherited from the host machine), but there is no wallpaper and the network drive is not operational -> Network Settings -> IPv4 Settings -> properties -> Manually set DNS 192.168.10.5 -> gpupdate/force -> reboot -> GPOs pulled, wallpaper in place, network drive fully functional!
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/Employee%20VM%20IP.png)

---

7/3/2026
**Adding Firewall rules and hardening VLANs**

-	PfSense firewall -> rules -> Employees -> add -> block -> Interface Employees -> protocol TCP -> Source Employee subnets -> Destination -> Employee address (192.168.20.1 gateway) -> destination port 443 HTTPS -> Description – Block EMPLOYEES from pfSense WebGUI -> Save.
-	Creating **Alias** for the necessary AD ports:
-	pfSense -> Firewall -> Aliases -> add -> name AD_Requiered_Ports -> description -> Core ports for AD -> add ports **53, 88, 123, 135, 389, 445,464, 636, 3268, 49152:65535** -> save -> Apply Changes
Adding Rules:
-	pfSense -> Firewall -> Rules -> Employees -> add up -> action: pass -> interface: Enployees -> address family: IPv4 -> Protocol: TCP/UPD -> source: Employees subnets -> destination: Servers subnets -> destination port range – from: AD_Requiered_Ports, to – blank -> Decription Allow Employees to Servers (AD ports) -> save -> apply changes.
-	Delete the rule Temoprary Allow All – Employees.
-	Tested by pinging 192.168.10.5 (timed out) and Test-NetConnection 192.168.10.5 -Port 3389 (failed) – **Firewall rules applied successfully!**
-	Allowing internet access:
-	Building an **RFC1918** Alias to exclude private Ips keeping the **Zero Trust Model** fully implemented while allowing for public internet access -> Firewall -> aliases -> IP -> add -> Name: RFC1918 -> Description: All private IP Ranges -> add network *10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16* -> Save -> Apply Changes
Adding Rules:
-	Firewall -> Rules -> add down -> action:pass -> address family. IPv4 -> Protocol: any -> source: Employees -> Destination: alias RFC1918 *(check invert match so they are excepted)* -> Description: Allow Employees to Public Internet (Zero-Trust) -> save -> apply changes.
-	Repeat for Servers
-	Internet access **VERIFIED**
--
**DC01 Remote Desktop Protocol:**
-	RDP blocked on the firewall level
-	IT Admin can RDP into DC01 due to Managment Override Allow ANY rule
-	Hardening the RDP:
-	Assigned a static IP to IT Admin -> Network Settings -> right click Ethernet Adapter -> Proterties -> IPV4 -> Properties -> Use the following IP Address – 192.168.1.143
-	Edit pfSense Wan rule -> source -> Network -> address or alias -> 192.168.1.143
-	DC01 Windows Defender lockdown -> wf.msc -> Inbound Rules -> Remote Desktop – User Mode (TCP-In) -> Remote IP Address -> These IP addresses -> Add This IP or subnet -> 192.168.1.143
-	**RDP’d back in succesfully**
-	Furhter verification -> Installed Windows App on my Samsung Galaxy s23 Ultra and tried to RDP into DC01 -> **UNSUCCESSFUL**

-	**The Great Migration officially completed – Network hardened.**

* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/Port%20Alias.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/IP%20Alias.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/Firewall%20Rules%20-%20EMPLOYEES.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/Firewall%20Rules%20-%20SERVERS.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/IT%20Admin%20Static.png)
* ![Evidence](https://github.com/AlexPopCyberSec/My-IT-Career-Transition-Journey/blob/main/Leap-Corp-Enterprise-Proxmox/images/week%205/Firewal%20IT%20Admin%20static.png)

---
