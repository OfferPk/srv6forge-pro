/\*\*  
 \* SRv6Forge Pro \- UI Orchestration & Event Controller  
 \*/

(function () {  
  'use strict';

  var currentTab \= 'cisco';  
  var presets \= {  
    'telco-5g-transport-cisco-iosxr': {  
      routerId: '192.0.2.1',  
      bgpAsn: '65001',  
      sourceIp: '2001:db8:ffff::1',  
      locatorName: 'uSID\_LOC01',  
      locatorBlock: 'fcbb:bb00::/32',  
      isMicroSid: true,  
      endpointBehaviors: \['End', 'End.X', 'End.DT4'\],  
      tenantVrf: 'VRF\_5G\_UPF',  
      encapsulationMode: 'H.Encaps.Red',  
      underlayMtu: 9192,  
      customerPayloadMtu: 1500,  
      enableTiLfa: true,  
      enableSbfd: true,  
      boundaryFiltering: true,  
      bgpSrv6Enabled: true,  
      segmentList: \['fcbb:bb00:0001:0002::', 'fcbb:bb00:0003:0004::'\]  
    },  
    'cloud-backbone-juniper-junos': {  
      routerId: '198.51.100.1',  
      bgpAsn: '64512',  
      sourceIp: '2001:db8:aaaa::1',  
      locatorName: 'JUNOS\_LOC\_EAST',  
      locatorBlock: '2001:db8:bbbb::/48',  
      isMicroSid: true,  
      endpointBehaviors: \['End', 'End.X', 'End.T'\],  
      tenantVrf: 'VRF\_TRANSIT\_EAST',  
      encapsulationMode: 'H.Encaps.Red',  
      underlayMtu: 9192,  
      customerPayloadMtu: 1500,  
      enableTiLfa: true,  
      enableSbfd: true,  
      boundaryFiltering: true,  
      bgpSrv6Enabled: true,  
      segmentList: \['2001:db8:bbbb:0001::', '2001:db8:bbbb:0002::', '2001:db8:bbbb:0003::'\]  
    },  
    'vulnerable-bloated-srh-network': {  
      routerId: '10.0.0.1',  
      bgpAsn: '65100',  
      sourceIp: '2001:db8::1',  
      locatorName: 'DEFAULT\_LOC',  
      locatorBlock: 'fe80::/64',  
      isMicroSid: false,  
      endpointBehaviors: \['End.DT4'\],  
      tenantVrf: '',  
      encapsulationMode: 'H.Encaps',  
      underlayMtu: 1500,  
      customerPayloadMtu: 1500,  
      enableTiLfa: false,  
      enableSbfd: false,  
      boundaryFiltering: false,  
      bgpSrv6Enabled: false,  
      segmentList: \[  
        '2001:db8:1::1',  
        '2001:db8:2::1',  
        '2001:db8:3::1',  
        '2001:db8:4::1',  
        '2001:db8:5::1',  
        '2001:db8:6::1'  
      \]  
    }  
  };

  function getFormConfig() {  
    var sidsRaw \= document.getElementById('segmentList').value.split('\\n');  
    var sids \= \[\];  
    for (var i \= 0; i \< sidsRaw.length; i++) {  
      var s \= sidsRaw\[i\].trim();  
      if (s) sids.push(s);  
    }

    var behaviors \= \[\];  
    if (document.getElementById('behEnd').checked) behaviors.push('End');  
    if (document.getElementById('behEndX').checked) behaviors.push('End.X');  
    if (document.getElementById('behEndDT4').checked) behaviors.push('End.DT4');  
    if (document.getElementById('behEndDT6').checked) behaviors.push('End.DT6');  
    if (document.getElementById('behEndDX2').checked) behaviors.push('End.DX2');  
    if (document.getElementById('behEndT').checked) behaviors.push('End.T');

    return {  
      routerId: document.getElementById('routerId').value.trim(),  
      bgpAsn: document.getElementById('bgpAsn').value.trim(),  
      sourceIp: document.getElementById('sourceIp').value.trim(),  
      locatorName: document.getElementById('locatorName').value.trim(),  
      locatorBlock: document.getElementById('locatorBlock').value.trim(),  
      isMicroSid: document.getElementById('isMicroSid').checked,  
      endpointBehaviors: behaviors,  
      tenantVrf: document.getElementById('tenantVrf').value.trim(),  
      encapsulationMode: document.getElementById('encapsulationMode').value,  
      underlayMtu: parseInt(document.getElementById('underlayMtu').value, 10\) || 9192,  
      customerPayloadMtu: parseInt(document.getElementById('customerPayloadMtu').value, 10\) || 1500,  
      enableTiLfa: document.getElementById('enableTiLfa').checked,  
      enableSbfd: document.getElementById('enableSbfd').checked,  
      boundaryFiltering: document.getElementById('boundaryFiltering').checked,  
      bgpSrv6Enabled: document.getElementById('bgpSrv6Enabled').checked,  
      segmentList: sids  
    };  
  }

  function applyPreset(presetKey) {  
    var p \= presets\[presetKey\];  
    if (\!p) return;

    document.getElementById('routerId').value \= p.routerId;  
    document.getElementById('bgpAsn').value \= p.bgpAsn;  
    document.getElementById('sourceIp').value \= p.sourceIp;  
    document.getElementById('locatorName').value \= p.locatorName;  
    document.getElementById('locatorBlock').value \= p.locatorBlock;  
    document.getElementById('isMicroSid').checked \= p.isMicroSid;

    document.getElementById('behEnd').checked \= p.endpointBehaviors.indexOf('End') \!== \-1;  
    document.getElementById('behEndX').checked \= p.endpointBehaviors.indexOf('End.X') \!== \-1;  
    document.getElementById('behEndDT4').checked \= p.endpointBehaviors.indexOf('End.DT4') \!== \-1;  
    document.getElementById('behEndDT6').checked \= p.endpointBehaviors.indexOf('End.DT6') \!== \-1;  
    document.getElementById('behEndDX2').checked \= p.endpointBehaviors.indexOf('End.DX2') \!== \-1;  
    document.getElementById('behEndT').checked \= p.endpointBehaviors.indexOf('End.T') \!== \-1;

    document.getElementById('tenantVrf').value \= p.tenantVrf;  
    document.getElementById('encapsulationMode').value \= p.encapsulationMode;  
    document.getElementById('underlayMtu').value \= p.underlayMtu;  
    document.getElementById('customerPayloadMtu').value \= p.customerPayloadMtu;  
    document.getElementById('enableTiLfa').checked \= p.enableTiLfa;  
    document.getElementById('enableSbfd').checked \= p.enableSbfd;  
    document.getElementById('boundaryFiltering').checked \= p.boundaryFiltering;  
    document.getElementById('bgpSrv6Enabled').checked \= p.bgpSrv6Enabled;  
    document.getElementById('segmentList').value \= p.segmentList.join('\\n');

    updateStudio();  
  }

  function updateStudio() {  
    var cfg \= getFormConfig();

    // 1\. Run Auditor  
    var audit \= window.SRv6SecurityAuditor.auditSRv6Configuration(cfg);  
    var badge \= document.getElementById('scoreBadge');  
    badge.textContent \= audit.grade;  
    badge.className \= 'score-badge ' \+ (  
      audit.grade \=== 'A+' || audit.grade \=== 'A' ? 'grade-a' :  
      audit.grade \=== 'B' ? 'grade-b' :  
      audit.grade \=== 'C' ? 'grade-c' : 'grade-f'  
    );

    document.getElementById('scoreTitle').textContent \= 'Zero-Trust SRv6 Score: ' \+ audit.score \+ ' / 100 (' \+ audit.grade \+ ')';  
    document.getElementById('scoreSummary').textContent \=  
      audit.passedChecks \+ ' Checks Passed \&bull; ' \+ audit.warningChecks \+ ' Warnings \&bull; ' \+ audit.failedChecks \+ ' Critical Risks';

    // Render 10 Rubric Items  
    var container \= document.getElementById('rubricContainer');  
    container.innerHTML \= '';  
    for (var i \= 0; i \< audit.findings.length; i++) {  
      var item \= audit.findings\[i\];  
      var div \= document.createElement('div');  
      var statusClass \= item.status \=== 'PASS' ? 'pass' : (item.status \=== 'WARN' ? 'warn' : 'fail');  
      var badgeClass \= item.status \=== 'PASS' ? 'status-pass-badge' : (item.status \=== 'WARN' ? 'status-warn-badge' : 'status-fail-badge');  
      div.className \= 'rubric-item ' \+ statusClass;  
      div.innerHTML \= '\<div\>\<strong\>' \+ (i \+ 1\) \+ '. ' \+ item.name \+ '\</strong\> (' \+ item.points \+ '/' \+ item.weight \+ ' pts)\<br\>' \+  
        '\<span style="color:var(--text-secondary);"\>' \+ item.description \+ '\</span\>\</div\>' \+  
        '\<span class="rubric-status ' \+ badgeClass \+ '"\>' \+ item.status \+ '\</span\>';  
      container.appendChild(div);  
    }

    // 2\. MTU Calculations  
    var mtu \= window.SRv6Engine.computeSRv6MTU({  
      underlayMtu: cfg.underlayMtu,  
      customerPayloadMtu: cfg.customerPayloadMtu,  
      sidsCount: cfg.segmentList.length,  
      isMicroSid: cfg.isMicroSid,  
      encapsulationMode: cfg.encapsulationMode  
    });

    var headroomSpan \= document.getElementById('mtuHeadroomText');  
    if (mtu.headroomBytes \>= 0\) {  
      headroomSpan.textContent \= '+' \+ mtu.headroomBytes \+ 'B Headroom';  
      headroomSpan.style.color \= '\#34d399';  
    } else {  
      headroomSpan.textContent \= mtu.headroomBytes \+ 'B Blackhole';  
      headroomSpan.style.color \= '\#f87171';  
    }

    var totalMax \= Math.max(mtu.underlayMtu, mtu.requiredMtu, 2000);  
    var payloadPct \= Math.round((mtu.customerPayloadMtu / totalMax) \* 100);  
    var overheadPct \= Math.round((mtu.totalOverhead / totalMax) \* 100);  
    var headroomPct \= mtu.headroomBytes \> 0 ? Math.round((mtu.headroomBytes / totalMax) \* 100\) : 0;

    document.getElementById('barPayload').style.width \= payloadPct \+ '%';  
    document.getElementById('barOverhead').style.width \= overheadPct \+ '%';  
    document.getElementById('barHeadroom').style.width \= headroomPct \+ '%';

    document.getElementById('valPayload').textContent \= mtu.customerPayloadMtu \+ 'B';  
    document.getElementById('valOverhead').textContent \= mtu.totalOverhead \+ 'B (' \+ (mtu.isReduced ? 'Reduced' : 'Full') \+ ')';  
    document.getElementById('valUnderlay').textContent \= mtu.underlayMtu \+ 'B';

    // 3\. Hex Dump Wireframe  
    var wire \= window.SRv6Engine.buildSRv6Wireframe(cfg);  
    var hex \= window.SRv6Engine.formatHexDump(wire.bytes);  
    document.getElementById('wireframeMeta').textContent \=  
      'Total: ' \+ wire.bytes.length \+ 'B (IPv6: ' \+ wire.ipv6BaseLen \+ 'B | SRH: ' \+ wire.srhLen \+ 'B | SIDs: ' \+ wire.segmentsCount \+ ')';  
    document.getElementById('hexDumpOutput').textContent \= hex;

    // 4\. Multi-Vendor Synthesizer  
    var code \= '';  
    if (currentTab \=== 'cisco') code \= window.SRv6ConfigSynthesizer.synthesizeCiscoIOSXR(cfg);  
    else if (currentTab \=== 'juniper') code \= window.SRv6ConfigSynthesizer.synthesizeJuniperJunos(cfg);  
    else if (currentTab \=== 'nokia') code \= window.SRv6ConfigSynthesizer.synthesizeNokiaSROS(cfg);  
    else if (currentTab \=== 'linux') code \= window.SRv6ConfigSynthesizer.synthesizeLinuxKernel(cfg);  
    else if (currentTab \=== 'prometheus') code \= window.SRv6ConfigSynthesizer.synthesizePrometheusExporter(cfg);

    document.getElementById('configOutput').textContent \= code;  
  }

  // Event Listeners  
  document.addEventListener('DOMContentLoaded', function () {  
    var inputs \= document.querySelectorAll('input, select, textarea');  
    for (var i \= 0; i \< inputs.length; i++) {  
      inputs\[i\].addEventListener('input', updateStudio);  
      inputs\[i\].addEventListener('change', updateStudio);  
    }

    document.getElementById('presetSelect').addEventListener('change', function (e) {  
      applyPreset(e.target.value);  
    });

    var tabBtns \= document.querySelectorAll('.tab-btn');  
    for (var j \= 0; j \< tabBtns.length; j++) {  
      tabBtns\[j\].addEventListener('click', function (e) {  
        for (var k \= 0; k \< tabBtns.length; k++) tabBtns\[k\].classList.remove('active');  
        e.target.classList.add('active');  
        currentTab \= e.target.getAttribute('data-tab');  
        updateStudio();  
      });  
    }

    document.getElementById('copyConfigBtn').addEventListener('click', function () {  
      var text \= document.getElementById('configOutput').textContent;  
      navigator.clipboard.writeText(text).then(function () {  
        var btn \= document.getElementById('copyConfigBtn');  
        btn.textContent \= 'Copied\!';  
        setTimeout(function () { btn.textContent \= 'Copy Config'; }, 2000);  
      });  
    });

    document.getElementById('exportJsonBtn').addEventListener('click', function () {  
      var cfg \= getFormConfig();  
      var jsonStr \= JSON.stringify(cfg, null, 2);  
      var blob \= new Blob(\[jsonStr\], { type: 'application/json' });  
      var url \= URL.createObjectURL(blob);  
      var a \= document.createElement('a');  
      a.href \= url;  
      a.download \= (cfg.locatorName || 'srv6-config') \+ '.json';  
      a.click();  
      URL.revokeObjectURL(url);  
    });

    // Initial render  
    updateStudio();  
  });  
})();  
