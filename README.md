Azure Point to Site (P2S) VPN with EAP-MSCHAPv2 Authentication
========================================
Azure networking release their P2S VPN support on MAC OS and EAP-MSCHAPv2 authentication. <br>
Now P2S VPN support both SSTP and IKEv2, authentication support both certificate and RADIUS. <br>
This documentation will describe how to setup P2S VPN with EAP authentication. <br>
We will use Windows Server 2016 NPS and FreeRADIUS as RADIUS server. <br>

Topology
-----------------
![](https://github.com/yinghli/Azure-P2S-VPN/blob/master/P2SVPN.png)

Azure P2S VPN Setup
--------------------
In Azure side, we will use Azure Portal to setup all vpn configuration. PowerShell and Azure CLI can do the same setup. <br>
We will use below parameters to setup. <br>

Parameters            | Values
----------------------| -------------
VNET Name             | P2S
Address Space         | 10.0.0.0/16
Resource Group        | P2S
Location              | West Europe
Subnet                | vlan1
Address Range         | 10.0.1.0/24
GatewaySubnet         | 10.0.0.0/24
VPN Gateway Name      | P2S
VPN Type              | Route-based
VPN SKU               | VpnGw1
VPN Address Pool      | 192.168.2.0/24
VPN Tunnel Type       | IKEv2
VPN Authentication    | RADIUS
RADIUS Server         | 10.0.1.4
Server secret         | cisco123

P2S VPN Gateway Setup
----------------------
We don't include VPN gateway setup. If needed, you can check [Azure IPSec VPN with Cisco ASA using BGP](https://github.com/yinghli/azure-vpn-asa/edit/master/README.md)
After the VPN gateway setup, check the point-to-site configuration and add address pools, tunnel type, RADIUS authentication and RADIUS server information. <br> 
![](https://github.com/yinghli/Azure-P2S-VPN/blob/master/P2SVPNGW.PNG)
After setup, click the "Download VPN Client" to save your VPN client configuration file. <br>
Folders named 'WindowsAmd64' and 'WindowsX86' contain the Windows 64-bit and 32-bit installer packages. <br>
Folder 'GenericDevice' contains general information used to create your own VPN client configuration.<br>
Folder named 'Mac' contains a file named 'mobileconfig'. This file is used to configure Mac clients. <br>

RADIUS Server Configuration
-------------------------

