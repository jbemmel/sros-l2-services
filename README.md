# sros-l2-services
Virtual lab showcasing various L2 services and how to configure them

Inspired by the [Advanced Solutions Guide](https://documentation.nokia.com/cgi-bin/dbaccessfilename.cgi/3HE14991AAAITQZZA01_V1_Advanced%20Configuration%20Guide%20Part%20II%20for%20Releases%20Up%20To%2022.10.R3.pdf) around page 102

![image](https://github.com/jbemmel/sros-l2-services/assets/2031627/dc195830-c6bf-4923-b0b8-0402580eec69)

## Getting started
This lab uses [Netlab](https://github.com/ipspace/netlab) to bring up the initial topology:
```
netlab up
```
Notes:
* assumes you have built a vrnetlab SR OS image, see https://containerlab.dev/manual/vrnetlab/
* requires a license for VSR

## Testing services
Log in to the nodes to see services:
```
ssh admin@clab-L2-services-r1
show service service-using
```
```
A:admin@r1# /show service service-using 

===============================================================================
Services 
===============================================================================
ServiceId    Type      Adm  Opr  CustomerId Service Name
-------------------------------------------------------------------------------
1            VPLS      Up   Up   1          VPLS-1
2            Epipe     Up   Up   1          Epipe-2
3            VPLS      Up   Up   1          BGP-VPLS-3
4            VPLS      Up   Up   1          BGP-AD-VPLS-4
5            Epipe     Up   Down 1          BGP-VPWS-5
6            Epipe     Up   Up   1          BGP-EVPN-VPWS
11           VPRN      Up   Up   1          client-1
12           VPRN      Up   Up   1          client-2
13           VPRN      Up   Up   1          client-3
14           VPRN      Up   Up   1          client-4
15           VPRN      Up   Up   1          client-5
16           VPRN      Up   Up   1          client-6
2147483648   IES       Up   Down 1          _tmnx_InternalIesService
2147483649   intVpls   Up   Down 1          _tmnx_InternalVplsService
-------------------------------------------------------------------------------
Matching Services : 14
-------------------------------------------------------------------------------
```

Note how BGP-VPWS-5 is still down - being worked on

For example, to test service Epipe-2:
```
A:admin@r1# ping 10.0.12.22 router-instance "client-2" 
PING 10.0.12.22 56 data bytes
64 bytes from 10.0.12.22: icmp_seq=1 ttl=64 time=5.48ms.
64 bytes from 10.0.12.22: icmp_seq=2 ttl=64 time=1.89ms.
64 bytes from 10.0.12.22: icmp_seq=3 ttl=64 time=2.74ms.
ping aborted by user

```

## Status (on SR-1)

|     Case      |   Working?  |
| ------------- | ----------- |
|  VPLS-1       |     ✅      |  `ping 10.0.11.21 router-instance "client-1"`
|  Epipe-2      |     ✅      |  `ping 10.0.12.22 router-instance "client-2"`
|  BGP-VPLS-3   |     ✅      |  `ping 10.0.13.23 router-instance "client-3"`, requires SDP local/far end to match BGP peering addresses
| BGP-AD-VPLS-4 |     ✅      |  `ping 10.0.14.24 router-instance "client-4"`, SDP local/far match *and* GRE loopback IP must match iBGP peering IP
|  BGP-VPWS-5   |     ✅      |  `ping 10.0.15.25 router-instance "client-5"`, requires SDP local/far end to match BGP peering addresses
| BGP-EVPN-VPWS |     ❌      |  `ping 10.0.16.26 router-instance "client-6"`

### BGP l2-vpn status

```
A:admin@r1# /show router bgp neighbor "10.0.0.1" received-routes l2-vpn
===============================================================================
 BGP Router ID:10.0.0.2         AS:65000       Local AS:65000      
===============================================================================
 Legend -
 Status codes  : u - used, s - suppressed, h - history, d - decayed, * - valid
                 l - leaked, x - stale, > - best, b - backup, p - purge
 Origin codes  : i - IGP, e - EGP, ? - incomplete

===============================================================================
BGP L2VPN Routes
===============================================================================
Flag  RouteType                   Prefix                             MED
      RD                          SiteId                             Label
      Nexthop                     VeId                   BlockSize   LocalPref
      As-Path                     BaseOffset             vplsLabelBa 
                                                         se          
-------------------------------------------------------------------------------
i     AutoDiscovery               10.0.0.2               -           0
      10.0.0.2:60000              -                                  -
      10.0.0.2                    -                      -           100
      No As-Path                  -                      -            
i     VPLS                        -                      -           0
      10.0.0.2:60001              -                                  -
      10.0.0.2                    1                      8           100
      No As-Path                  1                      524271       
i     VPWS                        -                      -           0
      10.0.0.2:60002              -                                  -
      10.0.0.2                    1                      1           100
      No As-Path                  2                      524284       
u*>i  AutoDiscovery               10.0.0.3               -           0
      10.0.0.3:60000              -                                  -
      10.0.0.3                    -                      -           100
      No As-Path                  -                      -            
u*>i  VPLS                        -                      -           0
      10.0.0.3:60001              -                                  -
      10.0.0.3                    2                      8           100
      No As-Path                  1                      524271       
u*>i  VPWS                        -                      -           0
      10.0.0.3:60002              -                                  -
      10.0.0.3                    2                      1           100
      No As-Path                  1                      524282       
-------------------------------------------------------------------------------
Routes : 6
===============================================================================
```