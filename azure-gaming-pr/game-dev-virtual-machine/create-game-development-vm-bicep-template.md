---
title: 'Quickstart: Create an Game Developer VM - Bicep'
description: In this quickstart, you use Bicep to quickly deploy a Game Developer Virtual Machine
author: dciborow
ms.topic: quickstart
ms.date: 08/11/2022
ms.author: dciborow
ms.service: azure-gaming
---

# Quickstart: Create an Game Developer Virtual Machine using Bicep

> [!IMPORTANT]
> [!INCLUDE [reminder](includes/game-dev-banner.md)]

This quickstart will show you how to create a Windows 10 Game Developer Virtual Machine using Bicep. Game Developer Virtual Machines are cloud-based virtual machines preloaded with a suite of game developer frameworks and tools. When deployed on GPU-powered compute resources, all tools and libraries are configured to use the GPU.

## Prerequisites

An Azure subscription. If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/services/machine-learning/) before you begin.

## Review the Bicep file

The Bicep file is based on the quickstart from [Azure Quickstart Templates](https://azure.microsoft.com/resources/templates/gamedev-vm/).

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
    adminName : 'gdvmadmin'
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
1. Deploy the Bicep file using either [Azure CLI](/azure/azure-resource-manager/bicep/deploy-cli) or [Azure PowerShell](/azure/azure-resource-manager/bicep/deploy-powershell).

    # [CLI](#tab/CLI)

    ```azurecli
    az group create \
      --name exampleRG \
      --location eastus
    az deployment group create \
      --resource-group exampleRG \
      --template-file main.bicep \
      --parameters adminName=<admin-user> \
      vmName=<vm-name>

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

Use the
[Azure CLI](/azure-resource-manager/management/manage-resource-groups-cli#list-resource-groups),
or [Azure PowerShell](/azure/azure-resource-manager/management/manage-resource-groups-powershell#list-resource-groups) to list the deployed resources in the resource group.

# [CLI](#tab/CLI)

```azurecli-interactive
az resource list \
  --resource-group exampleRG
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Get-AzResource -ResourceGroupName exampleRG
```

---

## Clean up resources

When no longer needed, use the
[Azure CLI](/azure/azure-resource-manager/management/manage-resource-groups-cli#delete-resource-groups),
or [Azure PowerShell](/azure/azure-resource-manager/management/manage-resource-groups-powershell#delete-resource-groups) to delete the resource group and its resources.

# [CLI](#tab/CLI)

```azurecli-interactive
az group delete \
  --name exampleRG
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Remove-AzResourceGroup -Name exampleRG
```

---

## See also
- [Deploy Bicep files from Visual Studio Code](/azure/azure-resource-manager/bicep/deploy-vscode)
- [Configure your Bicep environment](/azure/azure-resource-manager/bicep/bicep-config)
- [Use Bicep linter](/azure/azure-resource-manager/bicep/linter)
- [Use functions in Bicep](/azure/azure-resource-manager/bicep/bicep-functions)
- [Use modules in Bicep](/azure/azure-resource-manager/bicep/modules)
