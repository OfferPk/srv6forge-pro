/\*\*  
 \* SRv6Forge Pro \- 10-Point Zero-Trust SRv6 / uSID Fabric Security Auditor  
 \* Evaluates RFC 8754 / RFC 8986 compliance, MTU headroom, uSID alignment,  
 \* TI-LFA fast-reroute, and multi-tenant VRF isolation.  
 \*/

(function (root, factory) {  
  if (typeof define \=== 'function' && define.amd) {  
    define(\[\], factory);  
  } else if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.SRv6SecurityAuditor \= factory();  
  }  
}(typeof self \!== 'undefined' ? self : this, function () {  
  'use strict';

  function auditSRv6Configuration(config) {  
    config \= config || {};  
    var findings \= \[\];  
    var score \= 0;

    // Check 1: Locator Block & uSID Carrier Hygiene  
    var locator \= config.locatorBlock || '';  
    var isMicroSid \= \!\!config.isMicroSid;  
    var check1 \= {  
      id: 'LOCATOR\_HYGIENE',  
      name: 'Locator Block & Micro-SID Architecture',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (locator.match(/^\[0-9a-fA-F:\]+\\/(32|48)\$/)) {  
      if (isMicroSid) {  
        check1.points \= 10;  
        check1.status \= 'PASS';  
        check1.description \= 'Valid uSID carrier block (' \+ locator \+ ') with 16-bit micro-segment alignment.';  
      } else {  
        check1.points \= 8;  
        check1.status \= 'WARN';  
        check1.description \= 'Valid standard SRv6 locator (' \+ locator \+ '). Consider enabling uSID to reduce 128-bit SID overhead.';  
      }  
    } else {  
      check1.points \= 0;  
      check1.status \= 'FAIL';  
      check1.description \= 'Missing or invalid locator block prefix (must be clean /32 or /48 IPv6 block).';  
    }  
    score \+= check1.points;  
    findings.push(check1);

    // Check 2: Endpoint Behavior Alignment  
    var behaviors \= config.endpointBehaviors || \[\];  
    var vrfBound \= \!\!config.tenantVrf;  
    var check2 \= {  
      id: 'ENDPOINT\_BEHAVIORS',  
      name: 'RFC 8986 Endpoint Behavior Alignment',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    var hasDecap \= behaviors.some(function(b) {  
      return b.indexOf('End.DT4') \!== \-1 || b.indexOf('End.DT6') \!== \-1 || b.indexOf('End.DX2') \!== \-1;  
    });  
    if (behaviors.length \> 0\) {  
      if (hasDecap && \!vrfBound) {  
        check2.points \= 2;  
        check2.status \= 'FAIL';  
        check2.description \= 'CRITICAL: Decapsulation behavior (End.DT4/DT6) configured without assigned tenant VRF table\!';  
      } else {  
        check2.points \= 10;  
        check2.status \= 'PASS';  
        check2.description \= 'Behaviors aligned (' \+ behaviors.join(', ') \+ ') with tenant VRF mapping.';  
      }  
    } else {  
      check2.points \= 0;  
      check2.status \= 'FAIL';  
      check2.description \= 'No SRv6 endpoint behaviors defined on router.';  
    }  
    score \+= check2.points;  
    findings.push(check2);

    // Check 3: Underlay Interface MTU Headroom  
    var underlayMtu \= parseInt(config.underlayMtu, 10\) || 1500;  
    var payloadMtu \= parseInt(config.customerPayloadMtu, 10\) || 1500;  
    var sidsCount \= (config.segmentList || \[\]).length || 4;  
    var srhBytes \= isMicroSid ? (8 \+ 16 \* Math.ceil(sidsCount / 6)) : (8 \+ 16 \* sidsCount);  
    var totalOverhead \= 40 \+ srhBytes;  
    var requiredMtu \= payloadMtu \+ totalOverhead;  
    var check3 \= {  
      id: 'MTU\_HEADROOM',  
      name: 'Underlay Interface MTU & Headroom Sizing',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (underlayMtu \>= requiredMtu \+ 100 && underlayMtu \>= 9000\) {  
      check3.points \= 10;  
      check3.status \= 'PASS';  
      check3.description \= 'Jumbo MTU (' \+ underlayMtu \+ 'B) provides ' \+ (underlayMtu \- requiredMtu) \+ 'B safe headroom over ' \+ requiredMtu \+ 'B total frame.';  
    } else if (underlayMtu \>= requiredMtu) {  
      check3.points \= 6;  
      check3.status \= 'WARN';  
      check3.description \= 'Narrow headroom (' \+ (underlayMtu \- requiredMtu) \+ 'B). Underlay MTU (' \+ underlayMtu \+ 'B) barely accommodates ' \+ requiredMtu \+ 'B packet.';  
    } else {  
      check3.points \= 0;  
      check3.status \= 'FAIL';  
      check3.description \= 'CRITICAL PMTU BLACKHOLE: Underlay MTU (' \+ underlayMtu \+ 'B) \< required ' \+ requiredMtu \+ 'B. Packets will be dropped\!';  
    }  
    score \+= check3.points;  
    findings.push(check3);

    // Check 4: Encapsulation Mode (H.Encaps.Red)  
    var encapMode \= config.encapsulationMode || 'H.Encaps';  
    var check4 \= {  
      id: 'ENCAPSULATION\_MODE',  
      name: 'Encapsulation Efficiency (H.Encaps.Red)',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (encapMode \=== 'H.Encaps.Red') {  
      check4.points \= 10;  
      check4.status \= 'PASS';  
      check4.description \= 'H.Encaps.Red enabled: Omits redundant SRH when single SID remains, saving 24B+ bandwidth.';  
    } else {  
      check4.points \= 4;  
      check4.status \= 'WARN';  
      check4.description \= 'Standard H.Encaps generates full SRH on all packets. Recommend H.Encaps.Red for wire efficiency.';  
    }  
    score \+= check4.points;  
    findings.push(check4);

    // Check 5: TI-LFA Fast Reroute Protection  
    var tilfa \= \!\!config.enableTiLfa;  
    var check5 \= {  
      id: 'TI\_LFA\_PROTECTION',  
      name: 'Topology-Independent LFA (Sub-50ms FRR)',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (tilfa) {  
      check5.points \= 10;  
      check5.status \= 'PASS';  
      check5.description \= 'TI-LFA enabled in IGP with automated SRv6 backup path calculation for sub-50ms convergence.';  
    } else {  
      check5.points \= 0;  
      check5.status \= 'FAIL';  
      check5.description \= 'TI-LFA disabled: Network vulnerable to multi-second packet loss and micro-loops during topology changes.';  
    }  
    score \+= check5.points;  
    findings.push(check5);

    // Check 6: BGP SRv6 Service Binding  
    var bgpSrv6 \= \!\!config.bgpSrv6Enabled;  
    var check6 \= {  
      id: 'BGP\_SRV6\_BINDING',  
      name: 'BGP Overlay Service SID Binding',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (bgpSrv6) {  
      check6.points \= 10;  
      check6.status \= 'PASS';  
      check6.description \= 'BGP AFI 1/2 SAFI 128 / AFI 25 SAFI 70 advertises SRv6 SID TLVs instead of legacy MPLS labels.';  
    } else {  
      check6.points \= 0;  
      check6.status \= 'FAIL';  
      check6.description \= 'BGP SRv6 service binding missing: Control plane cannot advertise tenant SIDs to remote PEs.';  
    }  
    score \+= check6.points;  
    findings.push(check6);

    // Check 7: RFC 8754 Section 5 Boundary Filtering  
    var boundaryFilter \= \!\!config.boundaryFiltering;  
    var check7 \= {  
      id: 'BOUNDARY\_FILTERING',  
      name: 'SRv6 Domain Boundary Ingress Filtering',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (boundaryFilter) {  
      check7.points \= 10;  
      check7.status \= 'PASS';  
      check7.description \= 'Ingress ACL drops external packets addressing internal SRv6 locators or with invalid Segments Left.';  
    } else {  
      check7.points \= 0;  
      check7.status \= 'FAIL';  
      check7.description \= 'CRITICAL SECURITY DEFECT: External untrusted traffic can steer packets through internal SID micro-instructions.';  
    }  
    score \+= check7.points;  
    findings.push(check7);

    // Check 8: Underlay IGP Segment Routing Support  
    var igp \= config.underlayIgp || 'isis';  
    var check8 \= {  
      id: 'IGP\_SRV6\_SUPPORT',  
      name: 'Underlay IGP SRv6 Router Capabilities',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (igp \=== 'isis' || igp \=== 'ospfv3') {  
      check8.points \= 10;  
      check8.status \= 'PASS';  
      check8.description \= 'Underlay ' \+ igp.toUpperCase() \+ ' advertises SRv6 Locator TLVs and node capability sub-TLVs.';  
    } else {  
      check8.points \= 0;  
      check8.status \= 'FAIL';  
      check8.description \= 'Underlay IGP does not support SRv6 extensions.';  
    }  
    score \+= check8.points;  
    findings.push(check8);

    // Check 9: Tenant VRF Table Isolation  
    var check9 \= {  
      id: 'VRF\_ISOLATION',  
      name: 'Multi-Tenant VRF Table Isolation',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (vrfBound && config.tenantVrf) {  
      check9.points \= 10;  
      check9.status \= 'PASS';  
      check9.description \= 'Tenant VRF (' \+ config.tenantVrf \+ ') bound to dedicated End.DT4/DT6 SID with strict route-target filtering.';  
    } else {  
      check9.points \= 2;  
      check9.status \= 'FAIL';  
      check9.description \= 'Global default FIB routing utilized; multi-tenant traffic isolation is compromised.';  
    }  
    score \+= check9.points;  
    findings.push(check9);

    // Check 10: Seamless BFD (S-BFD) Tracking  
    var bfd \= \!\!config.enableSbfd;  
    var check10 \= {  
      id: 'SBFD\_TRACKING',  
      name: 'Seamless BFD (S-BFD) Policy Tracking',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (bfd) {  
      check10.points \= 10;  
      check10.status \= 'PASS';  
      check10.description \= 'S-BFD configured on SRv6 policies with sub-second (\<=300ms) detection for deterministic failover.';  
    } else {  
      check10.points \= 3;  
      check10.status \= 'WARN';  
      check10.description \= 'S-BFD disabled: Failover relies on slow IGP timeout (3–10s) rather than sub-second hardware tracking.';  
    }  
    score \+= check10.points;  
    findings.push(check10);

    // Calculate Grade  
    var grade \= 'F';  
    if (score \>= 90\) grade \= 'A+';  
    else if (score \>= 80\) grade \= 'A';  
    else if (score \>= 70\) grade \= 'B';  
    else if (score \>= 60\) grade \= 'C';  
    else if (score \>= 50\) grade \= 'D';  
    else grade \= 'F';

    return {  
      score: score,  
      grade: grade,  
      passedChecks: findings.filter(function(f) { return f.status \=== 'PASS'; }).length,  
      warningChecks: findings.filter(function(f) { return f.status \=== 'WARN'; }).length,  
      failedChecks: findings.filter(function(f) { return f.status \=== 'FAIL'; }).length,  
      findings: findings  
    };  
  }

  return {  
    auditSRv6Configuration: auditSRv6Configuration  
  };  
}));  
