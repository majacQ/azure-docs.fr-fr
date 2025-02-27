---
title: Gérer des espaces de travail dans le portail ou le kit de développement logiciel Python
titleSuffix: Azure Machine Learning
description: Découvrez comment gérer des espaces de travail Azure Machine Learning à l’aide du portail Azure ou du Kit de développement logiciel (SDK) pour Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: sgilley
author: sdgilley
ms.date: 04/22/2021
ms.topic: how-to
ms.custom: fasttrack-edit, FY21Q4-aml-seo-hack, contperf-fy21q4
ms.openlocfilehash: 5d569598c51429cb12027f3955fa9315a05b16bb
ms.sourcegitcommit: 2ed2d9d6227cf5e7ba9ecf52bf518dff63457a59
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2021
ms.locfileid: "132521982"
---
# <a name="manage-azure-machine-learning-workspaces-in-the-portal-or-with-the-python-sdk"></a>Gérer les espaces de travail Azure Machine Learning dans le portail ou avec le SDK Python

Cet article explique comment créer, afficher et supprimer des [**espaces de travail Azure Machine Learning**](concept-workspace.md) pour le service [Azure Machine Learning](overview-what-is-azure-machine-learning.md) à l’aide du portail Azure ou du [Kit de développement logiciel (SDK) pour Python](/python/api/overview/azure/ml/)

À mesure que vos besoins évoluent ou que les exigences en matière d’automation augmentent, vous pouvez également gérer des espaces de travail [à l’aide de l’interface CLI](reference-azure-machine-learning-cli.md) ou [via l’extension VS Code](how-to-setup-vs-code.md).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://azure.microsoft.com/free/) dès aujourd’hui.
* Si vous comptez utiliser le Kit de développement logiciel (SDK) Python, [installez-le](/python/api/overview/azure/ml/install).

## <a name="limitations"></a>Limites

[!INCLUDE [register-namespace](../../includes/machine-learning-register-namespace.md)]

Par défaut, la création d’un espace de travail crée également une instance ACR (Azure Container Registry).  Étant donné qu’ACR ne prend actuellement pas en charge les caractères Unicode dans le nom des groupes de ressources, utilisez un groupe de ressources qui n’en contient pas.

## <a name="create-a-workspace"></a>Créer un espace de travail

# <a name="python"></a>[Python](#tab/python)

* **Spécification par défaut.** Par défaut, les ressources dépendantes et le groupe de ressources sont créés automatiquement. Ce code crée un espace de travail nommé `myworkspace` et un groupe de ressources nommé `myresourcegroup` dans `eastus2`.
    
    ```python
    from azureml.core import Workspace
    
    ws = Workspace.create(name='myworkspace',
                   subscription_id='<azure-subscription-id>',
                   resource_group='myresourcegroup',
                   create_resource_group=True,
                   location='eastus2'
                   )
    ```
    Si vous disposez déjà d’un groupe de ressources Azure que vous souhaitez utiliser comme espace de travail, définissez `create_resource_group` sur False.

