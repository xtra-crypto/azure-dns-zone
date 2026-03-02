
To make this professional and ready for GitHub, you should structure it as a **Infrastructure-as-Code (IaC) Demo Repository**.

Below is a complete template. You can create these files in your local folder and push them to your GitHub repository.

### 📂 Recommended Repository Structure
```text
azure-private-dns-demo/
│
├── README.md                 # Documentation
├── deploy-cli.sh             # Azure CLI Script
├── deploy-ps.ps1             # PowerShell Script
├── cleanup-cli.sh            # Cleanup Script (CLI)
└── cleanup-ps.ps1            # Cleanup Script (PowerShell)
```

---

### 1. 📄 `README.md`
*Copy this into your README file. It explains what the project does.*

```markdown
# Azure Private DNS Zone Multi-VNet Demo

This repository contains scripts to automate the deployment of an **Azure Private DNS Zone** linked to **two separate Virtual Networks**. This allows resources in both VNets to resolve the same internal domain names.

## 🚀 Features
- Creates a Resource Group.
- Deploys 2 Virtual Networks (VNet-A, VNet-B).
- Creates a Private DNS Zone (e.g., `internal.corp.com`).
- Links both VNets to the DNS Zone.
- Creates a sample DNS A-Record for testing.

## 🛠 Prerequisites
- **Azure CLI** OR **Azure PowerShell** installed.
- An active Azure Subscription.
- Permissions to create resources (Contributor or Owner).

## 📦 Usage

### Option 1: Azure CLI (Bash)
```bash
chmod +x deploy-cli.sh
./deploy-cli.sh
```

### Option 2: Azure PowerShell
```powershell
.\deploy-ps.ps1
```

## 🧪 How to Test
1. Deploy a VM into **VNet-A** and **VNet-B**.
2. SSH/RDP into either VM.
3. Run: `nslookup myapp.internal.corp.com`
4. You should see the IP address defined in the script.

## 🧹 Cleanup
To avoid incurring costs, delete the resources when finished.

**CLI:**
```bash
./cleanup-cli.sh
```

**PowerShell:**
```powershell
.\cleanup-ps.ps1
```
```

---

### 2. 📜 `deploy-cli.sh`
*Save this as a Bash script for Azure CLI users.*

```bash
#!/bin/bash
# Azure Private DNS Zone Deployment Script (CLI)

set -e # Exit on error

# --- CONFIGURATION ---
RESOURCE_GROUP="rg-dns-demo-001"
LOCATION="eastus"
DNS_ZONE_NAME="internal.corp.com"
VNET1_NAME="VNet-A"
VNET2_NAME="VNet-B"
VNET1_PREFIX="10.1.0.0/16"
VNET2_PREFIX="10.2.0.0/16"
SAMPLE_RECORD_IP="10.1.0.4"

echo "🔐 Checking Azure Login..."
az account show > /dev/null || { echo "Please run 'az login' first."; exit 1; }

echo "📦 Creating Resource Group: $RESOURCE_GROUP..."
az group create --name $RESOURCE_GROUP --location $LOCATION --output none

echo "🌐 Creating Virtual Network 1: $VNET1_NAME..."
az network vnet create \
    --resource-group $RESOURCE_GROUP \
    --name $VNET1_NAME \
    --address-prefix $VNET1_PREFIX \
    --subnet-name subnet1 \
    --subnet-prefix 10.1.0.0/24 \
    --output none

echo "🌐 Creating Virtual Network 2: $VNET2_NAME..."
az network vnet create \
    --resource-group $RESOURCE_GROUP \
    --name $VNET2_NAME \
    --address-prefix $VNET2_PREFIX \
    --subnet-name subnet1 \
    --subnet-prefix 10.2.0.0/24 \
    --output none

echo "🔍 Creating Private DNS Zone: $DNS_ZONE_NAME..."
az network private-dns zone create \
    --resource-group $RESOURCE_GROUP \
    --name $DNS_ZONE_NAME \
    --output none

echo "🔗 Linking $VNET1_NAME to DNS Zone..."
az network private-dns link vnet create \
    --resource-group $RESOURCE_GROUP \
    --zone-name $DNS_ZONE_NAME \
    --name "link-vnet-a" \
    --virtual-network $VNET1_NAME \
    --registration-enabled false \
    --output none

echo "🔗 Linking $VNET2_NAME to DNS Zone..."
az network private-dns link vnet create \
    --resource-group $RESOURCE_GROUP \
    --zone-name $DNS_ZONE_NAME \
    --name "link-vnet-b" \
    --virtual-network $VNET2_NAME \
    --registration-enabled false \
    --output none

echo "📝 Creating Sample A Record (myapp.$DNS_ZONE_NAME -> $SAMPLE_RECORD_IP)..."
az network private-dns record-set a create \
    --resource-group $RESOURCE_GROUP \
    --zone-name $DNS_ZONE_NAME \
    --name "myapp" \
    --ttl 3600 \
    --output none

az network private-dns record-set a add-record \
    --resource-group $RESOURCE_GROUP \
    --zone-name $DNS_ZONE_NAME \
    --record-set-name "myapp" \
    --ipv4-address $SAMPLE_RECORD_IP \
    --output none

echo "✅ Deployment Complete!"
echo "👉 Test with: nslookup myapp.$DNS_ZONE_NAME"
```

