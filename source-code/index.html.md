SRv6 • uSID

# 

SRv6Forge Pro

v1.0.0  
RFC 8754 / RFC 8986 Segment Routing over IPv6 & Micro-SID Studio  
Preset: Telco 5G UPF (Cisco IOS-XR) \[Grade A+\] Preset: Cloud Backbone TE (Juniper Junos) \[Grade A+\] Preset: Vulnerable Bloated SRH Network \[Grade F\] Copy Config Export JSON

## 

1\. Node & Locator Architecture

Router ID / Loopback  
BGP ASN  
SRv6 Source IPv6 Address

Locator Name  
Locator Block (IPv6 Prefix)  
Enable Compressed Micro-SID (uSID \- 16-bit shift mode)

## 

2\. Endpoint Behaviors & Multi-Tenancy

Active RFC 8986 Behaviors  
End (Transit SID) End.X (L3 Adjacency) End.DT4 (IPv4 VRF Decap) End.DT6 (IPv6 VRF Decap) End.DX2 (EVPN L2 Decap) End.T (Transit Table)

Tenant VRF Table Name  
Encapsulation Mode H.Encaps.Red (Reduced SRH) H.Encaps (Full SRH)

## 

3\. Segment List & MTU Sizing

Segment List (SIDs in reverse order) fcbb:bb00:0001:0002:: fcbb:bb00:0003:0004::

Underlay Interface MTU  
Payload MTU

## **4\. Zero-Trust Controls**

Enable Topology-Independent LFA (Sub-50ms FRR) Enable Seamless BFD (S-BFD ≤ 300ms Tracking) RFC 8754 Section 5 Boundary Ingress Filtering BGP SRv6 Service SID Binding (AFI 1/2 SAFI 128 / AFI 25\)

A+

#### 

Zero-Trust SRv6 Score: 100 / 100

uSID Carrier Active • H.Encaps.Red • TI-LFA Protected • Jumbo Safe

**IPv6 / SRH Encapsulation Headroom** **\+7604B Headroom**  
Payload: **1500B** Overhead (IPv6+SRH): **88B** Interface MTU: **9192B**

## 

10-Point Zero-Trust SRv6 Fabric Scorecard

## 

RFC 8754 SRH Wireframe & Hex Dump  
IPv6: 40B | SRH: 48B

`// Loading hex dump...`

Cisco IOS-XR Juniper Junos Nokia SR-OS Linux Kernel / FRR Prometheus Exporter  
`! Generating configuration...`