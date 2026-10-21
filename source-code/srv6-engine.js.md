/\*\*  
 \* SRv6Forge Pro \- RFC 8754 / RFC 8986 Segment Routing over IPv6 (SRv6) Engine  
 \* Models IPv6 Base Headers, Segment Routing Headers (SRH Routing Type 4),  
 \* uSID (Micro-SID) compression, and MTU headroom calculation.  
 \*/

(function (root, factory) {  
  if (typeof define \=== 'function' && define.amd) {  
    define(\[\], factory);  
  } else if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.SRv6Engine \= factory();  
  }  
}(typeof self \!== 'undefined' ? self : this, function () {  
  'use strict';

  // Helper: Parse IPv6 string to 16-byte array  
  function parseIPv6(ipStr) {  
    if (\!ipStr || typeof ipStr \!== 'string') {  
      return new Array(16).fill(0);  
    }  
    var clean \= ipStr.trim().toLowerCase();  
    var parts \= clean.split('::');  
    var left \= parts\[0\] ? parts\[0\].split(':') : \[\];  
    var right \= parts.length \> 1 && parts\[1\] ? parts\[1\].split(':') : \[\];

    var bytes \= new Array(16).fill(0);  
    var leftWords \= \[\];  
    for (var i \= 0; i \< left.length; i++) {  
      if (left\[i\] \!== '') leftWords.push(parseInt(left\[i\], 16\) || 0);  
    }

    var rightWords \= \[\];  
    for (var j \= 0; j \< right.length; j++) {  
      if (right\[j\] \!== '') rightWords.push(parseInt(right\[j\], 16\) || 0);  
    }

    var totalWords \= 8;  
    var missingWords \= totalWords \- (leftWords.length \+ rightWords.length);  
    var combined \= \[\];  
    combined \= combined.concat(leftWords);  
    for (var m \= 0; m \< missingWords; m++) {  
      combined.push(0);  
    }  
    combined \= combined.concat(rightWords);

    for (var w \= 0; w \< 8; w++) {  
      var word \= combined\[w\] || 0;  
      bytes\[w \* 2\] \= (word \>\> 8\) & 0xff;  
      bytes\[w \* 2 \+ 1\] \= word & 0xff;  
    }  
    return bytes;  
  }

  // Helper: Format 16-byte array as IPv6 string  
  function formatIPv6(bytes) {  
    var words \= \[\];  
    for (var i \= 0; i \< 16; i \+= 2\) {  
      words.push(((bytes\[i\] \<\< 8\) | bytes\[i \+ 1\]).toString(16));  
    }  
    return words.join(':');  
  }

  // Encode RFC 8754 Segment Routing Header (Routing Type 4\)  
  function encodeSRH(params) {  
    params \= params || {};  
    var nextHeader \= params.nextHeader \!== undefined ? params.nextHeader : 4; // 4 \= IPv4 encapsulation, 41 \= IPv6, 59 \= None  
    var sids \= params.segmentList || \[\]; // Array of IPv6 SID strings  
    if (sids.length \=== 0\) {  
      sids \= \['fcbb:bb00:0001:0001::'\];  
    }

    var numSids \= sids.length;  
    // Hdr Ext Len \= (8 \+ 16 \* numSids \- 8\) / 8 \= 2 \* numSids  
    var hdrExtLen \= 2 \* numSids;  
    var routingType \= 4; // RFC 8754 SRH  
    var lastEntry \= numSids \- 1;  
    var segmentsLeft \= params.segmentsLeft \!== undefined ? params.segmentsLeft : lastEntry;  
    var flags \= params.flags \!== undefined ? params.flags : 0x00; // Bit 7: Protected, Bit 6: OAM  
    if (params.flagProtected) flags |= 0x80;  
    if (params.flagOam) flags |= 0x40;

    var tag \= params.tag \!== undefined ? params.tag : 0x0000;

    var srhBytes \= \[\];  
    srhBytes.push(nextHeader & 0xff);  
    srhBytes.push(hdrExtLen & 0xff);  
    srhBytes.push(routingType & 0xff);  
    srhBytes.push(segmentsLeft & 0xff);  
    srhBytes.push(lastEntry & 0xff);  
    srhBytes.push(flags & 0xff);  
    srhBytes.push((tag \>\> 8\) & 0xff);  
    srhBytes.push(tag & 0xff);

    // Segment List \[0..Last Entry\] in reverse order (RFC 8754: Segments are listed in reverse execution order)  
    // Segment List\[0\] is the final destination, Segment List\[Last Entry\] is first hop.  
    for (var s \= 0; s \< numSids; s++) {  
      var sidBytes \= parseIPv6(sids\[s\]);  
      for (var b \= 0; b \< 16; b++) {  
        srhBytes.push(sidBytes\[b\]);  
      }  
    }

    return srhBytes;  
  }

  // Encode Outer IPv6 Base Header (RFC 8200 \- 40 Bytes)  
  function encodeIPv6Base(params, payloadLength) {  
    params \= params || {};  
    var srcIp \= parseIPv6(params.sourceIp || '2001:db8:ffff::1');  
    var dstIp \= parseIPv6(params.destinationIp || 'fcbb:bb00:0001:0001::');  
    var nextHeader \= params.hasSRH ? 43 : (params.payloadNextHeader || 4); // 43 \= Routing Header  
    var hopLimit \= params.hopLimit || 64;  
    var trafficClass \= params.trafficClass || 0;  
    var flowLabel \= params.flowLabel || 0;

    var bytes \= \[\];  
    // Version (4 bits: 6\) \+ Traffic Class top 4 bits  
    bytes.push(0x60 | ((trafficClass \>\> 4\) & 0x0f));  
    // Traffic Class lower 4 bits \+ Flow Label top 4 bits  
    bytes.push(((trafficClass & 0x0f) \<\< 4\) | ((flowLabel \>\> 16\) & 0x0f));  
    // Flow Label mid 8 bits  
    bytes.push((flowLabel \>\> 8\) & 0xff);  
    // Flow Label low 8 bits  
    bytes.push(flowLabel & 0xff);

    // Payload Length (16 bits)  
    bytes.push((payloadLength \>\> 8\) & 0xff);  
    bytes.push(payloadLength & 0xff);

    // Next Header  
    bytes.push(nextHeader & 0xff);  
    // Hop Limit  
    bytes.push(hopLimit & 0xff);

    // Source IP (16 bytes)  
    for (var i \= 0; i \< 16; i++) bytes.push(srcIp\[i\]);  
    // Destination IP (16 bytes)  
    for (var j \= 0; j \< 16; j++) bytes.push(dstIp\[j\]);

    return bytes;  
  }

  // Complete SRv6 Packet Header Wireframe  
  function buildSRv6Wireframe(params) {  
    params \= params || {};  
    var isReduced \= params.encapsulationMode \=== 'H.Encaps.Red' && (params.segmentList || \[\]).length \<= 1;  
    var srhBytes \= \[\];  
    if (\!isReduced) {  
      srhBytes \= encodeSRH(params);  
    }

    var dummyPayloadBytes \= params.dummyPayloadBytes || 20; // Simulated inner IP header  
    var totalPayloadLen \= srhBytes.length \+ dummyPayloadBytes;

    var ipv6Bytes \= encodeIPv6Base({  
      sourceIp: params.sourceIp,  
      destinationIp: isReduced && params.segmentList && params.segmentList.length \> 0 ? params.segmentList\[0\] : (params.segmentList ? params.segmentList\[params.segmentList.length \- 1\] : 'fcbb:bb00::1'),  
      hasSRH: \!isReduced,  
      payloadNextHeader: params.nextHeader || 4,  
      trafficClass: params.trafficClass,  
      flowLabel: params.flowLabel,  
      hopLimit: params.hopLimit  
    }, totalPayloadLen);

    var fullWireframe \= ipv6Bytes.concat(srhBytes);  
    return {  
      bytes: fullWireframe,  
      ipv6BaseLen: ipv6Bytes.length,  
      srhLen: srhBytes.length,  
      isReduced: isReduced,  
      segmentsCount: (params.segmentList || \[\]).length  
    };  
  }

  // Calculate MTU Overhead and Safe Headroom  
  function computeSRv6MTU(params) {  
    params \= params || {};  
    var underlayMtu \= parseInt(params.underlayMtu, 10\) || 9192;  
    var customerPayloadMtu \= parseInt(params.customerPayloadMtu, 10\) || 1500;  
    var sidsCount \= parseInt(params.sidsCount, 10\) || 4;  
    var isMicroSid \= \!\!params.isMicroSid;  
    var isReduced \= params.encapsulationMode \=== 'H.Encaps.Red' && sidsCount \<= 1;

    var ipv6HeaderOverhead \= 40;  
    var srhHeaderOverhead \= 0;

    if (\!isReduced) {  
      if (isMicroSid) {  
        // uSID packs 6 micro-SIDs per 128-bit carrier  
        var carrierBlocks \= Math.ceil(sidsCount / 6\) || 1;  
        srhHeaderOverhead \= 8 \+ (16 \* carrierBlocks);  
      } else {  
        srhHeaderOverhead \= 8 \+ (16 \* sidsCount);  
      }  
    }

    var totalEncapsulationOverhead \= ipv6HeaderOverhead \+ srhHeaderOverhead;  
    var requiredUnderlayMtu \= customerPayloadMtu \+ totalEncapsulationOverhead;  
    var headroomBytes \= underlayMtu \- requiredUnderlayMtu;

    var status \= 'SAFE';  
    var statusMessage \= 'Headroom sufficient for zero fragmentation.';  
    if (headroomBytes \< 0\) {  
      status \= 'BLACKHOLE';  
      statusMessage \= 'CRITICAL: Interface MTU drops encapsulated packets (' \+ Math.abs(headroomBytes) \+ 'B deficit)\!';  
    } else if (headroomBytes \< 64\) {  
      status \= 'TIGHT';  
      statusMessage \= 'WARNING: Narrow MTU headroom buffer (\<64B). Potential fragmentation risk.';  
    }

    return {  
      underlayMtu: underlayMtu,  
      customerPayloadMtu: customerPayloadMtu,  
      ipv6Overhead: ipv6HeaderOverhead,  
      srhOverhead: srhHeaderOverhead,  
      totalOverhead: totalEncapsulationOverhead,  
      requiredMtu: requiredUnderlayMtu,  
      headroomBytes: headroomBytes,  
      status: status,  
      statusMessage: statusMessage,  
      isMicroSid: isMicroSid,  
      isReduced: isReduced  
    };  
  }

  // Format bytes into Wireshark / hexdump format  
  function formatHexDump(bytes) {  
    var lines \= \[\];  
    for (var i \= 0; i \< bytes.length; i \+= 16\) {  
      var chunk \= bytes.slice(i, i \+ 16);  
      var offsetHex \= ('0000' \+ i.toString(16)).slice(-4);  
      var hexParts \= \[\];  
      var asciiParts \= \[\];

      for (var j \= 0; j \< 16; j++) {  
        if (j \< chunk.length) {  
          var b \= chunk\[j\];  
          hexParts.push(('0' \+ b.toString(16)).slice(-2));  
          asciiParts.push((b \>= 32 && b \<= 126\) ? String.fromCharCode(b) : '.');  
        } else {  
          hexParts.push('  ');  
          asciiParts.push(' ');  
        }  
      }

      var hexFormatted \= hexParts.slice(0, 8).join(' ') \+ '  ' \+ hexParts.slice(8).join(' ');  
      lines.push(offsetHex \+ ':  ' \+ hexFormatted \+ '  |' \+ asciiParts.join('') \+ '|');  
    }  
    return lines.join('\\n');  
  }

  return {  
    parseIPv6: parseIPv6,  
    formatIPv6: formatIPv6,  
    encodeSRH: encodeSRH,  
    encodeIPv6Base: encodeIPv6Base,  
    buildSRv6Wireframe: buildSRv6Wireframe,  
    computeSRv6MTU: computeSRv6MTU,  
    formatHexDump: formatHexDump  
  };  
}));  
