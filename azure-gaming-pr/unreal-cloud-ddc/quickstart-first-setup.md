---
title: 'Quickstart: Set up Unreal Cloud DDC by using the Azure portal'
description: In this quickstart, you'll learn how to set up and configure an Unreal Cloud DDC provider by using the Azure portal.
author: dciborow
ms.topic: quickstart
ms.date: 10/20/2022
ms.author: dciborow
ms.service: azure-gaming
---

# Setup for Unreal Cloud DDC

[!INCLUDE [preview](./includes/preview.md)]

This article, part one of two, describes one-time setup process before you can create an Unreal Cloud DDC managed application using the Azure portal. You only need to do this on the first time.

* Part one: Setup and configure resources. Accept terms of use. (Current article)
* Part two: [Manage identity and access](quickstart-first-setup-id.md)

## Prerequisites

* If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/) before you begin.
* To complete setup, you need to have **Contributor** and **RBACRoleAssignment**, or **Owner** access to the Azure subscription.
This requires specific permission within the Azure Active Directory tenant.
For more information, see [Manage access](../intro-to-azure/concept-manage-access.md) and [Azure built-in roles](/azure/role-based-access-control/built-in-roles).

## Setup and configure resources

1. Create a signed HTTPS certificate in Azure Key Vault. To create a signed HTTPS certificate with your public DNS server, see [Create and merge a certificate signing request in Key Vault](/azure/key-vault/certificates/create-certificate-signing-request?tabs=azure-portal)
1. Create a new Resource Group with Traffic Manager profile.
1. Then, point Traffic Manager to the DNS server. For instructions, see [Traffic Manager pointing to Internet domain](/azure/traffic-manager/traffic-manager-point-internet-domain) to forward your DNS traffic to the Azure Traffic Manager. Now, we can connect any deployment of Unreal Cloud DDC to this Profile, and do not have to change the DNS for new deployments.
1. Configure Azure Kubernetes Service (AKS) to use preview features using the code snipets below. For details, see [Register the 'EnableWorkloadIdentityPreview' feature flag](/azure/aks/workload-identity-deploy-cluster#register-the-enableworkloadidentitypreview-feature-flag).

Register the `EnableWorkloadIdentityPreview` feature flag by using the `az feature register` command, as shown in the following example.

```azurecli-interactive
    az feature register \
    --namespace "Microsoft.ContainerService" \
    --name "EnableWorkloadIdentityPreview"
```

It takes a few minutes for the status to show **Registered**. Verify the registration status by using the `az feature list` command:

```azurecli-interactive
az feature list \
    -o table \
    --query "[?contains(name, 'Microsoft.ContainerService/EnableWorkloadIdentityPreview')].{Name:name,State:properties.state}"
```

* When ready, refresh the registration of the *Microsoft.ContainerService* resource provider by using the [az provider register][az-provider-register] command:

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

### Accept terms of use

Using the Azure CLI, run the following script to accept Unreal Engine's terms for using the Unreal Cloud DDC feature.

```azurecli-interactive
az term accept \
    --publisher "microsoft-azure-gaming" \
    --product "unreal-cloud-ddc-preview" \
    --plan "preview"
```

## Next steps

* [Manage identity and access](quickstart-first-setup-id.md)
