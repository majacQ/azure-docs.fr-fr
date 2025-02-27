---
title: Créer des rôles personnalisés dans le contrôle d’accès en fonction du rôle Azure AD | Microsoft Docs
description: Créez et attribuez des rôles Azure AD personnalisés avec l’étendue des ressources sur les ressources Azure Active Directory.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: how-to
ms.date: 10/06/2021
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8425c85f74e5540094be4dab12eb3e3e7c931d1b
ms.sourcegitcommit: 106f5c9fa5c6d3498dd1cfe63181a7ed4125ae6d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/02/2021
ms.locfileid: "131057321"
---
# <a name="create-and-assign-a-custom-role-in-azure-active-directory"></a>Créer et attribuer un rôle personnalisé dans Azure Active Directory

Cet article explique comment créer des rôles personnalisés dans Azure Active Directory (Azure AD). Pour connaître les principes de base des rôles personnalisés, consultez la [vue d’ensemble des rôles personnalisés](custom-overview.md). Le rôle peut être affecté soit au niveau de l’étendue au niveau du répertoire, soit à une étendue de ressource d’inscription d’application uniquement.

Vous pouvez créer des rôles personnalisés sous l’onglet [Rôles et administrateurs](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RolesAndAdministrators) de la page vue d’ensemble de Azure AD.

## <a name="prerequisites"></a>Prérequis

- Licence Azure AD Premium P1 ou P2
- Administrateur de rôle privilégié ou Administrateur général
- Module AzureADPreview (avec PowerShell)
- Consentement administrateur (avec l’Afficheur Graph pour l’API Microsoft Graph)

Pour plus d’informations, consultez [Prérequis pour utiliser PowerShell ou de l’Afficheur Graph](prerequisites.md).

## <a name="create-a-role-in-the-azure-portal"></a>Créer un rôle dans le portail Azure

### <a name="create-a-new-custom-role-to-grant-access-to-manage-app-registrations"></a>Créer un rôle personnalisé pour accorder l’accès pour gérer les inscriptions des applications

1. Connectez-vous au [portail Azure](https://portal.azure.com) ou au [Centre d’administration Azure AD](https://aad.portal.azure.com).

1. Sélectionnez **Azure Active Directory** > **Rôles et administrateurs** > **Nouveau rôle personnalisé**.

   ![Créer ou modifier des rôles à partir de la page Rôles et administrateurs](./media/custom-create/new-custom-role.png)

1. Dans l'onglet **Notions de base**, indiquez un nom et une description pour le rôle, puis cliquez sur **Suivant**.

   ![fournir un nom et une description pour un rôle personnalisé sous l’onglet Notions de base](./media/custom-create/basics-tab.png)

1. Sous l'onglet **Permissions**, sélectionnez les permissions nécessaires pour gérer les propriétés de base et les propriétés des informations d’identification des inscriptions d’applications. Pour obtenir une description détaillée de chaque permission, consultez [Sous-types et permissions d'inscription de l’application dans Azure Active Directory](custom-available-permissions.md).
   1. Tout d’abord, entrez « informations d’identification » dans la barre de recherche et sélectionnez la permission `microsoft.directory/applications/credentials/update`.

      ![Sélectionnez les permissions pour un rôle personnalisé sous l’onglet Permissions.](./media/custom-create/permissions-tab.png)

   1. Ensuite, entrez « notions de base » dans la barre de recherche, sélectionnez la permission `microsoft.directory/applications/basic/update`, puis cliquez sur **Suivant**.
1. Sur l’onglet **Vérifier + Créer**, vérifiez les permissions, puis sélectionnez **Créer**.

Votre rôle personnalisé s’affiche dans la liste des rôles disponibles à affecter.

## <a name="create-a-role-using-powershell"></a>Créer un rôle avec PowerShell

### <a name="connect-to-azure"></a>Connexion à Azure

Pour vous connecter à Azure Active Directory, utilisez la commande suivante :

``` PowerShell
Connect-AzureAD
```

### <a name="create-the-custom-role"></a>Créer le rôle personnalisé

Créez un nouveau rôle à l’aide du script PowerShell suivant :

``` PowerShell
# Basic role information
$displayName = "Application Support Administrator"
$description = "Can manage basic aspects of application registrations."
$templateId = (New-Guid).Guid
 
# Set of permissions to grant
$allowedResourceAction =
@(
    "microsoft.directory/applications/basic/update",
    "microsoft.directory/applications/credentials/update"
)
$rolePermissions = @{'allowedResourceActions'= $allowedResourceAction}
 
# Create new custom admin role
$customAdmin = New-AzureADMSRoleDefinition -RolePermissions $rolePermissions -DisplayName $displayName -Description $description -TemplateId $templateId -IsEnabled $true
```

### <a name="assign-the-custom-role-using-powershell"></a>Attribuer un rôle personnalisé avec PowerShell