* <a name="create-multi-tenant"></a>**Locataires multiples.**  Si vous avez plusieurs comptes, ajoutez l’ID de locataire du service Azure Active Directory que vous souhaitez utiliser.  Recherchez votre ID de locataire dans le [portail Azure](https://portal.azure.com) sous **Azure Active Directory, Identités externes**.

    ```python
    from azureml.core.authentication import InteractiveLoginAuthentication
    from azureml.core import Workspace
    
    interactive_auth = InteractiveLoginAuthentication(tenant_id="my-tenant-id")
    ws = Workspace.create(name='myworkspace',
                subscription_id='<azure-subscription-id>',
                resource_group='myresourcegroup',
                create_resource_group=True,
                location='eastus2',
                auth=interactive_auth
                )
    ```

* **[Cloud souverain](reference-machine-learning-cloud-parity.md)** . Si vous travaillez dans un cloud souverain, vous aurez besoin de code supplémentaire pour vous authentifier auprès d’Azure.

    ```python
    from azureml.core.authentication import InteractiveLoginAuthentication
    from azureml.core import Workspace
    
    interactive_auth = InteractiveLoginAuthentication(cloud="<cloud name>") # for example, cloud="AzureUSGovernment"
    ws = Workspace.create(name='myworkspace',
                subscription_id='<azure-subscription-id>',
                resource_group='myresourcegroup',
                create_resource_group=True,
                location='eastus2',
                auth=interactive_auth
                )
    ```

* **Utiliser des ressources Azure existantes**.  Vous pouvez également créer un espace de travail utilisant des ressources Azure existantes avec le format d’ID de ressource Azure. Recherchez les ID de ressource Azure spécifiques dans le portail Azure ou à l’aide du Kit de développement logiciel (SDK). Cet exemple suppose que le groupe de ressources, le compte de stockage, le coffre de clés, la fonctionnalité Application Insights et le registre de conteneurs existent déjà.

   ```python
   import os
   from azureml.core import Workspace
   from azureml.core.authentication import ServicePrincipalAuthentication

   service_principal_password = os.environ.get("AZUREML_PASSWORD")

   service_principal_auth = ServicePrincipalAuthentication(
      tenant_id="<tenant-id>",
      username="<application-id>",
      password=service_principal_password)

                        auth=service_principal_auth,
                             subscription_id='<azure-subscription-id>',
                             resource_group='myresourcegroup',
                             create_resource_group=False,
                             location='eastus2',
                             friendly_name='My workspace',
                             storage_account='subscriptions/<azure-subscription-id>/resourcegroups/myresourcegroup/providers/microsoft.storage/storageaccounts/mystorageaccount',
                             key_vault='subscriptions/<azure-subscription-id>/resourcegroups/myresourcegroup/providers/microsoft.keyvault/vaults/mykeyvault',
                             app_insights='subscriptions/<azure-subscription-id>/resourcegroups/myresourcegroup/providers/microsoft.insights/components/myappinsights',
                             container_registry='subscriptions/<azure-subscription-id>/resourcegroups/myresourcegroup/providers/microsoft.containerregistry/registries/mycontainerregistry',
                             exist_ok=False)
   ```

Pour plus d’informations, consultez la [Référence du Kit de développement logiciel (SDK) d’espace de travail](/python/api/azureml-core/azureml.core.workspace.workspace).

Si vous rencontrez des problèmes pour accéder à votre abonnement, consultez [Configurer l’authentification pour des ressources et workflows Azure Machine Learning](how-to-setup-authentication.md), ainsi que le bloc-notes [Authentification dans Azure Machine Learning](https://aka.ms/aml-notebook-auth).

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Connectez-vous au [Portail Azure](https://portal.azure.com/) à l’aide des informations d’identification de votre abonnement Azure. 

1. En haut à gauche du portail Azure, sélectionnez **+ Créer une ressource**.

      ![Créer une nouvelle ressource](./media/how-to-manage-workspace/create-workspace.gif)

1. Utilisez la barre de recherche pour rechercher **Machine Learning**.

1. Sélectionnez **Machine Learning**.

1. Dans le volet **Machine Learning**, sélectionnez **Créer** pour commencer.

1. Fournissez les informations suivantes pour configurer votre nouvel espace de travail :

   Champ|Description 
   ---|---
   Nom de l’espace de travail |Entrez un nom unique qui identifie votre espace de travail. Dans cet exemple, nous allons utiliser **docs-ws**. Dans le groupe de ressources, les noms doivent être uniques. Utilisez un nom dont il est facile de se rappeler et que vous pouvez facilement différencier des autres espaces de travail. Le nom de l’espace de travail n’est pas sensible à la casse.
   Abonnement |Sélectionnez l’abonnement Azure que vous souhaitez utiliser.
   Resource group | Utilisez un groupe de ressources existant dans votre abonnement, ou entrez un nom pour créer un groupe de ressources. Un groupe de ressources contient les ressources associées d’une solution Azure. Dans cet exemple, nous allons utiliser **docs-aml**. Pour utiliser un groupe de ressources existant, vous devez disposer du rôle de *contributeur* ou de *propriétaire*.  Pour plus d'informations sur l'accès, consultez [Gérer l'accès à un espace de travail Azure Machine Learning](how-to-assign-roles.md).
   Région | Sélectionnez la région Azure la plus proche de vos utilisateurs et des ressources de données pour créer votre espace de travail.
   | Compte de stockage | Le compte de stockage par défaut de l’espace de travail. Par défaut, un nouveau compte est créé. |
   | Key Vault | Coffre Azure Key Vault utilisé par l’espace de travail. Par défaut, un nouveau coffre est créé. |
   | Application Insights | L’instance Application Insights de l’espace de travail. Par défaut, une nouvelle instance est créée. |
   | Container Registry | L’instance Azure Container Registry de l’espace de travail. Par défaut, _aucune_ nouvelle instance n’est initialement créée pour l’espace de travail. Au lieu de cela, l’instance est créée quand vous en avez besoin lors de la création d’une image Docker pendant la formation ou le déploiement. |

   :::image type="content" source="media/how-to-manage-workspace/create-workspace-form.png" alt-text="Configurer votre espace de travail.":::

1. Lorsque vous avez terminé de configurer l’espace de travail, sélectionnez **Vérifier + créer**. Si vous le souhaitez, utilisez les sections [Mise en réseau](#networking) et [Avancés](#advanced) pour configurer d’autres paramètres pour l’espace de travail.

1. Passez en revue les paramètres et apportez toute modification ou correction supplémentaire. Lorsque vous êtes satisfait des paramètres, sélectionnez **Créer**.

   > [!Warning] 
   > La création de votre espace de travail dans le cloud peut prendre plusieurs minutes.

   Une fois le processus terminé, un message indiquant la réussite du déploiement s’affiche. 
 
 1. Pour afficher le nouvel espace de travail, sélectionnez **Accéder à la ressource**.
 
---



### <a name="networking"></a>Mise en réseau  

> [!IMPORTANT]  
> Pour plus d’informations sur l’utilisation d’un point de terminaison privé et d’un réseau virtuel avec votre espace de travail, consultez [Isolement réseau et confidentialité](how-to-network-security-overview.md).


# <a name="python"></a>[Python](#tab/python)

Fournie par le SDK Python Azure Machine Learning, la classe [PrivateEndpointConfig](/python/api/azureml-core/azureml.core.privateendpointconfig) peut être utilisée avec [Workspace.create()](/python/api/azureml-core/azureml.core.workspace.workspace#create-name--auth-none--subscription-id-none--resource-group-none--location-none--create-resource-group-true--sku--basic---tags-none--friendly-name-none--storage-account-none--key-vault-none--app-insights-none--container-registry-none--adb-workspace-none--cmk-keyvault-none--resource-cmk-uri-none--hbi-workspace-false--default-cpu-compute-target-none--default-gpu-compute-target-none--private-endpoint-config-none--private-endpoint-auto-approval-true--exist-ok-false--show-output-true-) pour créer un espace de travail avec un point de terminaison privé. Cette classe nécessite un réseau virtuel existant.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. La configuration réseau par défaut consiste à utiliser un __point de terminaison public__, accessible sur l’Internet public. Pour limiter l’accès de votre espace de travail à un réseau virtuel Azure que vous avez créé, vous pouvez à la place sélectionner __Point de terminaison privé__ comme __Méthode de connectivité__, puis utiliser __+ Ajouter__ pour configurer le point de terminaison. 

   :::image type="content" source="media/how-to-manage-workspace/select-private-endpoint.png" alt-text="Sélection du point de terminaison privé":::  

1. Dans le formulaire __Créer un point de terminaison privé__, définissez l’emplacement, le nom et le réseau virtuel à utiliser. Si vous souhaitez utiliser le point de terminaison avec une zone DNS privée, sélectionnez __Intégrer à une zone DNS privée__, puis choisissez la zone à l’aide du champ __Zone DNS privée__. Cliquez sur __OK__ pour créer le point de terminaison.   

   :::image type="content" source="media/how-to-manage-workspace/create-private-endpoint.png" alt-text="Création d’un point de terminaison privé":::   

1. Lorsque vous avez terminé la configuration de la mise en réseau, vous pouvez sélectionner __Vérifier + créer__ ou passer à la configuration facultative __Avancé__.

---

### <a name="vulnerability-scanning"></a>Analyse des vulnérabilités

Microsoft Defender pour cloud fournit une gestion unifiée de la sécurité et une protection avancée contre les menaces dans les charges de travail cloud hybrides. Vous devez autoriser Microsoft Defender for Cloud à analyser vos ressources et à suivre ses recommandations. Pour plus d’informations, consultez [Analyse d’images Azure Container Registry par Defender for Cloud](../security-center/defender-for-container-registries-introduction.md) et [Intégration d’Azure Kubernetes Service à Defender for Cloud](../security-center/defender-for-kubernetes-introduction.md).

### <a name="advanced"></a>Avancé

Par défaut, les métadonnées de l’espace de travail sont stockées dans une instance d’Azure Cosmos DB gérée par Microsoft. Les données sont chiffrées avec des clés managées par Microsoft.

Pour limiter les données que Microsoft collecte sur votre espace de travail, sélectionnez l’espace de travail __HBI (High Business Impact)__ dans le portail, ou définissez `hbi_workspace=true ` dans Python. Pour plus d’informations sur ce paramètre, consultez [Chiffrement au repos](concept-data-encryption.md#encryption-at-rest).

> [!IMPORTANT]  
> Sélectionner High Business Impact ne peut être effectué que lors de la création d’un espace de travail. Vous ne pouvez pas modifier ce paramètre une fois l’espace de travail créé.   

#### <a name="use-your-own-key"></a>Utiliser votre propre clé

Vous pouvez fournir votre propre clé pour le chiffrement des données. Cela crée l’instance d’Azure Cosmos DB qui stocke les métadonnées dans votre abonnement Azure.

[!INCLUDE [machine-learning-customer-managed-keys.md](../../includes/machine-learning-customer-managed-keys.md)]

Pour fournir votre propre clé, procédez comme suit :

> [!IMPORTANT]  
> Avant de commencer, vous devez effectuer les actions suivantes :   
>
> 1. Autorisez l’__application Azure Machine Learning__ (dans la gestion des identités et des accès) avec des autorisations de contributeur pour votre abonnement.  
> 1. Suivez les étapes décrites dans [Configurer les clés gérées par le client](../cosmos-db/how-to-setup-cmk.md) pour :
>     * Inscrire le fournisseur Azure Cosmos DB
>     * Créer et configurer un coffre Azure Key Vault
>     * Générer une clé
>   
>     Vous n’avez pas besoin de créer manuellement l’instance d’Azure Cosmos DB, l’une sera créée pour vous lors de la création de l’espace de travail. Cette instance de Azure Cosmos DB sera créée dans un groupe de ressources distinct à l’aide d’un nom basé sur ce modèle : `<your-workspace-resource-name>_<GUID>`.   
>   
> Vous ne pouvez pas modifier ce paramètre une fois l’espace de travail créé. Si vous supprimez l’Azure Cosmos DB utilisé par votre espace de travail, vous devez également supprimer l’espace de travail qui l’utilise.

# <a name="python"></a>[Python](#tab/python)

Utilisez les valeurs `cmk_keyvault` et `resource_cmk_uri` pour spécifier la clé gérée par le client.

```python
from azureml.core import Workspace
   ws = Workspace.create(name='myworkspace',
               subscription_id='<azure-subscription-id>',
               resource_group='myresourcegroup',
               create_resource_group=True,
               location='eastus2'
               cmk_keyvault='subscriptions/<azure-subscription-id>/resourcegroups/myresourcegroup/providers/microsoft.keyvault/vaults/<keyvault-name>', 
               resource_cmk_uri='<key-identifier>'
               )

```

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Sélectionnez __Clés gérées par le client__, puis __Cliquer pour sélectionner la clé__.

    :::image type="content" source="media/how-to-manage-workspace/advanced-workspace.png" alt-text="Clés gérées par le client":::

1. Dans le formulaire __Sélectionner une clé dans Azure Key Vault__, sélectionnez un coffre de clés Azure Key Vault existant, une clé dans ce coffre, et la version de la clé. Cette clé est utilisée pour chiffrer toutes les données stockées dans Azure Cosmos DB. Enfin, cliquez sur le bouton __Sélectionner__ pour utiliser cette clé.

   :::image type="content" source="media/how-to-manage-workspace/select-key-vault.png" alt-text="Sélectionner la clé":::

---

### <a name="download-a-configuration-file"></a>Télécharger un fichier de configuration

Si vous comptez créer une [instance de calcul](quickstart-create-resources.md), ignorez cette étape.  L’instance de calcul a déjà créé une copie de ce fichier pour vous.

# <a name="python"></a>[Python](#tab/python)

Si vous envisagez d’utiliser sur votre environnement local du code faisant référence à cet espace de travail (`ws`), écrivez le fichier de configuration :

```python
ws.write_config()
```

# <a name="portal"></a>[Portail](#tab/azure-portal)

Si vous prévoyez d’utiliser du code sur votre environnement local qui référence cet espace de travail, sélectionnez **Télécharger config.json** dans la section **Vue d’ensemble** de l’espace de travail.  

   ![Télécharger config.json](./media/how-to-manage-workspace/configure.png)

---

Placez le fichier dans la structure de répertoires avec vos scripts Python ou vos notebooks Jupyter. Il peut se trouver dans le même répertoire, dans un sous-répertoire nommé *.azureml* ou dans un répertoire parent. Quand vous créez une instance de calcul, ce fichier est automatiquement ajouté dans le répertoire approprié sur la machine virtuelle.

## <a name="connect-to-a-workspace"></a>Se connecter à un espace de travail

Dans votre code Python, vous créez un objet espace de travail à connecter à votre espace de travail.  Ce code lira le contenu du fichier de configuration pour trouver votre espace de travail.  Si vous n’êtes pas encore authentifié, vous recevez une invite pour vous connecter.

```python
from azureml.core import Workspace

ws = Workspace.from_config()
```

* <a name="connect-multi-tenant"></a>**Locataires multiples.**  Si vous avez plusieurs comptes, ajoutez l’ID de locataire du service Azure Active Directory que vous souhaitez utiliser.  Recherchez votre ID de locataire dans le [portail Azure](https://portal.azure.com) sous **Azure Active Directory, Identités externes**.

    ```python
    from azureml.core.authentication import InteractiveLoginAuthentication
    from azureml.core import Workspace
    
    interactive_auth = InteractiveLoginAuthentication(tenant_id="my-tenant-id")
    ws = Workspace.from_config(auth=interactive_auth)
    ```

* **[Cloud souverain](reference-machine-learning-cloud-parity.md)** . Si vous travaillez dans un cloud souverain, vous aurez besoin de code supplémentaire pour vous authentifier auprès d’Azure.

    ```python
    from azureml.core.authentication import InteractiveLoginAuthentication
    from azureml.core import Workspace
    
    interactive_auth = InteractiveLoginAuthentication(cloud="<cloud name>") # for example, cloud="AzureUSGovernment"
    ws = Workspace.from_config(auth=interactive_auth)
    ```
    
Si vous rencontrez des problèmes pour accéder à votre abonnement, consultez [Configurer l’authentification pour des ressources et workflows Azure Machine Learning](how-to-setup-authentication.md), ainsi que le bloc-notes [Authentification dans Azure Machine Learning](https://aka.ms/aml-notebook-auth).

## <a name="find-a-workspace"></a><a name="view"></a>Trouver un espace de travail

Consultez la liste de tous les espaces de travail que vous pouvez utiliser.

# <a name="python"></a>[Python](#tab/python)

Recherchez vos abonnements dans la [page Abonnements du portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade). Copiez l’ID et utilisez-le dans le code ci-dessous pour afficher tous les espaces de travail disponibles pour cet abonnement.

```python
from azureml.core import Workspace

Workspace.list('<subscription-id>')
```

La méthode Workspace.list (..) ne retourne pas l’objet espace de travail complet. Il comprend uniquement des informations de base sur les espaces de travail existants dans l’abonnement. Pour obtenir un objet complet pour un espace de travail spécifique, utilisez Workspace.get(..).

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Dans le champ de recherche du haut, tapez **Machine Learning**.  

1. Sélectionnez **Machine Learning**.

   ![Rechercher l’espace de travail Azure Machine Learning](./media/how-to-manage-workspace/find-workspaces.png)

1. Examinez la liste des espaces de travail trouvés. Vous pouvez filtrer en fonction d’abonnements, de groupes de ressources et de localisations.  

1. Sélectionnez un espace de travail pour afficher ses propriétés.

---


## <a name="delete-a-workspace"></a>Supprimer un espace de travail

Lorsque vous n’avez plus besoin d’un espace de travail, supprimez-le.  

[!INCLUDE [machine-learning-delete-workspace](../../includes/machine-learning-delete-workspace.md)]

Si vous avez supprimé accidentellement votre espace de travail, vous pouvez toujours récupérer vos blocs-notes. Pour plus de détails, voir [Basculement de la continuité d’activité et reprise d’activité](/azure/machine-learning/how-to-high-availability-machine-learning#workspace-deletion).

# <a name="python"></a>[Python](#tab/python)

Supprimer l’espace de travail `ws` :

```python
ws.delete(delete_dependent_resources=False, no_wait=False)
```

L’action par défaut ne consiste pas à supprimer les ressources associées à l’espace de travail, à savoir le registre de conteneurs, le compte de stockage, le coffre de clés et la fonctionnalité Application Insights.  Définissez `delete_dependent_resources` sur True pour supprimer également ces ressources.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Dans le [portail Azure](https://portal.azure.com/), sélectionnez **Supprimer** en haut de l’espace de travail que vous souhaitez supprimer.

:::image type="content" source="./media/how-to-manage-workspace/delete-workspace.png" alt-text="Supprimer un espace de travail":::

---

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

## <a name="troubleshooting"></a>Dépannage

* **Navigateurs pris en charge dans Azure Machine Learning studio** : Nous vous recommandons d’utiliser le navigateur le plus récent compatible avec votre système d’exploitation. Les opérateurs suivants sont pris en charge :
  * Microsoft Edge (le nouveau Microsoft Edge, dernière version. Pas Microsoft Edge hérité)
  * Safari (dernière version, Mac uniquement)
  * Chrome (version la plus récente)
  * Firefox (version la plus récente)

* **Portail Azure**: 
  * Si vous accédez directement à votre espace de travail à partir d’un lien de partage provenant du kit SDK ou du Portail Azure, vous ne pourrez pas afficher la page **Vue d’ensemble** standard comportant des informations sur l’abonnement dans l’extension. Dans ce scénario, vous ne pouvez pas non plus basculer vers un autre espace de travail. Pour afficher un autre espace de travail, accédez directement à [Azure Machine Learning Studio](https://ml.azure.com), puis recherchez le nom de l’espace de travail.
  * Toutes les ressources (jeux de données, expériences, calculs, etc.) sont uniquement disponibles dans [Azure Machine Learning Studio](https://ml.azure.com). Ils ne sont *pas* disponibles dans le portail Azure.

### <a name="workspace-diagnostics"></a>Diagnostics de l’espace de travail

[!INCLUDE [machine-learning-workspace-diagnostics](../../includes/machine-learning-workspace-diagnostics.md)]

### <a name="resource-provider-errors"></a>Erreurs du fournisseur de ressources

[!INCLUDE [machine-learning-resource-provider](../../includes/machine-learning-resource-provider.md)]

### <a name="moving-the-workspace"></a>Déplacement de l’espace de travail

> [!WARNING]
> Le déplacement de votre espace de travail Azure Machine Learning vers un autre abonnement, ou le déplacement de l’abonnement propriétaire vers un nouveau locataire, n’est pas pris en charge. En effet, cela peut provoquer des erreurs.

### <a name="deleting-the-azure-container-registry"></a>Suppression d'Azure Container Registry

L'espace de travail Azure Machine Learning utilise Azure Container Registry (ACR) pour certaines opérations. Il crée automatiquement une instance ACR dès qu'il en a besoin.

[!INCLUDE [machine-learning-delete-acr](../../includes/machine-learning-delete-acr.md)]

## <a name="examples"></a>Exemples

Exemples de création d’espace de travail :
* utiliser le portail Azure pour [créer un espace de travail et une instance de calcul](quickstart-create-resources.md) ;

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous disposez d’un espace de travail, découvrez comment [effectuer l’apprentissage d’un modèle, puis déployer celui-ci](tutorial-train-models-with-aml.md).

Pour en savoir plus sur la planification d’un espace de travail pour les besoins de votre organisation, consultez [Organiser et configurer Azure Machine Learning](/azure/cloud-adoption-framework/ready/azure-best-practices/ai-machine-learning-resource-organization).
