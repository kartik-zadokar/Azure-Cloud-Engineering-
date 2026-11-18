# Day 4 – Azure Standard Load Balancer & High Availability Lab

## Project Overview

Built a two-VM web application environment using Azure Virtual Machines, Virtual Network, Network Security Groups, and Azure Standard Load Balancer.

Configured HTTP health probes and load-balancing rules, tested traffic distribution, simulated an application failure, and verified recovery.

## Architecture

**Traffic flow:**

Internet → Azure Standard Load Balancer → Backend Pool → VM1 / VM2 → Nginx

### Azure Resources

| Resource | Name / Configuration |
|---|---|
| Resource Group | `rg-azure-lab-day4` |
| Region | Central India |
| Virtual Network | `vnet-day4-lab` |
| VNet Address Space | `10.0.0.0/16` |
| Subnet | `snet-app` |
| Subnet Address Range | `10.0.1.0/24` |
| Network Security Group | `nsg-day4-app` |
| Virtual Machine 1 | `vm-day4-01` |
| Virtual Machine 2 | `vm-day4-02` |
| Operating System | Ubuntu Server 24.04 LTS |
| Web Server | Nginx |
| Load Balancer | `lb-day4-app` |
| Load Balancer SKU | Standard |
| Frontend Configuration | `fe-day4-public` |
| Public IP | `pip-day4-lb` |
| Backend Pool | `bp-day4-vms` |
| Health Probe | `hp-day4-http` |
| Load Balancing Rule | `lbr-day4-http` |

## 1. Network Security Group Configuration

Created inbound security rules:

| Rule Name | Priority | Port | Protocol | Action |
|---|---:|---:|---|---|
| Allow-HTTP-Inbound | 100 | 80 | TCP | Allow |
| Allow-Azure-HealthProbe | 110 | 80 | TCP | Allow |
| Allow-SSH-Inbound | 120 | 22 | TCP | Allow |

SSH access was restricted to the administrator's public IP address using `/32`.

Associated the NSG with the application subnet.

## 2. Virtual Machine Deployment

Deployed two Ubuntu virtual machines in the application subnet.

Installed Nginx on both VMs.

```bash
sudo apt update
sudo apt install nginx -y
```

Verified Nginx:

```bash
sudo systemctl status nginx
curl -I http://localhost
```

Expected HTTP response:

```text
HTTP/1.1 200 OK
```

Configured different homepage responses to identify backend servers.

**VM1:**

```bash
echo "Hello from VM1 - Availability Zone 1" | sudo tee /var/www/html/index.html
```

**VM2:**

```bash
echo "Hello from VM2 - Availability Zone 1" | sudo tee /var/www/html/index.html
```

> Note: Verify the actual Availability Zone of each VM in Azure Portal before claiming zone-level redundancy. The homepage labels above are identifying text only.

## 3. Azure Standard Load Balancer

Created a public Standard Load Balancer.

### Frontend IP Configuration

- Name: `fe-day4-public`
- IP version: IPv4
- Public IP: `pip-day4-lb`
- SKU: Standard
- Availability: Zone-redundant, if supported and selected

### Backend Pool

Name: `bp-day4-vms`

Added both virtual machine network interfaces:

- `vm-day4-01`
- `vm-day4-02`

### Health Probe

Name: `hp-day4-http`

| Setting | Value |
|---|---|
| Protocol | HTTP |
| Port | 80 |
| Path | `/` |
| Interval | 5 seconds |
| Probe threshold | 2 |

The health probe checks whether each backend VM responds successfully over HTTP.

### Load Balancing Rule

Name: `lbr-day4-http`

| Setting | Value |
|---|---|
| Protocol | TCP |
| Frontend Port | 80 |
| Backend Port | 80 |
| Backend Pool | `bp-day4-vms` |
| Health Probe | `hp-day4-http` |
| Session Persistence | None |

## 4. Load Balancer Traffic Testing

Load Balancer frontend public IP:

`20.207.89.168`

Tested using:

```bash
for i in {1..15}; do
  curl -s --http1.1 \
    -H "Connection: close" \
    "http://20.207.89.168/?test=$RANDOM"
  echo
done
```

Observed responses from both backend VMs.

This confirmed that traffic was being distributed across the healthy backend pool.

## 5. Failure Simulation and Troubleshooting

Simulated an application failure by stopping Nginx on VM2.

```bash
sudo systemctl stop nginx
```

Waited for the health probe to detect the failure.

Tested Load Balancer traffic:

```bash
for i in {1..10}; do
  curl -s --max-time 5 "http://20.207.89.168/?test=$RANDOM"
  echo
done
```

**Observed result:**

All responses came from VM1.

This demonstrated that the Load Balancer stopped directing new traffic to the unhealthy backend.

## 6. Application Recovery

Restarted Nginx on VM2:

```bash
sudo systemctl start nginx
```

Verified the service:

```bash
sudo systemctl status nginx
curl -I http://localhost
```

Retested Load Balancer traffic.

**Observed result:**

Responses from both VM1 and VM2 appeared again.

This demonstrated backend recovery and re-entry into the healthy backend pool.

## 7. Key Concepts Learned

- Azure Virtual Networks and subnets
- Network Security Groups and inbound rules
- SSH access and secure connectivity
- Azure Virtual Machine deployment
- Nginx installation and service management
- Azure Standard Load Balancer
- Frontend IP configuration
- Backend pools
- HTTP health probes
- Load-balancing rules
- Backend health monitoring
- Application failure simulation
- Health-based traffic routing
- Application recovery and troubleshooting

## 8. Troubleshooting Commands

```bash
# Check Nginx service
sudo systemctl status nginx

# Start Nginx
sudo systemctl start nginx

# Stop Nginx
sudo systemctl stop nginx

# Restart Nginx
sudo systemctl restart nginx

# Check local HTTP response
curl -I http://localhost

# Check listening ports
sudo ss -tulpn

# Check Nginx logs
sudo journalctl -u nginx --no-pager -n 50
```

## 9. Project Outcome

Successfully deployed two Nginx web servers behind an Azure Standard Load Balancer.

Verified traffic distribution, simulated an application failure, observed health-probe-based traffic exclusion, and restored the failed application.

**Status:** Core Load Balancer and failure-recovery lab completed.

## 10. Cost Management

This was a hands-on Pay-As-You-Go Azure lab.

After documenting the project:

- Delete the lab Resource Group when no longer needed.
- Verify that all associated resources have been removed.
- Review Azure Cost Management for any remaining charges.

---

**Project:** Azure Cloud Engineering – Day 4  
**Focus:** Azure Networking, Load Balancing, Health Probes and Troubleshooting  
**Environment:** Microsoft Azure + Ubuntu Linux + Nginx