Attribuez le rôle à l’aide du script PowerShell ci-dessous :

``` PowerShell
# Get the user and role definition you want to link
$user = Get-AzureADUser -Filter "userPrincipalName eq 'cburl@f128.info'"
$roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Application Support Administrator'"

# Get app registration and construct resource scope for assignment.
$appRegistration = Get-AzureADApplication -Filter "displayName eq 'f/128 Filter Photos'"
$resourceScope = '/' + $appRegistration.objectId

# Create a scoped role assignment
$roleAssignment = New-AzureADMSRoleAssignment -DirectoryScopeId $resourceScope -RoleDefinitionId $roleDefinition.Id -PrincipalId $user.objectId
```

## <a name="create-a-role-with-the-microsoft-graph-api"></a>Créer un rôle avec l’API Microsoft Graph

1. Créez la définition de rôle.

    Requête HTTP pour créer une définition de rôle personnalisée.

    POST

    ``` HTTP
    https://graph.microsoft.com/beta/roleManagement/directory/roleDefinitions
    ```

    body

    ``` HTTP
    {
       "description": "Can manage basic aspects of application registrations.",
       "displayName": "Application Support Administrator",
       "isEnabled": true,
       "templateId": "<GUID>",
       "rolePermissions": [
           {
               "allowedResourceActions": [
                   "microsoft.directory/applications/basic/update",
                   "microsoft.directory/applications/credentials/update"
               ]
           }
       ]
    }
    ```

    > [!Note]
    > `"templateId": "GUID"` est un paramètre facultatif qui est envoyé dans le corps en fonction des besoins. Si vous avez besoin de créer plusieurs rôles personnalisés différents avec des paramètres communs, il est préférable de créer un modèle et de définir une valeur de `templateId`. Vous pouvez générer au préalable une valeur de `templateId` à l’aide de l’applet de commande PowerShell `(New-Guid).Guid`. 

1. Créez l’attribution de rôle.

    Requête HTTP pour créer une définition de rôle personnalisée.

    POST

    ```http
    https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments
    ```

    body

    ```http
    {
        "principalId":"<GUID OF USER>",
        "roleDefinitionId":"<GUID OF ROLE DEFINITION>",
        "resourceScope":"/<GUID OF APPLICATION REGISTRATION>"
    }
    ```

## <a name="assign-a-custom-role-scoped-to-a-resource"></a>Affecter un rôle personnalisé étendu à une ressource

Tout comme les rôles intégrés, les rôles personnalisés sont attribués par défaut à l'échelle de l'entreprise pour accorder des permissions d'accès à toutes les inscriptions d’applications de votre organisation. En outre, des rôles personnalisés et certains rôles intégrés pertinents (selon le type de ressource Azure AD) peuvent également être attribués à l’étendue d’une seule ressource Azure AD. Cela vous permet de donner à l’utilisateur l’autorisation de mettre à jour les informations d’identification et les propriétés de base d’une application unique sans avoir à créer un deuxième rôle personnalisé.

1. Connectez-vous au [portail Azure](https://portal.azure.com) ou au[centre d’administration Azure AD](https://aad.portal.azure.com) avec les autorisations du Développeur d’applications.

1. Sélectionnez **Azure Active Directory** > **Inscriptions des applications**.

1. Sélectionnez l’inscription d’application à laquelle vous accordez l’accès à gérer. Vous devrez peut-être sélectionner **Toutes les applications** pour afficher la liste complète des inscriptions d’applications dans votre organisation Azure AD.

    ![Sélectionner l’inscription d’application en tant qu’étendue de ressource pour une attribution de rôle](./media/custom-create/appreg-all-apps.png)

1. Dans inscription d’application, sélectionnez **Rôles et administrateurs**. Si vous n’en avez encore jamais créé, vous trouverez des instructions dans la [procédure précédente](#create-a-new-custom-role-to-grant-access-to-manage-app-registrations).

1. Sélectionnez le rôle pour ouvrir la page **Affectations**.

1. Sélectionnez **Ajouter une affectation** pour ajouter un utilisateur. L'utilisateur se verra accorder toutes les autorisations pour l’inscription d’application sélectionnée seulement.

## <a name="next-steps"></a>Étapes suivantes

- N’hésitez pas à nous donner votre avis sur le [forum des rôles d’administrateur Azure AD](https://feedback.azure.com/d365community/forum/22920db1-ad25-ec11-b6e6-000d3a4f0789).
- Pour plus d’informations sur les autorisations associées aux rôles, consultez [Rôles intégrés Azure AD](permissions-reference.md).
- Pour les autorisations d’utilisateur par défaut, consultez une [comparaison des autorisations par défaut d’un utilisateur invité et d’un membre](../fundamentals/users-default-permissions.md?context=azure%2factive-directory%2froles%2fcontext%2fugr-context).
