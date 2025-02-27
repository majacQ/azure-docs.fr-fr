---
title: 'Tutoriel : Déployer une application Dapr sur Azure Container Apps à l’aide d’un modèle ARM'
description: Déployer une application Dapr sur Azure Container Apps à l’aide d’un modèle ARM
services: container-apps
author: asw101
ms.service: container-apps
ms.topic: conceptual
ms.date: 11/02/2021
ms.author: aawislan
ms.custom: ignite-fall-2021
ms.openlocfilehash: d38af387910551903d2781863deb3de3fcd36160
ms.sourcegitcommit: 4cd97e7c960f34cb3f248a0f384956174cdaf19f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2021
ms.locfileid: "132026916"
---
# <a name="tutorial-deploy-a-dapr-application-to-azure-container-apps-using-an-arm-template"></a>Tutoriel : Déployer une application Dapr sur Azure Container Apps à l’aide d’un modèle ARM

[Dapr](https://dapr.io/) (Distributed Application Runtime) est un runtime qui permet de générer des microservices résilients, sans état et avec état. Dans ce tutoriel, un exemple d’application Dapr est déployé sur Azure Container Apps.

Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer un environnement Container Apps pour vos applications de conteneur
> * Créer un magasin d’état Stockage Blob Azure pour l’application de conteneur
> * Déployer deux applications qui génèrent et consomment des messages et les conservent à l’aide du magasin d’état
> * Vérifier l’interaction entre les deux microservices

Azure Container Apps offre une version complètement managée des API Dapr lors de la création de microservices. Quand vous utilisez Dapr dans Azure Container Apps, vous pouvez autoriser les side-car à s’exécuter à côté de vos microservices qui fournissent un ensemble complet de fonctionnalités. Les API Dapr disponibles incluent [Appels de service à service](https://docs.dapr.io/developing-applications/building-blocks/service-invocation/), [Pub/Sub](https://docs.dapr.io/developing-applications/building-blocks/pubsub/), [Liaisons d’événements](https://docs.dapr.io/developing-applications/building-blocks/bindings/), [Magasins d’état](https://docs.dapr.io/developing-applications/building-blocks/state-management/) et [Acteurs](https://docs.dapr.io/developing-applications/building-blocks/actors/).

Dans ce tutoriel, vous déployez les mêmes applications à partir du guide de démarrage rapide [Hello World](https://github.com/dapr/quickstarts/tree/master/hello-kubernetes) Dapr, qui se compose d’une application cliente (Python) qui génère des messages, et d’une application de service (nœud) qui consomme et conserve ces messages dans un magasin d’état configuré. Le diagramme d’architecture suivant illustre les composants qui composent ce tutoriel :

:::image type="content" source="media/microservices-dapr/azure-container-apps-microservices-dapr.png" alt-text="Diagramme d’architecture pour microservices Hello World Dapr sur Azure Container Apps":::

## <a name="prerequisites"></a>Prérequis

* [Azure CLI](/cli/azure/install-azure-cli)

## <a name="before-you-begin"></a>Avant de commencer

Ce guide utilise les variables d’environnement suivantes :

# <a name="bash"></a>[Bash](#tab/bash)

```bash
RESOURCE_GROUP="my-containerapps"
LOCATION="canadacentral"
CONTAINERAPPS_ENVIRONMENT="containerapps-env"
LOG_ANALYTICS_WORKSPACE="containerapps-logs"
STORAGE_ACCOUNT_CONTAINER="mycontainer"
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
$RESOURCE_GROUP="my-containerapps"
$LOCATION="canadacentral"
$CONTAINERAPPS_ENVIRONMENT="containerapps-env"
$LOG_ANALYTICS_WORKSPACE="containerapps-logs"
$STORAGE_ACCOUNT_CONTAINER="mycontainer"
```

---

L’extrait de code ci-dessus peut être utilisé pour définir les variables d’environnement à l’aide de bash, zsh ou PowerShell.

# <a name="bash"></a>[Bash](#tab/bash)

```bash
STORAGE_ACCOUNT="<storage account name>"
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
$STORAGE_ACCOUNT="<storage account name>"
```

---

Choisissez un nom pour `STORAGE_ACCOUNT`. Il sera créé à l’étape suivante. Les noms des comptes de stockage doivent être *uniques dans Azure* et comporter entre 3 et 24 caractères, uniquement des lettres minuscules et des chiffres.

## <a name="setup"></a>Programme d’installation

Commencez par vous connecter à Azure à partir de l’interface CLI.

Exécutez la commande suivante et suivez les invites pour terminer le processus d’authentification.

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az login
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurecli
az login
```

---

Vérifiez que vous exécutez la dernière version de l’interface CLI à l’aide de la commande de mise à niveau.

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az upgrade
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurecli
az upgrade
```

---

Ensuite, installez l’extension Azure Container Apps dans l’interface CLI.

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az extension add \
  --source https://workerappscliextension.blob.core.windows.net/azure-cli-extension/containerapp-0.2.0-py2.py3-none-any.whl 
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurecli
az extension add `
  --source https://workerappscliextension.blob.core.windows.net/azure-cli-extension/containerapp-0.2.0-py2.py3-none-any.whl 
```

---

Maintenant que l’extension est installée, inscrivez l’espace de noms `Microsoft.Web`.

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az provider register --namespace Microsoft.Web
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurecli
az provider register --namespace Microsoft.Web
```

---

Créez un groupe de ressources pour organiser les services liés à votre nouvelle application de conteneur.

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az group create \
  --name $RESOURCE_GROUP \
  --location "$LOCATION"
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurecli
az group create `
  --name $RESOURCE_GROUP `
  --location "$LOCATION"
```

---

Une fois l’interface CLI mise à niveau et un nouveau groupe de ressources disponible, vous pouvez créer un environnement Container Apps et déployer votre application de conteneur.

## <a name="create-an-environment"></a>Créer un environnement

Les environnements Azure Container Apps jouent le rôle de limites d’isolement entre un groupe d’applications de conteneur. Les applications de conteneur déployées dans le même environnement sont déployées dans le même réseau virtuel et écrivent les journaux dans le même espace de travail Log Analytics.

Azure Log Analytics permet de superviser votre application de conteneur et est nécessaire lors de la création d’un environnement Container Apps.

Créez un espace de travail Log Analytics à l’aide de la commande suivante :

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az monitor log-analytics workspace create \
  --resource-group $RESOURCE_GROUP \
  --workspace-name $LOG_ANALYTICS_WORKSPACE
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurecli
az monitor log-analytics workspace create `
  --resource-group $RESOURCE_GROUP `
  --workspace-name $LOG_ANALYTICS_WORKSPACE
```

---

Ensuite, récupérez l’ID client et la clé secrète client Log Analytics.

# <a name="bash"></a>[Bash](#tab/bash)

Veillez à exécuter chaque requête séparément afin de disposer de suffisamment de temps pour que la requête se termine.

```bash
LOG_ANALYTICS_WORKSPACE_CLIENT_ID=`az monitor log-analytics workspace show --query customerId -g $RESOURCE_GROUP -n $LOG_ANALYTICS_WORKSPACE --out tsv`
```

```bash
LOG_ANALYTICS_WORKSPACE_CLIENT_SECRET=`az monitor log-analytics workspace get-shared-keys --query primarySharedKey -g $RESOURCE_GROUP -n $LOG_ANALYTICS_WORKSPACE --out tsv`
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Veillez à exécuter chaque requête séparément afin de disposer de suffisamment de temps pour que la requête se termine.

```powershell
$LOG_ANALYTICS_WORKSPACE_CLIENT_ID=(az monitor log-analytics workspace show --query customerId -g $RESOURCE_GROUP -n $LOG_ANALYTICS_WORKSPACE --out tsv)
```

```powershell
$LOG_ANALYTICS_WORKSPACE_CLIENT_SECRET=(az monitor log-analytics workspace get-shared-keys --query primarySharedKey -g $RESOURCE_GROUP -n $LOG_ANALYTICS_WORKSPACE --out tsv)
```

---

Les applications de conteneur sont déployées dans un environnement Azure Container Apps. Pour créer l’environnement, exécutez la commande suivante :

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az containerapp env create \
  --name $CONTAINERAPPS_ENVIRONMENT \
  --resource-group $RESOURCE_GROUP \
  --logs-workspace-id $LOG_ANALYTICS_WORKSPACE_CLIENT_ID \
  --logs-workspace-key $LOG_ANALYTICS_WORKSPACE_CLIENT_SECRET \
  --location "$LOCATION"
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurecli
az containerapp env create `
  --name $CONTAINERAPPS_ENVIRONMENT `
  --resource-group $RESOURCE_GROUP `
  --logs-workspace-id $LOG_ANALYTICS_WORKSPACE_CLIENT_ID `
  --logs-workspace-key $LOG_ANALYTICS_WORKSPACE_CLIENT_SECRET `
  --location "$LOCATION"
```

---

## <a name="set-up-a-state-store"></a>Configurer un magasin d’état

### <a name="create-an-azure-blob-storage-account"></a>Créer un compte de stockage Blob Azure

Utilisez la commande suivante pour créer un compte Stockage Azure.

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location "$LOCATION" \
  --sku Standard_RAGRS \
  --kind StorageV2
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurecli
az storage account create `
  --name $STORAGE_ACCOUNT `
  --resource-group $RESOURCE_GROUP `
  --location "$LOCATION" `
  --sku Standard_RAGRS `
  --kind StorageV2
```

---

Une fois votre compte Stockage Blob Azure créé, les valeurs suivantes sont nécessaires pour les étapes suivantes de ce tutoriel.

* `storage_account_name` est la valeur de la variable `STORAGE_ACCOUNT` que vous avez choisie ci-dessus.

* `storage_container_name` est la valeur de `STORAGE_ACCOUNT_CONTAINER` définie ci-dessus (par exemple, `mycontainer`). Dapr crée un conteneur portant ce nom s’il n’existe pas déjà dans votre compte Stockage Azure.

Récupérez la clé du compte de stockage à l’aide de la commande suivante.

# <a name="bash"></a>[Bash](#tab/bash)

```bash
STORAGE_ACCOUNT_KEY=`az storage account keys list --resource-group $RESOURCE_GROUP --account-name $STORAGE_ACCOUNT --query '[0].value' --out tsv`
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
$STORAGE_ACCOUNT_KEY=(az storage account keys list --resource-group $RESOURCE_GROUP --account-name $STORAGE_ACCOUNT --query '[0].value' --out tsv)
```

---

### <a name="create-azure-resource-manager-arm-templates"></a>Créer des modèles ARM (Azure Resource Manager)

Créez deux modèles ARM (Azure Resource Manager).

Le modèle ARM a la définition Container Apps et une définition de composant Dapr.

L’exemple suivant montre a quoi votre modèle ARM doit ressembler lorsqu’il est configuré pour votre compte Stockage Blob Azure.

Enregistrez le fichier suivant en tant que *serviceapp.json* :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "defaultValue": "canadacentral",
            "type": "String"
        },
        "environment_name": {
            "type": "String"
        },
        "storage_account_name": {
            "type": "String"
        },
        "storage_account_key": {
            "type": "String"
        },
        "storage_container_name": {
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "name": "nodeapp",
            "type": "Microsoft.Web/containerApps",
            "apiVersion": "2021-03-01",
            "kind": "containerapp",
            "location": "[parameters('location')]",
            "properties": {
                "kubeEnvironmentId": "[resourceId('Microsoft.Web/kubeEnvironments', parameters('environment_name'))]",
                "configuration": {
                    "ingress": {
                        "external": true,
                        "targetPort": 3000
                    },
                    "secrets": [
                        {
                            "name": "storage-key",
                            "value": "[parameters('storage_account_key')]"
                        }
                    ]
                },
                "template": {
                    "containers": [
                        {
                            "image": "dapriosamples/hello-k8s-node:latest",
                            "name": "hello-k8s-node",
                            "resources": {
                                "cpu": 0.5,
                                "memory": "1Gi"
                            }
                        }
                    ],
                    "scale": {
                        "minReplicas": 1,
                        "maxReplicas": 1
                    },
                    "dapr": {
                        "enabled": true,
                        "appPort": 3000,
                        "appId": "nodeapp",
                        "components": [
                            {
                                "name": "statestore",
                                "type": "state.azure.blobstorage",
                                "version": "v1",
                                "metadata": [
                                    {
                                        "name": "accountName",
                                        "value": "[parameters('storage_account_name')]"
                                    },
                                    {
                                        "name": "accountKey",
                                        "secretRef": "storage-key"
                                    },
                                    {
                                        "name": "containerName",
                                        "value": "[parameters('storage_container_name')]"
                                    }
                                ]
                            }
                        ]
                    }
                }
            }
        }
    ]
}
```

> [!NOTE]
> Actuellement, Container Apps ne prend pas en charge le [schéma de composants Dapr](https://docs.dapr.io/operations/components/component-schema/) natif. L’exemple ci-dessus utilise le schéma pris en charge.
>
> Dans une application de niveau production, suivez les instructions de [gestion des secrets](https://docs.dapr.io/operations/components/component-secrets) pour gérer vos secrets de manière sécurisée.

Enregistrez le fichier suivant en tant que *clientapp.json* :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "defaultValue": "canadacentral",
            "type": "String"
        },
        "environment_name": {
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "name": "pythonapp",
            "type": "Microsoft.Web/containerApps",
            "apiVersion": "2021-03-01",
            "kind": "containerapp",
            "location": "[parameters('location')]",
            "properties": {
                "kubeEnvironmentId": "[resourceId('Microsoft.Web/kubeEnvironments', parameters('environment_name'))]",
                "configuration": {},
                "template": {
                    "containers": [
                        {
                            "image": "dapriosamples/hello-k8s-python:latest",
                            "name": "hello-k8s-python",
                            "resources": {
                                "cpu": 0.5,
                                "memory": "1Gi"
                            }
                        }
                    ],
                    "scale": {
                        "minReplicas": 1,
                        "maxReplicas": 1
                    },
                    "dapr": {
                        "enabled": true,
                        "appId": "pythonapp"
                    }
                }
            }
        }
    ]
}
```

