---
title: Manage identity and access
description: Manage identity and access
author: dciborow
ms.topic: quickstart
ms.date: 10/20/2022
ms.author: dciborow
ms.prod: azure-gaming
---

# Manage identity and access

[!INCLUDE [retire-banner](./includes/retire-banner.md)]

This article, part two of two, describes one-time setup process before you can create an Unreal Cloud DDC managed application using the Azure portal.
You only need to do this on the first time.

* Part one: [Setup and configure resources. Accept terms of use.](quickstart-first-setup.md)
* Part two: Manage identity and access (current article)

In this article, you create three Service Principals to be used as Application, Client and Worker identities.
These Service Principals only need to be configured once during setup and can be reused by newer versions of the Unreal DDC managed application.

## Create an Application Identity

Create a new service principal for the deployment and set role access for the different user types.
It must be single-tenant.
This identity is used to manage user access to the Unreal Cloud DDC deployment.

```azurecli-interactive
echo "Name of Application Identity? (Type and press enter to continue)" && read -r APP_NAME
APP_ID=$(az ad app create --display-name $APP_NAME --sign-in-audience AzureADMyOrg --query appId --output tsv)

cat <<EOF > roles.json 
[{
    "allowedMemberTypes": [ "User", "Application"],
    "description": "Ability to replicate data across instances of Horde Storage",
    "displayName": "Replication Full Access",
    "isEnabled": true,
    "value": "transactionLog.full"
},
{
    "allowedMemberTypes": [ "User", "Application"],
    "description": "Admin access to Horde",
    "displayName": "Admin",
    "isEnabled": true,
    "value": "Admin"
},
{
    "allowedMemberTypes": [ "User", "Application"],
    "description": "Full access to Horde storage",
    "displayName": "Storage Full Access",
    "isEnabled": true,
    "value": "Storage.full"
},
{
    "allowedMemberTypes": [ "User", "Application"],
    "description": "Full access to Horde cache data",
    "displayName": "Cache Full Access",
    "isEnabled": true,
    "value": "Cache.full"
}]
EOF

az ad app update \
    --id $APP_ID \
    --app-roles @roles.json


cat <<EOF > claims.json 
{
    "api": {
        "oauth2PermissionScopes": {
            "value": "Horde.ReadWrite",
            "type": "User",
            "adminConsentDisplayName": "Read and Write Horde Data",
            "adminConsentDescription": "Allows the app to read and update Unreal Horde data on behalf of the signed in user",
            "userConsentDisplayName": "Read and Write Horde Data",
            "userConsentDescription": "Allows the app to read and update Unreal Horde data on your behalf",
            "isEnabled": true
        }
    }
}
EOF
# Stopped working per https://github.com/Azure/azure-cli/issues/23969
# az ad app update \
#     --id $APP_ID \
#     --set oauth2PermissionScopes=@claims.json
#
# az ad app update \
#     --id $APP_ID \
#     --identifier-uris api://$APP_ID
```

There is currently a bug trying to set **oauth2PermissionScopes**, and it must be set manually instead.
Navigate to the **Exposed an API** blade, and **Add a scope** using the following values.

* Scope Name: Horde.ReadWrite
* Who can consent?: Admins and users
* Admin Consent Display Name: Read and Write Horde Data
* Admin Consent Description: Allows the app to read and update Unreal Horde data on behalf of the signed in user
* User Consent Display Name: Read and Write Horde Data
* User Consent Description: Allows the app to read and update Unreal Horde data on your behalf
* isEnabled: true

## Create a Client Identity

Create a new Service Principal as the client for the deployment.
This will be used in the Visual Studio configuration files which will be added to the project.

```azurecli-interactive
echo "Name of Client Identity? (Type and press enter to continue)" && read -r CLIENT_NAME
CLIENT_ID=$(az ad app create --display-name $CLIENT_NAME --sign-in-audience AzureADMyOrg --query appId --output tsv)

cat <<EOF > redirectUris.json 
{
    "redirectUris": [
      "http://localhost:6552/callback",
      "https://login.microsoftonline.com/common/oauth2/nativeclient"
    ]
}
EOF
az ad app update \
    --id $CLIENT_ID \
    --web-redirect-uris @redirectUris.json

APP_ID=$(az ad app create --display-name $APP_NAME --sign-in-audience AzureADMyOrg --query appId --output tsv)
OAUTH2_ID=$(az ad app create --display-name $APP_NAME --sign-in-audience AzureADMyOrg --query api.oauth2PermissionScopes[0].id --output tsv)

cat <<EOF > requiredResourceAccess.json 
[
    {
      "resourceAccess": [
        {
          "id": "e1fe6dd8-ba31-4d61-89e7-88639da4683d",
          "type": "Scope"
        }
      ],
      "resourceAppId": "00000003-0000-0000-c000-000000000000"
    }
]
EOF

az ad app update \
    --id $CLIENT_ID \
    --required-resource-accesses @requiredResourceAccess.json
```

