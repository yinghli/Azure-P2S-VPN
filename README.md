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

Verification
------------------------
After all configuration, you can initial a vpn connection from your Windows or MAC client to check P2S connectivity.

