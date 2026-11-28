## 1. What is DHCPv6?

DHCPv6 provides IPv6 configuration to network clients, such as:

- IPv6 addresses (Stateful mode)
- DNS server addresses
- Other network configuration information

There are **two modes**:

|Mode|IPv6 address|DNS information|
|---|---|---|
|**Stateful DHCPv6**|DHCPv6|DHCPv6|
|**Stateless DHCPv6**|SLAAC|DHCPv6|

---

## 2. Common configuration

```
Router(config)# ipv6 unicast-routing
```

**Role:** Enables IPv6 routing on the router. It is needed for the router to send IPv6 Router Advertisements (RA).
**RA (Router Advertisement)** : Message envoyé par le routeur pour annoncer les informations du réseau IPv6.

---
## 3. Stateful DHCPv6

**Stateful = DHCPv6 assigns the IPv6 address and provides DNS information.**
### Step 1 — Create the DHCPv6 pool

```
Router(config)# ipv6 dhcp pool DHCP6
Router(config-dhcpv6)# address prefix 2001:db8:10::/64
Router(config-dhcpv6)# dns-server 2001:4860:4860::8888
```

- `ipv6 dhcp pool DHCP6`: creates a DHCPv6 pool named `DHCP6`.
- `address prefix`: defines the IPv6 prefix used for client addresses.
- `dns-server`: specifies the DNS server address.

### Step 2 — Configure the client-facing interface

```
Router(config)# interface g0/0
Router(config-if)# ipv6 address 2001:db8:10::1/64
Router(config-if)# ipv6 nd managed-config-flag
Router(config-if)# ipv6 dhcp server DHCP6
Router(config-if)# no shutdown
```

- `interface g0/0`: selects the interface connected to the clients.
- `ipv6 address`: configures the router's IPv6 address on that interface.
- `ipv6 nd managed-config-flag`: sets the M flag in Router Advertisements; clients should obtain their IPv6 address through DHCPv6.
- `ipv6 dhcp server DHCP6`: attaches the DHCPv6 pool to this interface.
- `no shutdown`: enables the interface.

---

## 4. Stateless DHCPv6

**Stateless = SLAAC gives the IPv6 address; DHCPv6 provides DNS and other information.**

### Step 1 — Create the DHCPv6 pool

```
Router(config)# ipv6 dhcp pool DHCP6
Router(config-dhcpv6)# dns-server 2001:4860:4860::8888
```

In this mode, the pool does not need an `address prefix` command because clients create their IPv6 addresses using SLAAC.

### Step 2 — Configure the client-facing interface

```
Router(config)# interface g0/0
Router(config-if)# ipv6 address 2001:db8:10::1/64
Router(config-if)# ipv6 nd other-config-flag
Router(config-if)# ipv6 dhcp server DHCP6
Router(config-if)# no shutdown
```

- `ipv6 nd other-config-flag`: sets the O flag in Router Advertisements; clients use SLAAC for their IPv6 address and DHCPv6 for additional information, such as DNS.
- `ipv6 dhcp server DHCP6`: attaches the pool to the interface.

---

## 5. Stateful vs Stateless — remember this!

|Command / concept|Meaning|
|---|---|
|`ipv6 unicast-routing`|Enables IPv6 routing|
|`ipv6 dhcp pool NAME`|Creates a DHCPv6 pool|
|`address prefix .../64`|Defines the client address prefix (Stateful)|
|`dns-server ...`|Specifies the DNS server|
|`ipv6 nd managed-config-flag`|M flag: use DHCPv6 for the IPv6 address|
|`ipv6 nd other-config-flag`|O flag: use DHCPv6 for other information|
|`ipv6 dhcp server NAME`|Applies the pool to an interface|

### Easy memory trick 🧠

- **M = Managed** → DHCPv6 manages the IPv6 address.
- **O = Other** → SLAAC manages the IPv6 address; DHCPv6 provides other settings.
- **SLAAC** → the client forms its own IPv6 address using information advertised by the router.

