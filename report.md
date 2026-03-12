# THREAT HUNTING REPORT

## EXECUTIVE SUMMARY:
`On May 1, 2020, suspicious activities were detected on the systems within the organization, specifically involving the process `CollectGuestLogs.exe`. The analysis indicates potential misuse of this data collection tool, which may lead to unauthorized data exfiltration. Immediate containment actions are recommended to mitigate any potential threats.

## TECHNICAL FINDINGS:
The threat hunting analysis revealed multiple suspicious events associated with the execution of `CollectGuestLogs.exe` on the host `NASHUA.dmevals.local`. The following key findings were noted:

1. **Process Creation**: 
   - `CollectGuestLogs.exe` was executed with a command line indicating data collection, raising concerns about potential data exfiltration.
   - Subsequent invocation of `cmd.exe` by `CollectGuestLogs.exe` suggests an attempt to execute commands that could facilitate unauthorized access or data exfiltration.

2. **Repeated Execution**: 
   - The repeated invocation of `cmd.exe` from `CollectGuestLogs.exe` on another host (`UTICA.dmevals.local`) further indicates potential malicious activity or misuse of administrative privileges.

3. **Enriched Context**: 
   - Additional logs within a 5-minute window show registry modifications and process access events that correlate with the suspicious activities, indicating further potential malicious behavior.

## ATTACK TIMELINE:
- **2020-05-01 23:22:31**: `CollectGuestLogs.exe` executed on `NASHUA.dmevals.local` with a command line indicating data collection.
- **2020-05-01 23:22:41**: `cmd.exe` initiated by `CollectGuestLogs.exe` on `NASHUA.dmevals.local`.
- **2020-05-01 23:23:14**: `cmd.exe` initiated again by `CollectGuestLogs.exe` on `UTICA.dmevals.local`.
- **2020-05-02 00:58:07**: File created by `svchost.exe` related to `WMIPRVSE.EXE`.
- **2020-05-02 03:22:12**: Registry value set by `Explorer.EXE`.
- **2020-05-02 03:22:31**: `CollectGuestLogs.exe` executed again, indicating continued activity.
- **2020-05-02 03:22:36**: Registry keys created by `CollectGuestLogs.exe`, indicating further modifications to system settings.

## AFFECTED ASSETS:
- **Systems**: 
  - `NASHUA.dmevals.local`
  - `UTICA.dmevals.local`
- **Accounts**: 
  - `NT AUTHORITY\SYSTEM`
- **Processes**: 
  - `CollectGuestLogs.exe`
  - `cmd.exe`
  - `svchost.exe`
  - `Explorer.EXE`
  - `WMIPRVSE.EXE`

## MITRE ATT&CK SUMMARY:
- **Tactic**: Exfiltration
  - **Technique ID**: T1041
  - **Technique Name**: Exfiltration Over Command and Control Channel
- **Tactic**: Execution
  - **Technique ID**: T1059
  - **Technique Name**: Command and Scripting Interpreter

## KQL HUNTING QUERIES:
1. **Query Name**: Detect `CollectGuestLogs.exe` Execution
   ```kql
   process where process.name == "CollectGuestLogs.exe" and process.command_line contains "-Mode:ga"
   ```
   - **Detection**: Identifies instances of `CollectGuestLogs.exe` being executed with specific command line arguments indicating data collection.

2. **Query Name**: Detect `cmd.exe` Invocations
   ```kql
   process where process.name == "cmd.exe" and process.parent.name == "CollectGuestLogs.exe"
   ```
   - **Detection**: Captures instances where `cmd.exe` is initiated by `CollectGuestLogs.exe`, indicating potential command execution for malicious purposes.

3. **Query Name**: Registry Modifications by `CollectGuestLogs.exe`
   ```kql
   registry where image == "C:\\WindowsAzure\\Packages\\CollectGuestLogs.exe"
   ```
   - **Detection**: Monitors registry changes made by `CollectGuestLogs.exe`, which may indicate attempts to modify system settings for malicious purposes.

4. **Query Name**: Process Access Events
   ```kql
   process where event.type == "access" and source.process.name == "svchost.exe"
   ```
   - **Detection**: Identifies access events involving `svchost.exe`, which may indicate lateral movement or privilege escalation attempts.

## RECOMMENDATIONS:
1. **Immediate Containment**:
   - Isolate affected systems (`NASHUA.dmevals.local` and `UTICA.dmevals.local`) from the network to prevent further data exfiltration.
   - Terminate the `CollectGuestLogs.exe` and `cmd.exe` processes on the affected systems.

2. **Investigation**:
   - Conduct a thorough investigation of the affected systems to identify any additional malicious artifacts or indicators of compromise.
   - Review user access logs and permissions to ensure no unauthorized access has occurred.

3. **Remediation**:
   - Remove any unauthorized software or processes identified during the investigation.
   - Implement stricter access controls and monitoring for sensitive data and administrative tools.

## OVERALL THREAT LEVEL: MEDIUM

The analysis indicates a medium threat level due to the potential for data exfiltration and misuse of administrative privileges. Immediate actions are necessary to mitigate risks and prevent further incidents.
