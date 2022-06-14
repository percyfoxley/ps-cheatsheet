# **Network**

### **Command Prompt**
---
### **Configuring IPv4**

#### **Show interfaces**

**netsh**

`netsh interface ipv4 show interfaces`

Idx |    Met    |     MTU    |      State  |              Name
--- | ---------- | ---------- | ------------ | ---------------------------
  1     |     75  |4294967295 | connected   |  Loopback Pseudo-Interface 1
 12     |     35    |    1500 | connected  |  Ethernet
  8     |     35   |     1500 | disconnected | Ethernet 2
 13     |      5    |    1392 | disconnected | Ethernet 3
 28     |   5000    |    1500 | connected   |  vEthernet (Default Switch)
 16      |    25    |    1500 | connected   |  Ethernet 4


### Set Dhcp

`netsh interface ipv4 set address <Idx or Interface Name> source=dhcp`

`netsh interface ipv4 set address 8 source=dhcp`

### Set Static
`netsh interface ipv4 set address <Idx or Interface Name> static <IPv4 Address> <Subnet Mask> <Gateway>`

`netsh interface ipv4 set address "Ethernet" static 10.0.0.31 255.0.0.0 10.0.0.1`
## **PowerShell** 
---

**List all Network Adapters**

`PS C:\Users\PC> Get-NetAdapter`

Name             |         InterfaceDescription             |       ifIndex  |  Status    |       LinkSpeed 
---|---|---|---|---|
Ethernet 4       |         VirtualBox Host-Only Ethernet Adapter     |   16 | Up    |         1 Gbps
Ethernet 3        |        Fortinet SSL VPN Virtual Ethernet Ad...  |    13 | Disconnected |   100 Gbps
Ethernet           |       Realtek Gaming GbE Family Controller     |    12 |  Up        |   100 Mbps
Yerel Ağ Bağlantısı    |   PPPoP WAN Adapter                        |    11 | Disconnected  |      0 bps
Ethernet 2          |      Fortinet Virtual Ethernet Adapter (N...    |   8 | Disconnected   | 100 Mbps
vEthernet (Default Swi... | Hyper-V Virtual Ethernet Adapter         |    28 | Up        |       10 Gbps

**Get a spesific network adapter by name**

`PS C:\Users\PC> Get-NetAdapter -Name *vEthernet*`

Name                |      InterfaceDescription        |            ifIndex | Status   |    MacAddress       |      LinkSpeed |
----       |               --------------------         |           ------- | ------    |   ----------         |    ---------
vEthernet (Default Swi... | Hyper-V Virtual Ethernet Adapter    |         28 | Up      |     00-15-5D-2C-EA-6B   |     10 Gbps

**Get more information**

`PS C:\Users\PC> Get-NetAdapter | ft Name,ifIndex,Status`

PS C:\Users\PC> Get-NetAdapter | ft Name,ifIndex,Status

Name      |                 ifIndex |Status
----       |                -------| ------
Ethernet 4   |                   16| Up
Ethernet 3  |                    13 |Disconnected
Ethernet      |                  12| Up
Yerel Ağ Bağlantısı      |       11| Disconnected
Ethernet 2                |       8| Disconnected
vEthernet (Default Switch) |     28| Up

**Disable and Enable a Network Adapter**

`Disable-NetAdapter -Name "Ethernet"`

`Enable-NetAdapter -Name "Ethernet"`

**Set IP Address**

`New-NetIPAddress -InterfaceIndex 8 -IPv4Address 10.0.1.31 -PrefixLength "24" -DefaultGateway 10.0.1.1 `

or DHCP enable

`Set-NetIPInterface -InterfaceIndex 8 -DHCP Enabled`

**Also we can network interface to a variable for future use**

`PS C:\Users\PC> $interface=Get-NetIPInterface -InterfaceIndex 16 -AddressFamily IPv4`

`PS C:\Users\PC> $interface`

ifIndex |InterfaceAlias           |       AddressFamily| NlMtu(Bytes)| InterfaceMetric| Dhcp    | ConnectionState |PolicyStore
-------| --------------          |        -------------| ------------ |---------------| ----  |   --------------- |-----------
16     | Ethernet 4              |        IPv4          |        1500         |     25| Enabled | Connected     |  ActiveStore