After running the above script, follow the below steps to complete the configuration.

1. In the **Authorizations** blade, "Add a platform", select the "Mobile and Desktop Applications", and provide a callback URL.
1. In the **API permissions** blade, select **Add api permission**
1. Select **APIs my organization uses** and search for the name of the Application Identity created in the previous step.
1. Select **Delegated permissions**, and select the "Horde.ReadWrite". Then finish by clicking **Add Permissions**
1. Admin consent must be provided for to complete the setup.
Contact your team's Active Directory administrator for information to complete this approval.

## Create a Worker Identity

The third service principal is used by the worker roles to enable cross-region replication.

```azurecli-interactive
echo "Name of Client Identity? (Type and press enter to continue)" && read -r WORKER_NAME
WORKER_ID=$(az ad app create --display-name $WORKER_NAME --sign-in-audience AzureADMyOrg --query appId --output tsv)
```

1. Under **API Permissions** add permissions from the Application Identity, and select the roles `HordeStorage.Cache` and `HordeStorage.Storage`.

## Configure access

### Application Identity

Configure access using [Azure portal](https://ms.portal.azure.com/).

1. Under **API Permissions** add the `Microsoft.Graph` permission `User.Read`
1. Under **Expose an API**, set the Application ID URI to `api://<insert-client-id>`, and add a new scope with the following configurations.
    * **Scope name**, select **Horde.ReadWrite**
    * **Who can consent?**, select **Admins and users**
    * **Admin consent display name**, select  **Read and Write Horde Data**
    * **Admin consent description**, select **Allows the app to read and update Unreal Horde data on behalf of the signed in user**
    * **User consent display name**, select **Read and Write Horde Data**
    * **State**, select **Enabled**
1. Under **App roles**, add the following four roles to manage service access: **Replication Full Access**, **Admin**, **Storage Full Access**, and **Cache Full Access**. You can also use the Azure CLI to add them.

    ```azurecli-interactive
    az ad app update \
        --id e042ec79-34cd-498f-9d9f-123456781234 \
        --required-resource-accesses @manifest.json
    ```

    manifest.json

    ```json
    [{
        "allowedMemberTypes": [
        "User",
        "Application"
        ],
        "description": "Ability to replicate data across instances of Horde Storage",
        "displayName": "Replication Full Access",
        "isEnabled": true,
        "value": "transactionLog.full"
    },
    {
        "allowedMemberTypes": [
        "User",
        "Application"
        ],
        "description": "Admin access to Horde",
        "displayName": "Admin",
        "isEnabled": true,
        "value": "Admin"
    },
    {
        "allowedMemberTypes": [
        "User",
        "Application"
        ],
        "description": "Full access to Horde storage",
        "displayName": "Storage Full Access",
        "isEnabled": true,
        "value": "Storage.full"
    },
    {
        "allowedMemberTypes": [
        "User",
        "Application"
        ],
        "description": "Full access to Horde cache data",
        "displayName": "Cache Full Access",
        "isEnabled": true,
        "value": "Cache.full"
    }]
    ```

### Client

Create a new Service Principal dedicated for the accessing the cache from Visual Studio.
Under **API Permissions**, add the API permission from the first SP, `Horde.ReadWrite`

### Worker

Create a new Service Principal dedicated for the accessing the cache from Visual Studio.
Under **API Permissions**, add the API permission from the first SP, `Horde.Cache` and `Horde.Storage`

### Add Users or Groups

After configuring the Application Identity, you can control the access to the Unreal Engine DDC. Use the following script to programmatically add additional users. This may also be done using the Azure Portal directly.

General directions can be found [here](/azure/active-directory/manage-apps/grant-admin-consent).

```azurecli-interactive
appRoleId=''
user=''
spObjectId=''
az rest \
    --method post \
    --uri https://graph.microsoft.com/beta/users/$user/appRoleAssignments \
    --body "{\"appRoleId\": \"$appRoleId\", \"principalId\": \"$user\", \"resourceId\": \"$spObjectId\"}" \
    --headers "Content-Type=application/json"
```

## Next steps

* [Create an Unreal DDC managed application in Azure portal](quickstart-portal.md)
