# Azure ARM Template – Secure NGINX Virtual Machine Deployment

## Overview

This project deploys a secure Ubuntu Linux Virtual Machine on Microsoft Azure using an ARM (Azure Resource Manager) template.

The deployment automatically:

- Creates a Virtual Network (VNet)
- Creates a subnet
- Creates a Network Security Group (NSG)
- Creates a Public IP Address
- Creates a Network Interface Card (NIC)
- Deploys an Ubuntu Virtual Machine
- Installs and configures NGINX automatically
- Restricts access to only one allowed public IP address

---

# Features

## Security Features

- SSH access restricted to a single IP address
- HTTP access restricted to a single IP address
- Password authentication disabled
- SSH key authentication enabled

## Infrastructure Features

- Ubuntu 24.04 LTS Virtual Machine
- Static Public IP
- Automated NGINX installation
- Custom Script Extension for server setup
- Modular ARM template structure

---

# Architecture

The deployment creates the following Azure resources:

| Resource | Purpose |
|---|---|
| Virtual Network | Provides private networking |
| Subnet | Segments network resources |
| Network Security Group | Controls inbound traffic |
| Public IP Address | Provides internet access |
| Network Interface | Connects VM to the network |
| Ubuntu VM | Hosts the NGINX web server |
| Custom Script Extension | Automates server configuration |

---

# Requirements

Before deployment, ensure you have:

- Microsoft Azure Subscription
- Azure CLI installed
- SSH public/private key pair
- Resource Group created

---

# Deployment Steps

## 1. Login to Azure

```bash
az login
```

---

## 2. Create a Resource Group

```bash
az group create \
  --name rg-nginx \
  --location westeurope
```

---

## 3. Get Your Public IP Address

```bash
curl ifconfig.me
```

Example Output:

```bash
102.88.45.10
```

---

## 4. Deploy the ARM Template

```bash
az deployment group create \
  --resource-group rg-nginx \
  --template-file azuredeploy.json \
  --parameters adminUsername=azureuser \
  --parameters adminPublicKey="$(cat ~/.ssh/id_rsa.pub)" \
  --parameters allowedIPAddress="102.88.45.10/32"
```

---

# Accessing the VM

After deployment completes:

## SSH into the VM

```bash
ssh azureuser@PUBLIC_IP
```

---

# Test NGINX

## Using Browser

```text
http://PUBLIC_IP
```

## Using Curl

```bash
curl http://PUBLIC_IP
```

You should see the default NGINX welcome page.

---

# Network Security Rules

| Port | Access |
|---|---|
| 22 (SSH) | Allowed only from your IP |
| 80 (HTTP) | Allowed only from your IP |

All other inbound traffic is blocked.

---

# Project Structure

```text
project-folder/
│
├── azuredeploy.json
├── README.md
```

---

# Outputs

The ARM template returns:

- Public IP Address
- SSH command

Example:

```bash
ssh azureuser@20.50.xx.xx
```

---

# Technologies Used

- Microsoft Azure
- Azure Resource Manager (ARM)
- Ubuntu Linux
- NGINX
- Azure CLI
- Bash Scripting

---

# Troubleshooting

## VM Size Not Available

Try changing:

```json
"vmSize": "Standard_B1s"
```

or another supported SKU in your region.

---

## Quota Errors

Check Azure VM quota limits:

```bash
az vm list-usage --location westeurope
```

---

## SSH Connection Refused

Verify:

- NSG rules
- Correct public IP
- Correct SSH key
- VM status

---

# Author

Okechukwu Orjionuchie

---

# License

This project is for educational and cloud practice purposes.