---

### 3. 📜 `deploy-ps.ps1`
*Save this as a PowerShell script for PS users.*

```powershell
# Azure Private DNS Zone Deployment Script (PowerShell)

# --- CONFIGURATION ---
$ResourceGroup = "rg-dns-demo-001"
$Location = "eastus"
$DnsZoneName = "internal.corp.com"
$Vnet1Name = "VNet-A"
$Vnet2Name = "VNet-B"
$Vnet1Prefix = "10.1.0.0/16"
$Vnet2Prefix = "10.2.0.0/16"
$SampleRecordIP = "10.1.0.4"

Write-Host "🔐 Checking Azure Login..." -ForegroundColor Cyan
if (-not (Get-AzContext)) {
    Write-Error "Please run 'Connect-AzAccount' first."
    exit
}

Write-Host "📦 Creating Resource Group: $ResourceGroup..." -ForegroundColor Cyan
New-AzResourceGroup -Name $ResourceGroup -Location $Location | Out-Null

Write-Host "🌐 Creating Virtual Network 1: $Vnet1Name..." -ForegroundColor Cyan
$vnet1 = New-AzVirtualNetwork `
    -ResourceGroupName $ResourceGroup `
    -Name $Vnet1Name `
    -AddressPrefix $Vnet1Prefix `
    -SubnetName "subnet1" `
    -SubnetAddressPrefix 10.1.0.0/24

Write-Host "🌐 Creating Virtual Network 2: $Vnet2Name..." -ForegroundColor Cyan
$vnet2 = New-AzVirtualNetwork `
    -ResourceGroupName $ResourceGroup `
    -Name $Vnet2Name `
    -AddressPrefix $Vnet2Prefix `
    -SubnetName "subnet1" `
    -SubnetAddressPrefix 10.2.0.0/24

Write-Host "🔍 Creating Private DNS Zone: $DnsZoneName..." -ForegroundColor Cyan
$zone = New-AzPrivateDnsZone `
    -ResourceGroupName $ResourceGroup `
    -Name $DnsZoneName

Write-Host "🔗 Linking $Vnet1Name to DNS Zone..." -ForegroundColor Cyan
New-AzPrivateDnsVirtualNetworkLink `
    -ResourceGroupName $ResourceGroup `
    -ZoneName $DnsZoneName `
    -Name "link-vnet-a" `
    -VirtualNetwork $vnet1.Id `
    -RegistrationEnabled $false | Out-Null

Write-Host "🔗 Linking $Vnet2Name to DNS Zone..." -ForegroundColor Cyan
New-AzPrivateDnsVirtualNetworkLink `
    -ResourceGroupName $ResourceGroup `
    -ZoneName $DnsZoneName `
    -Name "link-vnet-b" `
    -VirtualNetwork $vnet2.Id `
    -RegistrationEnabled $false | Out-Null

Write-Host "📝 Creating Sample A Record..." -ForegroundColor Cyan
New-AzPrivateDnsRecordSet `
    -ResourceGroupName $ResourceGroup `
    -ZoneName $DnsZoneName `
    -RecordType A `
    -Name "myapp" `
    -Ttl 3600 | Out-Null

Add-AzPrivateDnsRecordConfig `
    -RecordSet (Get-AzPrivateDnsRecordSet -ResourceGroupName $ResourceGroup -ZoneName $DnsZoneName -Name "myapp" -RecordType A) `
    -Ipv4Address $SampleRecordIP | `
    Set-AzPrivateDnsRecordSet | Out-Null

Write-Host "✅ Deployment Complete!" -ForegroundColor Green
Write-Host "👉 Test with: nslookup myapp.$DnsZoneName"
```

---

### 4. 🧹 `cleanup-cli.sh`
*Always provide a cleanup script for demos so people don't leave resources running.*

```bash
#!/bin/bash
RESOURCE_GROUP="rg-dns-demo-001"

echo "⚠️ Deleting Resource Group: $RESOURCE_GROUP..."
az group delete --name $RESOURCE_GROUP --yes --no-wait
echo "🗑️ Deletion started. Check Azure Portal for status."
```

---

### 5. 🧹 `cleanup-ps.ps1`

```powershell
$ResourceGroup = "rg-dns-demo-001"

Write-Host "⚠️ Deleting Resource Group: $ResourceGroup..." -ForegroundColor Yellow
Remove-AzResourceGroup -Name $ResourceGroup -Force -AsJob
Write-Host "🗑️ Deletion started. Check Azure Portal for status."
```

---
