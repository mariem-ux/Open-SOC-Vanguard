# **1.Audit Engine Deployment**
### Install auditd
```bash
sudo apt install auditd -y
```
### Start the Service
```bash
sudo systemctl start auditd
```
### Enable auditd at Boot
```bash
sudo systemctl enable auditd
```
### Verify the Service is Running
```bash
sudo systemctl status auditd
```
### Confirm Live Recording is Active
```bash
sudo tail -5 /var/log/audit/audit.log
```
# **2.Rule Generation: Custom Audit Rules**
## Open the Audit Rules File and Write Custom Rules
### Rule 1 : Track All Commands Executed (exec-commands)
```bash
#Rule 1 : Track all commands executed (execve system calls )
-a always, exit -F arch=b64 -S execve -k exec_commands
-a always, exit -F arch=b32 -S execve -k exec_commands
```
### Rule 2 : Watch System Binary Folders (binary-modification)
```bash
#Rule 2 : watch for modifcations to system binaries
-w /usr/bin/ -p wa -k binary_modification
-w /usr/sbin -p wa -k binary modification
-w /bin/ -p wa -k binary_modification
```
### Rule 3 : Watch Critical User Account Files (passwd-changes)
```bash
#Rule3: watch for changes to /etc/passwd
-w /etc/passwd -p wa -k passwd_changes
-w /etc/shadow -p wa -k passwd_changes
-w /etc/sudoers -p wa -k passwd_changes
```
## Restart auditd to Apply the Rules

```bash
sudo systemctl restart auditd
```
## Verify the Rules Are Loaded
```bash
sudo auditctl -l
```
## Quick Test to Confirm It Works (on Lubuntu)
```bash
whoami
sudo tail -50 /var/log/audit/audit.log | grep exec_commands
```
# **3.SIEM Mapping: Connecting auditd to Wazuh**
## Locate the Wazuh Agent Configuration File
```bash
<localfile>
<log_format>syslog</log_format>
<location>/var/log/syslog</location>
</localfile>
```
## Add the auditd Log Ingestion Block
```bash
<localfile>
<log_format>syslog</log_format>
<location>/var/log/syslog</location>
</localfile>

<localfile>
<log_format>audit</log_format>
<loction>/var/log/audit/audit. log</location>
</localfile>
```
## Restart the Wazuh Agent to Apply the Changes
```bash
sudo systemctl restart wazuh-agent
sudo systemctl status wazuh-agent
```
# **4.Behavior Verification: End-to-End Pipeline Test**
## Test Command Execution Tracking
```bash
whoami
id
uname -a
```
## Test Privilege Escalation Tracking
```bash
sudo cat /etc/passwd
sudo ls /root
```
## Test Cron Job Modification Tracking
```bash
crontab -e
```
open the cron editor and add in the bottom 
```bash
@reboot echo ”soc test”
```
## Test the /etc/passwd Watch Rule
```bash
sudo touch /etc/passwd
```
## Verify All Alerts in the Wazuh Dashboard
```bash
sudo touch /etc/passwd
sudo touch /etc/shallow
sudo touch /etc/sudoers
sudo ausearch -k passwd_changes --start recent
```


























