---

## 6. How it works

### Stateful

1. The client receives a Router Advertisement with the **M flag** enabled.
2. The client contacts the DHCPv6 server.
3. DHCPv6 assigns the IPv6 address and provides DNS information.

### Stateless

1. The client receives a Router Advertisement with the **O flag** enabled.
2. The client creates its IPv6 address using SLAAC.
3. The client contacts DHCPv6 to obtain DNS and other configuration information.

---

## 7. Verification commands

```
show ipv6 interface g0/0
show ipv6 dhcp pool
show ipv6 dhcp binding
show running-config
```

- `show ipv6 interface g0/0`: checks IPv6 interface configuration and RA flags.
- `show ipv6 dhcp pool`: displays DHCPv6 pool information.
- `show ipv6 dhcp binding`: displays DHCPv6 address bindings (mainly useful in Stateful mode).
- `show running-config`: displays the router configuration.

# **🌐 OSPFv2 & OSPFv3 — Simple Explanation**

## **9.2 OSPFv2 (IPv4)**

|Command|Role|
|---|---|
|`router ospf 1`|Starts OSPF process 1.|
|`router-id 1.1.1.1`|Sets the router's unique OSPF ID.|
|`network 192.168.10.0 0.0.0.255 area 0`|Activates OSPF on matching interfaces in Area 0.|
|`network 192.168.10.1 0.0.0.0 area 0`|Activates OSPF on the interface with this exact IP.|
|`ip ospf 1 area 0`|Activates OSPF directly on an interface.|
|`passive-interface g0/0`|Stops sending Hello packets but still advertises the network.|
|`default-information originate`|Advertises the default route to OSPF neighbors.|
### **🧠 Important concepts**

- **OSPF 1:** Process ID (locally significant).
    
- **Router ID:** Unique identifier of the router.
    
- **Area 0:** Backbone area.
    
- **Wildcard mask:** Identifies which IP addresses match.
    
- **Passive interface:** No OSPF Hello messages are sent, but the connected network can still be advertised.
### **Three ways to activate OSPF**

- **M1:** Match a network using a wildcard mask.
    
- **M2:** Match one exact interface IP using `0.0.0.0`.
    
- **M3:** Activate OSPF directly inside interface configuration.

## **9.3 OSPFv3 (IPv6)**

|Command|Role|
|---|---|
|`ipv6 router ospf 1`|Starts OSPFv3 process 1.|
|`router-id 1.1.1.1`|Sets the OSPFv3 Router ID.|
|`ipv6 ospf 1 area 0`|Activates OSPFv3 on the interface in Area 0.|

### **Example configuration**

**Step 1 — Enable IPv6 routing**

```
ipv6 unicast-routing
```

**Step 2 — Create OSPFv3 process**

```
Router(config)# ipv6 router ospf 1
Router(config-rtr)# router-id 1.1.1.1
```

**Step 3 — Configure the interface**

```
Router(config)# interface g0/0
Router(config-if)# ipv6 ospf 1 area 0                 # Activates OSPFv3 on the interface and assigns it to Area 0.
```

**example for all**

```bash
enable
configure terminal

ipv6 unicast-routing

ipv6 router ospf 1
router-id 1.1.1.1
exit

interface g0/0
ipv6 address 2001:db8:12::1/64
ipv6 ospf 1 area 0
no shutdown
```

## **🔥 OSPFv2 vs OSPFv3**

|OSPFv2|OSPFv3|
|---|---|
|IPv4|IPv6|
|`router ospf 1`|`ipv6 router ospf 1`|
|`ip ospf 1 area 0`|`ipv6 ospf 1 area 0`|
|IPv4 network command|Interface-based activation|

**Remember for your exam:**

- OSPFv2 = IPv4.
    
- OSPFv3 = IPv6.
    
- Router ID = identifies the router.
    
- Area 0 = backbone.
    
- Passive interface = no Hello packets, network still advertised.
    
- Default-information originate = advertises a default route (if one exists).
