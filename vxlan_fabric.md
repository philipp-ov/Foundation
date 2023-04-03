# VxLAN EVPN fabric full config example

## Diagram

<img width="1738" alt="image" src="https://user-images.githubusercontent.com/116812447/229617253-8db5f2fb-0458-44f2-a055-76c6fa092d16.png">

## Leaf-1-1 Config

```
## Enable all required features on Nexus
nv overlay evpn
feature bgp
feature interface-vlan
feature vn-segment-vlan-based
feature lacp
feature vpc
feature nv overlay

# Define virtual MAC, the same on all Leafs, so all VMs can see it as one big gateway
fabric forwarding anycast-gateway-mac 0001.0002.0003

# Define all required VLANs, including VLANs for L3 VNIs (not actually used VLANs) and assign VNI to them
vlan 1,100,200,300,400,500
vlan 100
  vn-segment 100
vlan 200
  vn-segment 200
vlan 300
  vn-segment 300
vlan 400
  vn-segment 400
vlan 500
  vn-segment 500

# Create first VRF, assign RT, RD, VNI for it. We can have as mamy VRFs as we want. VRF's VNI is used for all traffic between hosts in one VRF but different VLANs
vrf context OTUS
  vni 300
  
  # All routes from this VRF will be exported to global BGP table with this RD
  rd 10.10.2.3:300
  
  # All routes with RT 300:300 will be imported to this VRF from global BGP tables in L2VPN and IPv4 address families, as well as exported
  address-family ipv4 unicast
    route-target import 300:300
    route-target import 300:300 evpn
    route-target export 300:300
    route-target export 300:300 evpn

# Create second VRF and configure parametres for it
vrf context OTUS2
  vni 500
  rd 10.10.2.3:500
  address-family ipv4 unicast
    route-target import 500:500
    route-target import 500:500 evpn
    route-target export 500:500
    route-target export 500:500 evpn

# Create separate VRF for vPC peer-keepalive interface, according to vendor recomendations
vrf context VPC

# Configure TCAM memory, so we can use command "fabric forwarding mode anycast-gateway" on VLAN interface
hardware access-list tcam region racl 512
hardware access-list tcam region arp-ether 256 double-wide


vpc domain 8
  peer-switch
  role priority 5
  peer-keepalive destination 172.16.16.1 source 172.16.16.2 vrf VPC
  delay restore 10
  peer-gateway
  layer3 peer-router
  ip arp synchronize

interface Vlan100
  no shutdown
  vrf member OTUS
  no ip redirects
  ip address 192.168.1.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan200
  no shutdown
  vrf member OTUS
  no ip redirects
  ip address 192.168.2.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan300
  no shutdown
  vrf member OTUS
  no ip redirects
  ip forward
  no ipv6 redirects

interface Vlan400
  no shutdown
  vrf member OTUS2
  no ip redirects
  ip address 192.168.3.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan500
  no shutdown
  vrf member OTUS2
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel1
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link

interface port-channel101
  switchport access vlan 100
  vpc 8

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 100
    ingress-replication protocol bgp
  member vni 200
    ingress-replication protocol bgp
  member vni 300 associate-vrf
  member vni 400
    ingress-replication protocol bgp
  member vni 500 associate-vrf

interface Ethernet1/1
  no switchport
  ip address 10.1.1.2/30
  no shutdown

interface Ethernet1/2
  switchport access vlan 100
  channel-group 101 mode active

interface Ethernet1/3
  switchport access vlan 200

interface Ethernet1/4
  no switchport
  vrf member VPC
  ip address 172.16.16.2/30
  no shutdown

interface Ethernet1/5
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/6
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/7
  switchport access vlan 400

interface Ethernet1/8

interface Ethernet1/9

interface Ethernet1/10

interface Ethernet1/11

interface Ethernet1/12

interface Ethernet1/13

interface Ethernet1/14

interface Ethernet1/15

interface Ethernet1/16

interface Ethernet1/17

interface Ethernet1/18

interface Ethernet1/19

interface Ethernet1/20

interface Ethernet1/21

interface Ethernet1/22

interface Ethernet1/23

interface Ethernet1/24

interface Ethernet1/25

interface Ethernet1/26

interface Ethernet1/27

interface Ethernet1/28

interface Ethernet1/29

interface Ethernet1/30

interface Ethernet1/31

interface Ethernet1/32

interface Ethernet1/33

interface Ethernet1/34

interface Ethernet1/35

interface Ethernet1/36

interface Ethernet1/37

interface Ethernet1/38

interface Ethernet1/39

interface Ethernet1/40

interface Ethernet1/41

interface Ethernet1/42

interface Ethernet1/43

interface Ethernet1/44

interface Ethernet1/45

interface Ethernet1/46

interface Ethernet1/47

interface Ethernet1/48

interface Ethernet1/49

interface Ethernet1/50

interface Ethernet1/51

interface Ethernet1/52

interface Ethernet1/53

interface Ethernet1/54

interface Ethernet1/55

interface Ethernet1/56

interface Ethernet1/57

interface Ethernet1/58

interface Ethernet1/59

interface Ethernet1/60

interface Ethernet1/61

interface Ethernet1/62

interface Ethernet1/63

interface Ethernet1/64

interface mgmt0
  vrf member management

interface loopback0
  ip address 10.10.2.3/32

interface loopback1
  ip address 10.10.1.4/32
  ip address 10.10.1.3/32 secondary
icam monitor scale

line console
line vty
boot nxos bootflash:/nxos.9.3.10.bin sup-1
router bgp 64701
  router-id 10.10.2.3
  address-family ipv4 unicast
    network 10.10.1.3/32
    network 10.10.1.4/32
    network 10.10.2.3/32
  neighbor 10.1.1.1
    remote-as 64600
    address-family ipv4 unicast
  neighbor 10.10.2.1
    remote-as 64600
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      allowas-in 3
      send-community
      send-community extended
evpn
  vni 100 l2
    rd 10.10.2.3:100
    route-target import 100:100
    route-target export 100:100
  vni 200 l2
    rd 10.10.2.3:200
    route-target import 200:200
    route-target export 200:200
  vni 400 l2
    rd 10.10.2.3:400
    route-target import 400:400
    route-target export 400:400

```
