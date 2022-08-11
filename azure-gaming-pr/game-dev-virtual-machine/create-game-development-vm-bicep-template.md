---
title: 'Quickstart: Create an Game Developer VM - Bicep'
titleSuffix: Azure Game Developer Virtual Machine
description: In this quickstart, you use Bicep to quickly deploy a Game Developer Virtual Machine
services: azure-gaming
author: sdciborow
ms.author: dciborow 
ms.date: 08/11/2022
ms.topic: quickstart
ms.service: azure-gaming
ms.custom: subject-armqs, mode-arm
---

# Quickstart: Create an Game Developer Virtual Machine using Bicep

This quickstart will show you how to create a Windows 10 Game Developer Virtual Machine using Bicep. Game Developer Virtual Machines are cloud-based virtual machines preloaded with a suite of game developer frameworks and tools. When deployed on GPU-powered compute resources, all tools and libraries are configured to use the GPU.

## Prerequisites

An Azure subscription. If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/services/machine-learning/) before you begin.

## Review the Bicep file

The Bicep file used in this quickstart is from [Azure Quickstart Templates](https://azure.microsoft.com/en-us/resources/templates/gamedev-vm/).

```bicep
param vmName string
param adminName string
@secure()
param adminPass string = newGuid()

module win10 'br/public:azure-gaming/game-dev-vm:1.0.1' = {
  name: 'win10_ue_4_27_2'
  params: {
    location  : resourceGroup().location
    vmName    : 'bicep'
    adminName : 'dcibadmin'
    adminPass : adminPass
    osType    : 'win10'
    gameEngine: 'ue_4_27_2'
    vmSize    : 'Standard_D4s_v3'
  }
}

outputs adminPass string = adminPass
```

## Deploy the Bicep file

1. Save the Bicep file as **main.bicep** to your local computer.
1. Deploy the Bicep file using either Azure CLI or Azure PowerShell.

    # [CLI](#tab/CLI)

    ```azurecli
    az group create --name exampleRG --location eastus
    az deployment group create --resource-group exampleRG --template-file main.bicep --parameters adminName=<admin-user> vmName=<vm-name>
    ```

    # [PowerShell](#tab/PowerShell)

    ```azurepowershell
    New-AzResourceGroup -Name exampleRG -Location eastus
    New-AzResourceGroupDeployment -ResourceGroupName exampleRG -TemplateFile ./main.bicep -adminName "<admin-user>" -vmName "<vm-name>" 
    ```

    ---

    > [!NOTE]
    > Replace **\<admin-user\>** with the username for the administrator account. Replace **\<vm-name\>** with the name of your virtual machine.

    When the deployment finishes, you should see a message indicating the deployment succeeded.

## Review deployed resources

Use the Azure portal, Azure CLI, or Azure PowerShell to list the deployed resources in the resource group.

# [CLI](#tab/CLI)

```azurecli-interactive
az resource list --resource-group exampleRG
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Get-AzResource -ResourceGroupName exampleRG
```

---

## Clean up resources

When no longer needed, use the Azure portal, Azure CLI, or Azure PowerShell to delete the resource group and its resources.

# [CLI](#tab/CLI)

```azurecli-interactive
az group delete --name exampleRG
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Remove-AzResourceGroup -Name exampleRG
```

---