## <a name="deploy-the-service-application-http-web-server"></a>Déployer l’application de service (serveur web HTTP)

Accédez au répertoire dans lequel vous avez stocké le fichier de modèle ARM et exécutez la commande ci-dessous pour déployer l’application de conteneur de service.

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az deployment group create \
  --resource-group "$RESOURCE_GROUP" \
  --template-file ./serviceapp.json \
  --parameters \
      environment_name="$CONTAINERAPPS_ENVIRONMENT" \
      location="$LOCATION" \
      storage_account_name="$STORAGE_ACCOUNT" \
      storage_account_key="$STORAGE_ACCOUNT_KEY" \
      storage_container_name="$STORAGE_ACCOUNT_CONTAINER"
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurecli
az deployment group create `
  --resource-group "$RESOURCE_GROUP" `
  --template-file ./serviceapp.json `
  --parameters `
      environment_name="$CONTAINERAPPS_ENVIRONMENT" `
      location="$LOCATION" `
      storage_account_name="$STORAGE_ACCOUNT" `
      storage_account_key="$STORAGE_ACCOUNT_KEY" `
      storage_container_name="$STORAGE_ACCOUNT_CONTAINER"
```

---

Cette commande déploie le serveur d’applications de service (nœud) sur `targetPort: 3000` (le port de l’application) avec son side-car Dapr associé configuré avec `"appId": "nodeapp",` et dapr `"appPort": 3000,` pour la détection et l’appel du service. Votre magasin d’état est configuré à l’aide de l’objet `components` de `"type": "state.azure.blobstorage"`, ce qui permet au side-car de conserver l’état.

## <a name="deploy-the-client-application-headless-client"></a>Déployer l’application cliente (client administré à distance)

Exécutez la commande ci-dessous pour déployer l’application de conteneur client.

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az deployment group create --resource-group "$RESOURCE_GROUP" \
  --template-file ./clientapp.json \
  --parameters \
    environment_name="$CONTAINERAPPS_ENVIRONMENT" \
    location="$LOCATION"
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurecli
az deployment group create --resource-group "$RESOURCE_GROUP" `
  --template-file ./clientapp.json `
  --parameters `
    environment_name="$CONTAINERAPPS_ENVIRONMENT" `
    location="$LOCATION"
```

