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
11           VPRN      Up   Up   1          client-1
12           VPRN      Up   Up   1          client-2
13           VPRN      Up   Up   1          client-3
14           VPRN      Up   Up   1          client-4
15           VPRN      Up   Up   1          client-5
2147483648   IES       Up   Down 1          _tmnx_InternalIesService
2147483649   intVpls   Up   Down 1          _tmnx_InternalVplsService
-------------------------------------------------------------------------------
Matching Services : 12
-------------------------------------------------------------------------------
===============================================================================
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