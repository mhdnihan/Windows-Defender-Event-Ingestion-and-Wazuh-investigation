# Windows-Defender-Event-Ingestion-and-Wazuh-investigation
Investigation --  Windows Defender Event Ingestion and Wazuh investigation



Windows Defender Event Ingestion & Wazuh Investigation

Objective

To test whether Windows Defender security events generated on a Windows endpoint are collected and processed by Wazuh.

Lab Environment

* Endpoint: Windows 11 VM
* SIEM: Wazuh
* Endpoint telemetry: Windows Event Log + Sysmon
* Virtualization: VirtualBox
* Test: EICAR antivirus test file

Rule id            -    61102
Rule level         -    5
System Process id  -    2996
System Event Id    -    1801

Initial Test

A harmless EICAR test file was created on the Windows endpoint to trigger Windows Defender.

Windows Defender successfully detected the test file and displayed the detection in:

Windows Security → Virus & threat protection → Protection history

However, no corresponding Wazuh alert was initially observed.

Initial Finding

This indicated that Windows Defender was detecting the event locally, but the Defender event channel was not being ingested by the Wazuh agent.

Configuration Change

The Wazuh Windows agent configuration was updated to collect the Windows Defender Operational event channel:

<localfile>
  <location>Microsoft-Windows-Windows Defender/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>

The Wazuh agent was then restarted to load the updated configuration.

Validation

After the configuration change, a controlled Defender test was performed again.

The corresponding Windows event was subsequently visible in the Wazuh dashboard.

However, Wazuh classified the event under a generic:

Windows System Error

rather than presenting it as a clearly identified Windows Defender detection.

Investigation Result

The test demonstrated two separate stages of SIEM integration:

Before configuration:

EICAR Test
    ↓
Windows Defender Detection
    ↓
Windows Protection History
    ↓
Wazuh
    ✕ No corresponding event

After configuration:

EICAR Test
    ↓
Windows Defender Detection
    ↓
Windows Defender Event Log
    ↓
Wazuh Agent
    ↓
Wazuh Manager
    ↓
Event visible in Dashboard
    ↓
Generic Windows System Error classification


Rule id            -    61102
Rule level         -    5
System Process id  -    2996
System Event Id    -    1801

SOC Analysis

The endpoint security control was functioning correctly because Windows Defender detected the test file.

The initial Wazuh visibility gap was addressed by configuring the Windows Defender Operational event channel in the Wazuh agent.

The remaining issue is event classification/rule matching. The event reached Wazuh, but the displayed rule/description did not clearly identify it as a Defender detection.

This demonstrates an important SIEM concept:

Log ingestion and alert generation are separate stages.

A security product may successfully detect an event locally while the SIEM may initially have no visibility of it. Even after ingestion is enabled, the SIEM still needs appropriate decoding and rules to classify the event correctly.

SOC Workflow Demonstrated

Detection
   ↓
Log Collection
   ↓
SIEM Ingestion
   ↓
Alert/Rule Matching
   ↓
Triage
   ↓
Investigation
   ↓
Detection Improvement

Key Learning

This exercise demonstrated how to troubleshoot a SIEM visibility gap instead of assuming that every endpoint detection automatically becomes a SIEM alert.

The investigation identified that:

* Windows Defender generated the detection.
* The Defender event channel was initially not being collected by Wazuh.
* Wazuh agent configuration was updated.
* Defender events subsequently became visible in Wazuh.
* The resulting event still required appropriate rule classification for meaningful security alerting.

Note: The EICAR file was used strictly as an antivirus test artifact and does not represent real malware.
