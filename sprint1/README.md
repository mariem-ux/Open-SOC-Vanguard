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
scp wazuh-agent.deb User@192.168.2.128:/home/User/
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




















