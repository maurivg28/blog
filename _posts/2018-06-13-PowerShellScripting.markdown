---
title:  "PowerSHell"
date:   2018-06-13 19:30:00
categories: [Ansible]
tags: [PowerSHell, Azure, Cloud, Microsoft, DevOps]
---
Hoy en dia, para los que administramos ambientes de nube, alguna vez nos ha tocado mover maquinas virtuales y nos hemos encontramos con que al ejecutar dicha tarea aparecen varios errores.
Les voy a hablar sobre como copiar discos administrados, ya sea desde una suscripcion a otra o dentro de la misma suscripcion, desde un recurso a otro... Pero con el condimento de que vamos a mover una maquina creada desde un proveedor de terceros, por ejemplo Gitlab Comunity.

## Introducción ##

Mover discos administrados de maquinas virtuales y luego crear maquinas a partir de esos discos es una tarea sencilla utilizando powershell y el portal de azure. Pero hay un detalle no menor que es la creacion de una maquina virtual (de terceros) a partir de la copia de dicho disco.
Para ello les voy a mostrar una combinacion de scripts de powershell que use de Microsoft y los adapte a mis necesidades.

## Funcionamiento ##

Basicamente el script lo que hace es copiar el disco administrado a una nueva suscripcion o resource group. Luego a partir de ese disco va a crear una nueva maquina virtual, teniendo en cuenta que va a configurar el plan (Publisher, ImageName, SKU) para dicha VM, la cual no queda grabado cuando se intentan copiar maquinas de terceros.

## El script ##

```powershell
#Author: Mauricio Villagran
#This script is used to copy a disk managed from an RG or subscription and then creates a virtual machine to which the plan is configured.

CLS

#Provide the subscription Id of the subscription where managed disk exists
$sourceSubscriptionId = Read-Host "Source Subscription Id"

#Provide the name of your resource group where managed disk exists
$sourceResourceGroupName = Read-Host "Source Resource Group Name"

#Provide the name of the managed disk
$managedDiskName = Read-Host "Managed Disk Name"

#Login to Azure Cloud
Login-AzureRmAccount -Subscription $sourceSubscriptionId

#Set the context to the subscription Id where Managed Disk exists
Select-AzureRmSubscription -SubscriptionId $sourceSubscriptionId

#Get the source managed disk
$managedDisk = Get-AzureRMDisk -ResourceGroupName $sourceResourceGroupName -DiskName $managedDiskName

#Provide the subscription Id of the subscription where managed disk will be copied to
#If managed disk is copied to the same subscription then you can skip this step
$targetSubscriptionId = Read-Host "Target Subscription Id"

#Name of the resource group where snapshot will be copied to
$targetResourceGroupName = Read-Host "Target Resource Group Name"

#Set the context to the subscription Id where managed disk will be copied to
#If snapshot is copied to the same subscription then you can skip this step
Select-AzureRmSubscription -SubscriptionId $targetSubscriptionId

$diskConfig = New-AzureRmDiskConfig -SourceResourceId $managedDisk.Id -Location $managedDisk.Location -CreateOption Copy 

#Create a new managed disk in the target subscription and resource group
$disk = New-AzureRmDisk -Disk $diskConfig -DiskName $managedDiskName -ResourceGroupName $targetResourceGroupName

#####
Write-Host -ForegroundColor Green "Copying Managed Disk..."

#####

$VirtualMachineName = Read-Host "New Virtual Machine Name"
$virtualNetworkName = Read-Host "New Virtual Network Name"
$virtualMachineSize = Read-Host "Virtual Machine Size"
$location = 'eastus'
$ImageName = 'gitlab-ce'
$ProductName = 'gitlab-ce'
$Publisher = 'gitlab'


Write-Host -ForegroundColor Green "Starting VM Creation"

#Initialize virtual machine configuration
$VirtualMachine = New-AzureRmVMConfig -VMName $VirtualMachineName -VMSize $virtualMachineSize

### Set Plan
Set-AzureRmVMPlan -VM $VirtualMachine -Name $ImageName -Product $ProductName -Publisher $Publisher

#Use the Managed Disk Resource Id to attach it to the virtual machine. Please change the OS type to linux if OS disk has linux OS
$VirtualMachine = Set-AzureRmVMOSDisk -VM $VirtualMachine -ManagedDiskId $disk.Id -CreateOption Attach -Linux

#Create a public IP for the VM
$publicIp = New-AzureRmPublicIpAddress -Name ($VirtualMachineName.ToLower() + '_ip') -ResourceGroupName $targetResourceGroupName -Location $location -AllocationMethod Dynamic

#Create the virtual network subnet
$Subnet = New-AzureRmVirtualNetworkSubnetConfig -Name default -AddressPrefix "10.0.16.0/24"

#Create the virtual network
$vnet = New-AzureRmVirtualNetwork -Name $virtualNetworkName -ResourceGroupName $targetResourceGroupName -Location $location -AddressPrefix '10.0.0.0/16' -Subnet $Subnet

#Get the virtual network where virtual machine will be hosted
$vnet = Get-AzureRmVirtualNetwork -Name $virtualNetworkName -ResourceGroupName $targetResourceGroupName

# Create NIC in the first subnet of the virtual network
$nic = New-AzureRmNetworkInterface -Name ($VirtualMachineName.ToLower() + '_nic') -ResourceGroupName $targetResourceGroupName -Location $location -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $publicIp.Id

$VirtualMachine = Add-AzureRmVMNetworkInterface -VM $VirtualMachine -Id $nic.Id

#Create the virtual machine with Managed Disk
New-AzureRmVM -VM $VirtualMachine -ResourceGroupName $targetResourceGroupName -Location $location
```
