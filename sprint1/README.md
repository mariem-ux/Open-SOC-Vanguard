# **SIEM Installation**:

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
