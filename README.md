# nginx-vm Azure Deployment

An Azure ARM template that provisions a Ubuntu 24.04 LTS virtual machine with nginx installed and running.

## Overview

This template deploys the following Azure resources:

- **Virtual Machine** – Ubuntu 24.04 LTS (`Standard_D2s_v3` by default)
- **Network Interface** – connects the VM to the VNet and public IP
- **Public IP Address** – static Standard SKU IP for external access
- **Virtual Network** – `10.0.0.0/16` address space with a `10.0.0.0/24` subnet
- **Network Security Group** – attached to the subnet
- **Custom Script Extension** – installs and enables nginx on first boot

## Prerequisites

- An active Azure subscription
- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli) installed, or access to the Azure Portal
- An SSH key pair (the public key is passed as a parameter)

## Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `adminUsername` | string | *(required)* | Admin username for the VM |
| `adminPublicKey` | securestring | *(required)* | SSH public key for the admin user |
| `allowedIPAddress` | string | *(required)* | IP address allowed to access the VM |
| `vmSize` | string | `Standard_D2s_v3` | Azure VM size |
| `location` | string | Resource group location | Azure region for all resources |

## Deployment

### Azure CLI

```bash
az group create --name <resource-group> --location <location>

az deployment group create \
  --resource-group <resource-group> \
  --template-file deploy.json \
  --parameters \
      adminUsername=<your-username> \
      adminPublicKey="$(cat ~/.ssh/id_rsa.pub)" \
      allowedIPAddress=<your-ip>
```

### Azure Portal

1. Navigate to **Deploy a custom template** in the Azure Portal.
2. Click **Build your own template in the editor** and paste the contents of `deploy.json`.
3. Fill in the required parameters and click **Review + create**.

## Outputs

After a successful deployment, the template returns:

| Output | Description |
|---|---|
| `publicIPAddress` | The public IP address assigned to the VM |
| `sshCommand` | Ready-to-use SSH command to connect to the VM |

## Connecting to the VM

Use the `sshCommand` output value, or run:

```bash
ssh <adminUsername>@<publicIPAddress>
```

## ⚠️ Security Notice

The Network Security Group is currently configured with an **Allow All Inbound** rule (`*` on all ports and protocols). This is suitable for quick testing only.

Before deploying to any non-throwaway environment, replace the `Allow-All-Inbound` rule with rules that restrict access to only the ports and source addresses you need. For example:

```json
{
  "name": "Allow-SSH",
  "properties": {
    "priority": 100,
    "protocol": "Tcp",
    "access": "Allow",
    "direction": "Inbound",
    "sourceAddressPrefix": "<your-ip>/32",
    "sourcePortRange": "*",
    "destinationAddressPrefix": "*",
    "destinationPortRange": "22"
  }
}
```

The template already accepts an `allowedIPAddress` parameter — it is wired in but not yet applied to any NSG rule. Use it as the `sourceAddressPrefix` when tightening the rules.