---

Cette commande déploie `pythonapp`, qui s’exécute également avec un side-car Dapr qui est utilisé pour rechercher et appeler de manière sécurisée le side-car Dapr pour `nodeapp`. Cette application étant administrée à distance, il n’y a pas de `targetPort` pour démarrer un serveur, et il n’est pas non plus nécessaire d’activer l’entrée.

## <a name="verify-the-result"></a>Vérifier le résultat

### <a name="confirm-successful-state-persistence"></a>Confirmer la réussite de la persistance de l’état

Vous pouvez vérifier que les services fonctionnent correctement en affichant les données de votre compte Stockage Azure.

1. Ouvrez le [portail Azure](https://portal.azure.com) dans votre navigateur et accédez à votre compte de stockage.

1. Sélectionnez **Conteneurs** à gauche.

1. Sélectionnez **mycontainer**.

1. Vérifiez que vous pouvez voir le fichier nommé `order` dans le conteneur.

1. Cliquez sur le fichier.

1. Cliquez sur l’onglet **Modifier**.

1. Cliquez sur le bouton **Actualiser** pour observer les mises à jour.

### <a name="view-logs"></a>Voir les journaux

Les données journalisées via une application conteneur sont stockées dans la table personnalisée `ContainerAppConsoleLogs_CL` de l’espace de travail Log Analytics. Vous pouvez voir les journaux dans le portail Azure ou avec l’interface CLI. Avant que vous puissiez interroger les données journalisées, vous devrez peut-être patienter quelques minutes avant que l’analyse n’arrive pour la première fois.

Utilisez la commande CLI suivante pour voir les journaux sur la ligne de commande.

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az monitor log-analytics query \
  --workspace $LOG_ANALYTICS_WORKSPACE_CLIENT_ID \
  --analytics-query "ContainerAppConsoleLogs_CL | where ContainerAppName_s == 'nodeapp' and (Log_s contains 'persisted' or Log_s contains 'order') | project ContainerAppName_s, Log_s, TimeGenerated | take 5" \
  --out table
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurecli
az monitor log-analytics query `
  --workspace $LOG_ANALYTICS_WORKSPACE_CLIENT_ID `
  --analytics-query "ContainerAppConsoleLogs_CL | where ContainerAppName_s == 'nodeapp' and (Log_s contains 'persisted' or Log_s contains 'order') | project ContainerAppName_s, Log_s, TimeGenerated | take 5" `
  --out table
```

---

La sortie suivante montre le type de réponse de la commande CLI.

```console
ContainerAppName_s    Log_s                            TableName      TimeGenerated
--------------------  -------------------------------  -------------  ------------------------
nodeapp               Got a new order! Order ID: 61    PrimaryResult  2021-10-22T21:31:46.184Z
nodeapp               Successfully persisted state.    PrimaryResult  2021-10-22T21:31:46.184Z
nodeapp               Got a new order! Order ID: 62    PrimaryResult  2021-10-22T22:01:57.174Z
nodeapp               Successfully persisted state.    PrimaryResult  2021-10-22T22:01:57.174Z
nodeapp               Got a new order! Order ID: 63    PrimaryResult  2021-10-22T22:45:44.618Z
```

> [!TIP]
> Vous rencontrez des problèmes ? Faites-le nous savoir sur GitHub en ouvrant un problème dans le [dépôt Azure Container Apps](https://github.com/microsoft/azure-container-apps).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Une fois que vous avez terminé, nettoyez les ressources de votre application de conteneur en exécutant la commande suivante pour supprimer votre groupe de ressources.

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az group delete \
    --resource-group $RESOURCE_GROUP
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```azurecli
az group delete `
    --resource-group $RESOURCE_GROUP
```

---

Cette commande supprime les applications de conteneur, le compte de stockage, l’environnement d’applications de conteneur et toutes les autres ressources du groupe de ressources.

> [!NOTE]
> Étant donné que `pythonapp` effectue continuellement des appels à `nodeapp` avec des messages qui sont conservés dans votre magasin d’état configuré, il est important d’effectuer ces étapes de nettoyage pour éviter que des opérations ne vous soient facturées.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Gestion du cycle de vie des applications](application-lifecycle-management.md)
