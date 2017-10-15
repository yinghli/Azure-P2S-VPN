Azure Point to Site (P2S) VPN with RADIUS Authentication
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
We don't include VPN gateway setup. If needed, you can check [Azure IPSec VPN with Cisco ASA using BGP](https://github.com/yinghli/azure-vpn-asa/edit/master/README.md) <br>
After the VPN gateway setup, check the point-to-site configuration and add address pools, tunnel type, RADIUS authentication and RADIUS server information. <br> 
![](https://github.com/yinghli/Azure-P2S-VPN/blob/master/P2SVPNGW.PNG) <br>

After setup, click the "Download VPN Client" to save your VPN client configuration file. <br>
Folders named 'WindowsAmd64' and 'WindowsX86' contain the Windows 64-bit and 32-bit installer packages. <br>
Folder 'GenericDevice' contains general information used to create your own VPN client configuration.<br>
Folder named 'Mac' contains a file named 'mobileconfig'. This file is used to configure Mac clients. <br>
You need install one of them according to your platform. 

FreeRADIUS Server Configuration
-------------------------
We setup a Ubuntu server in subnet vlan1 to host RADIUS and use [FreeRADIUS](http://www.freeradius.org/) to provide RADIUS services.<br>

> **Note:** You can setup Raidus server in VNET or On premise connected by site-to-site VPN. An ExpressRoute connection CANNOT be used.

## Install freeRADIUS
```
sudo apt-get install freeradius
```
## Test freeRADIUS installation
```
freeradius -X
....
Listening on authentication address * port 1812
Listening on accounting address * port 1813
Listening on authentication address 127.0.0.1 port 18120 as server inner-tunnel
Listening on proxy address * port 1814
Ready to process requests.
```
## Setup Radius Client
P2S VPN Gateway subnet is 10.0.0.0/24, we add this subnet into Radius client configuration.
```
vi /etc/freeradius/clients.conf

client new {
        ipaddr = 10.0.0.0
        netmask = 24
        secret = cisco123
}
```
## Setup Radius users
We add a user "testing" with password "password" as test user. 
```
vi /etc/freeradius/users

testing Cleartext-Password := "password"
```

## Verification
After all configuration, you can initial a vpn connection from your Windows or MAC client to check P2S connectivity. <br>

From the freeRADIUS debug output, you can see the Radius request is comming from P2S VPN gateway with 10.0.0.5. 
```
rad_recv: Access-Request packet from host 10.0.0.5 port 51205, id=11, length=219
        NAS-Identifier = "RD0003FF6951F2"
        NAS-Port-Type = Virtual
        Tunnel-Type:0 = ESP
        Tunnel-Medium-Type:0 = IPv4
        Framed-MTU = 1300
        EAP-Message = 0x020200421a0202003d31b4dfea2672ed6cb8ab93f16c8903c7bb0000000000000000ae1ec0b318da0e39ea18993698e1e034e8ac1f2c7f0187e60074657374696e67
        User-Name = "testing"
        State = 0x2cf04a872df2508b7de07a0d613199ca
        MS-RAS-Vendor = 311
        MS-Network-Access-Server-Type = Remote-Access-Server
        Proxy-State = 0xfe800000000000003190a616c233831900000024
        Message-Authenticator = 0x74a564cb5c2dbcb6cd6cd9264d70acda
```



> **Note:** When the Windows device contains a large number of trusted root certificates, the message payload size during IKE exchange is large and causes IP layer fragmentation. The fragments are rejected at the Azure end, which results in the connection failing. The exact certificate count at which this problem occurs is difficult to estimate. As a result, IKEv2 connections from Windows devices are not guaranteed to work. When you configure both SSTP and IKEv2 in a mixed environment (consisting of Windows and Mac devices), the Windows VPN profile always tries IKEv2 tunnel first. If it fails due to the issue described here, it falls back to SSTP.

Windows Server 2016 NPS Configuration
-----------------------
## NPS Radius Client Setup
Add Gateway Subnet as client IP with share secret.<br>
![](https://github.com/yinghli/Azure-P2S-VPN/blob/master/P2S_Radius_Client.PNG)
## NPS Policy Setup
Setup a basic network policy. <br>
![](https://github.com/yinghli/Azure-P2S-VPN/blob/master/P2S_Policy_1.PNG)
Setup a "Framed Prototol" as PPP. <br>
![](https://github.com/yinghli/Azure-P2S-VPN/blob/master/P2S_Policy_2.PNG)
Authentication method should include "MS-CHAPv2". <br>
![](https://github.com/yinghli/Azure-P2S-VPN/blob/master/P2S_Policy_3.PNG)
Radius Attributes is basic configuration. <br>
![](https://github.com/yinghli/Azure-P2S-VPN/blob/master/P2S_Policy_4.PNG)
## NPS User Profile Setup
You must enable user "dial-in" network access pemission. 
![](https://github.com/yinghli/Azure-P2S-VPN/blob/master/P2S_User_Dial-in.PNG)
## Verification
After all configuration, you can initial a vpn connection from your Windows or MAC client to check P2S connectivity. <br>
You can check the logging from event viewer. <br>
![](https://github.com/yinghli/Azure-P2S-VPN/blob/master/P2S_Logging.PNG)
