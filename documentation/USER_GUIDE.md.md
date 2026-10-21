\# SRv6Forge Pro \- User Guide & CI/CD Telemetry Pipeline Manual

\#\# 1\. Web Studio Quickstart

1\. Open \`source-code/index.html\` in any modern web browser.  
2\. Select an industry preset from the top dropdown:  
   \- \*\*Telco 5G UPF (Cisco IOS-XR) \[Grade A+\]\*\*: 5G Standalone UPF transport network with Cisco IOS-XR, uSID \`/32\` block, \`End.DT4\` L3VPN, TI-LFA, and \`H.Encaps.Red\`.  
   \- \*\*Cloud Backbone TE (Juniper Junos) \[Grade A+\]\*\*: Hyperscale multi-region backbone with Juniper Junos, \`End\` \+ \`End.X\` traffic engineering policies, S-BFD tracking.  
   \- \*\*Vulnerable Bloated SRH Network \[Grade F\]\*\*: Uncompressed 6-hop legacy SRH with 1500B MTU drop, misaligned uSIDs, missing TI-LFA, unassigned VRF table.  
3\. Review the \*\*10-Point Zero-Trust Scorecard\*\* to inspect locator block alignment, MTU headroom, and TI-LFA protection.  
4\. Modify SRv6 parameters (Source IP, Locator Block, uSID mode, Endpoint Behaviors, Segment List, MTU).  
5\. Inspect the generated \*\*RFC 8754 SRH Wireframe & Hex Dump\*\* with byte offsets and decoded field markers.  
6\. Switch synthesizer tabs to export production configurations:  
   \- \*\*Cisco IOS-XR\*\*: \`segment-routing srv6\`, \`locators\`, \`router isis\`, \`router bgp\`.  
   \- \*\*Juniper Junos\*\*: \`set protocols source-packet-routing srv6\`, \`set routing-instances\`.  
   \- \*\*Nokia SR-OS\*\*: \`/configure router segment-routing srv6\`, \`/configure service vprn\`.  
   \- \*\*Linux Kernel / FRR\*\*: \`ip route add ... encap seg6\` and \`/etc/frr/frr.conf\`.  
   \- \*\*Prometheus Exporter\*\*: Telemetry exporter script for port \`:9126\`.

\---

\#\# 2\. DevSecOps CLI Tool (\`srv6forge-cli.py\`)

The standalone Python CLI tool requires zero external dependencies and runs on Python 3.8+.

\#\#\# 2.1 Encoding SRH Binary Hex Dumps  
\`\`\`bash  
python3 build/srv6forge-cli.py encode-srh \-p presets/telco-5g-transport-cisco-iosxr.json  
\`\`\`

\#\#\# 2.2 Calculating IPv6 & SRH MTU Headroom  
\`\`\`bash  
python3 build/srv6forge-cli.py calc-mtu \--underlay 9192 \--payload 1500 \--sids 4 \--usid  
\`\`\`  
Output:  
\`\`\`  
\=== SRv6 / uSID Encapsulation MTU Analysis \===  
Underlay Interface MTU:  9192 Bytes  
Customer Payload MTU:    1500 Bytes  
IPv6 Base Overhead:      40 Bytes  
SRH Extension Overhead:  24 Bytes  
Total Header Overhead:   64 Bytes  
Required MTU:            1564 Bytes  
Safe Fabric Headroom:    7628 Bytes  
MTU Compliance Status:   PASSED (Safe)  
\`\`\`

\#\#\# 2.3 Running CI/CD Pipeline Telemetry Audits  
\`\`\`bash  
\# Clean production configuration \-\> Exits code 0  
python3 build/srv6forge-cli.py audit \-p presets/telco-5g-transport-cisco-iosxr.json \--strict

\# Insecure/fragile configuration \-\> Exits code 1 (Pipeline Fails)  
python3 build/srv6forge-cli.py audit \-p presets/vulnerable-bloated-srh-network.json \--strict  
\`\`\`

\#\#\# 2.4 Synthesizing Router Configurations  
\`\`\`bash  
python3 build/srv6forge-cli.py synthesize \-p presets/telco-5g-transport-cisco-iosxr.json \-f cisco  
python3 build/srv6forge-cli.py synthesize \-p presets/cloud-backbone-juniper-junos.json \-f juniper  
\`\`\`

\---

\#\# 3\. GitHub Actions CI/CD Pipeline Gate

Add this workflow to your NetDevOps repository (\`.github/workflows/srv6-fabric-gate.yml\`):

\`\`\`yaml  
name: Zero-Trust SRv6 Network Programming CI/CD Gate

on:  
  pull\_request:  
    paths:  
      \- 'srv6/\*\*'  
      \- 'network/\*\*'  
      \- 'fabric/\*\*'

jobs:  
  srv6-audit:  
    runs-on: ubuntu-latest  
    steps:  
      \- name: Checkout Code  
        uses: actions/checkout@v4

      \- name: Set up Python  
        uses: actions/setup-python@v5  
        with:  
          python-version: '3.11'

      \- name: Audit SRv6 Configuration (Strict Gate)  
        run: |  
          python3 build/srv6forge-cli.py audit \-p presets/telco-5g-transport-cisco-iosxr.json \--strict  
\`\`\`  
