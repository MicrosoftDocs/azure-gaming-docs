---
title: 'Quickstart: Deploy Unreal Cloud DDC - Bicep'
description: In this quickstart, you use Bicep to quickly create an Unreal Cloud DDC deployment.
author: dciborow
ms.topic: quickstart
ms.date: 08/11/2022
ms.author: dciborow
ms.service: azure-gaming
---

# Quickstart: Create an Unreal Cloud DDC using Bicep

[!INCLUDE [preview](./includes/preview.md)]

This quickstart will show you how to create an Unreal Cloud DDC deployment using Bicep.

## Review the Bicep file

The Bicep file is based on the quickstart from [Azure Quickstart Templates](https://azure.microsoft.com/resources/templates/horde-storage/).

```bicep
param name                     string = 'horde-cluster'
param agentPoolName            string = 'k8agent'
param location                 string = 'eastus2'
param managedResourceGroupName string = 'mrg'
param agentPoolCount           int    = 3

var managedResourceGroupId = '${subscription().id}/resourceGroups/${resourceGroup().name}-${managedResourceGroupName}'

resource hordeStorage 'Microsoft.Solutions/applications@2017-09-01' = {
  location: location
  kind: 'MarketPlace'
  name: name
  plan: {
    name: 'aks'
    product: 'horde-storage-preview'
    publisher: 'microsoft-azure-gaming'
    version: '0.0.4'
  }
  properties: {
    managedResourceGroupId: managedResourceGroupId
    parameters: {
      location: {
        value: location
      }
      name: {
        value: name
      }
      agentPoolCount: {
        value: agentPoolCount
      }
      agentPoolName: {
        value: agentPoolName
      }
    }
    jitAccessPolicy: null
  }
}
```

## Deploy the Bicep file

1. Save the Bicep file as **main.bicep** to your local computer.
1. Deploy the Bicep file using either [Azure CLI](/azure/azure-resource-manager/bicep/deploy-cli) or [Azure PowerShell](/azure/azure-resource-manager/bicep/deploy-powershell).

    #### [CLI](#tab/CLI)

    ```azurecli-interactive
    az group create \
      --name exampleRG \
      --location eastus
    az deployment group create \
      --resource-group exampleRG \
      --template-file main.bicep \
      --parameters name=<horde-storage-name>

    ```

    #### [PowerShell](#tab/PowerShell)

    ```azurepowershell-interactive
    New-AzResourceGroup -Name exampleRG -Location eastus
    New-AzResourceGroupDeployment -ResourceGroupName exampleRG -TemplateFile ./main.bicep -adminName "<admin-user>" -vmName "<vm-name>" 
    ```

    > [!NOTE]
    > Replace **\<admin-user\>** with the username for the administrator account. Replace **\<vm-name\>** with the name of your virtual machine.

    When the deployment finishes, you should see a message indicating the deployment succeeded.

## Review deployed resources

[Azure CLI](/azure-resource-manager/management/manage-resource-groups-cli#list-resource-groups),
or [Azure PowerShell](/azure/azure-resource-manager/management/manage-resource-groups-powershell#list-resource-groups) to list the deployed resources in the resource group.

#### [CLI](#tab/CLI)

```azurecli-interactive
az resource list \
  --resource-group managedResourceGroupId
```

#### [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Get-AzResource -ResourceGroupName managedResourceGroupId
```

---

## Clean up resources

When no longer needed, use the
[Azure CLI](/azure/azure-resource-manager/management/manage-resource-groups-cli#delete-resource-groups),
or [Azure PowerShell](/azure/azure-resource-manager/management/manage-resource-groups-powershell#delete-resource-groups) to delete the resource group and its resources.

#### [CLI](#tab/CLI)

```azurecli-interactive
az group delete \
  --name exampleRG
```

#### [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Remove-AzResourceGroup -Name exampleRG
```

---

## Next steps

* [Post deployment configuration to complete setup](deployment-setup.md)

## See also

* [Deploy Bicep files from Visual Studio Code](/azure/azure-resource-manager/bicep/deploy-vscode)
* [Configure your Bicep environment](/azure/azure-resource-manager/bicep/bicep-config)
* [Use Bicep linter](/azure/azure-resource-manager/bicep/linter)
* [Use functions in Bicep](/azure/azure-resource-manager/bicep/bicep-functions)
* [Use modules in Bicep](/azure/azure-resource-manager/bicep/modules)
