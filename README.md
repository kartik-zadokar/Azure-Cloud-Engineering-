# Day 01 – Azure Virtual Machine Deployment

## Objective
Deploy an Ubuntu Linux Virtual Machine on Microsoft Azure, connect using SSH, and configure an Nginx web server.

## Azure Services Used
- Azure Virtual Machines
- Azure Virtual Network
- Network Security Group (NSG)
- Azure Managed Disks
- Public IP Address

## Implementation Steps

1. Created an Azure Resource Group.
2. Deployed an Ubuntu Server 24.04 LTS VM.
3. Configured SSH authentication.
4. Configured networking and NSG rules.
5. Connected to the VM using SSH from WSL.
6. Updated Ubuntu packages.
7. Installed Nginx.
8. Verified Nginx service status.
9. Accessed the Nginx welcome page using the VM's public IP.

## Commands Used

```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl status nginx
sudo ss -tulpn | grep :80
curl -I http://localhost
```

## Troubleshooting

Verified:
- VM running status
- SSH connectivity
- Nginx service status
- Port 80 listening status
- NSG inbound security rules

## Learning Outcomes

- Understanding Azure VM deployment.
- Connecting to Linux VMs using SSH.
- Understanding Azure networking basics.
- Installing and managing Linux services.
- Basic cloud infrastructure troubleshooting.

## Security Considerations

- SSH access should be restricted to trusted IP addresses.
- Never upload private SSH keys.
- Avoid publishing sensitive infrastructure information.

## Project Status

Completed – Day 01 Azure VM Deployment.
