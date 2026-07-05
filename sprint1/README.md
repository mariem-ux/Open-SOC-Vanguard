# **1.SIEM Installation**:

## System Update:
```bash
sudo apt update && sudo apt upgrade -y
```
## Downloading the Official Wazuh Installation Script
```bash
curl -s0 https://packages.wazuh.com/4.14/wazuh-install.sh
```
## Running the All-in-One Installer
```bash
sudo bash ./wazuh-install.sh -a
```
## Verifying All Three Wazuh Services
### wazuh-manager :
```bash
sudo sustemctl status wazuh-manager
```
### wazuh-indexer:
```bash
sudo sustemctl status wazuh-indexer
```
### wazuh-dashboard
```bash
sudo sustemctl status wazuh-dashboard
```
# **2.Agent Enrolement**

## Opening Required Ports on the Wazuh Server
```bash
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
```
## Installing the Wazuh Agent on LubuntuVM

### Downloading the Agent Package on the Wazuh Server
```bash
curl -L -O wazuh-agant.deb https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.5-1_amd64.deb
```
### Verifying the Downloaded Package
```bash
file wazuh-agent.deb
```
### Installing ,Enabling SSH on LubuntuVM and running SSH on LubuntuVM for File Transfer              
```bash
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```
### Transferring the Agent Package to LubuntuVM
```bash
scp wazuh-agent.deb mariem@192.168.2.128:/home/mariem/
```
### Installing the Agent on LubuntuVM
```bash
sudo WAZUH_MANAGER='192.168.2.10' dpkg -i wazuh-agent.deb
```
## Starting the Wazuh Agent Service on LubuntuVM
```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```
## Verifying the Agent is Running on LubuntuVM
```bash
sudo systemctl status wazuh-agent
```
## Confirming Agent Registration on the Wazuh Server
```bash
sudo /var/ossec/bin/agent_control -l
```

# **3.Network Log Integration**
## Configuring the Wazuh Manager to Receive Syslog fromOPNsense
```bash
sudo nano /var/ossec/etc/ossec.conf
<remote>
<connection>syslog</connection>
<port>514</port>
<protocol>udp</protocol>
<allowed-ips> 192.168.2.1</allowed-ips>
</remote>
```
## Restarting the Wazuh Manager
```bash
sudo systemctl restart wazuh-manager
sudo systemctl status wazuh-manager
```
## Verifying Log Reception on Wazuh
### Confirming Wazuh is Listening on Port 514
```bash
sudo ss -ulnp | grep 514
```
### Capturing Live Syslog Packets from OPNsense
```bash
sudo tcpdump -i any port 514 -n
```
### Verifing Wazuh is receiving and parsing the logs 
```bash
sudo tail -f  /var/ossec/logs/ossec.log
```
### Confirming Real-Time Security Alerts are Being Generated
```bash
sudo tail -f  /var/ossec/logs/alerts/alerts.log
```
### End-to-End Pipeline Validation : Triggering Live Network Alerts
```bash
sudo systemctl start wazuh-dashboard
sudo systemctl status wazuh-dashboard
```
on kali :
```bash
nmap -sS 192.168.2.128
```
or 
```bash
ping -c 10 192.168.2.10
```
# **Bonus : API Log Ingestion**
### Installing the Required Python Library
```bash
pip3 install requests
```
### the wazuh report : 
```python
import requests
import urllib3
urllib3.disable_warnings()

WAZUH_API = "https://localhost:55000"
USER = "wazuh"
PASSWORD = "NewPassword123*"

def get_token():
    r = requests.post(
        f"{WAZUH_API}/security/user/authenticate",
        auth=(USER, PASSWORD),
        verify=False
    )
    return r.json()['data']['token']

def get_alerts(token):
    headers = {"Authorization": f"Bearer {token}"}
    r = requests.get(
        f"{WAZUH_API}/manager/logs/summary",
        headers=headers,
        verify=False
    )
    return r.json()

def parse_alerts(data):
    results = []
    items = data['data']['affected_items'][0]
    # Step 1 : Loop
    for component, stats in items.items():
        # Step 2 : Extract
        if stats['error'] > 0 or stats['warning'] > 0 or stats['critical'] > 0:
            results.append({
                "component": component,
                "errors": stats['error'],
                "warnings": stats['warning'],
                "critical": stats['critical']
            })
    return results

token = get_token()
data = get_alerts(token)
# Step 3 : Store and display
alerts = parse_alerts(data)
print(f"\nFound {len(alerts)} components with issues\n")
for a in alerts:
    print(f"- {a['component']} | errors:{a['errors']} warnings:{a['warnings']} critical:{a['critical']}")


from datetime import datetime

def generate_report(alerts):
    print("\n# Wazuh High-Severity Report")
    print(f"**Generated:** {datetime.now().strftime('%Y-%m-%d %H:%M')}\n")
    print("| Component | Errors | Warnings | Critical |")
    print("|-----------|--------|----------|----------|")
    if not alerts:
        print("| No issues found | 0 | 0 | 0 |")
    else:
        for a in alerts:
            print(f"| {a['component']} | {a['errors']} | {a['warnings']} | {a['critical']} |")

generate_report(alerts)
```
























