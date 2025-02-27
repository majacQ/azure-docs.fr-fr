---
title: Importer les images conteneur
description: Importez des images conteneur dans un registre de conteneurs Azure à l’aide d’API Azure sans avoir à exécuter de commandes Docker.
ms.topic: article
ms.date: 09/13/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 14d2008599c0740bd36108760e3d4e50054b5f4d
ms.sourcegitcommit: f6e2ea5571e35b9ed3a79a22485eba4d20ae36cc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/24/2021
ms.locfileid: "128582266"
---
# <a name="import-container-images-to-a-container-registry"></a>Importer des images conteneur dans un registre de conteneurs

Vous pouvez facilement importer (copier) des images conteneur dans un registre de conteneurs Azure sans recourir à des commandes Docker. Vous pouvez par exemple importer des images d’un registre de développement dans un registre de production ou copier des images de base à partir d’un registre public.

Azure Container Registry gère un certain nombre de scénarios courants de copie d’images et d’autre artefacts à partir d’un registre existant :

* Importer des images à partir d’un registre public

* Importez des images ou des artefacts OCI, dont des graphiques Helm 3 à partir d’un autre Azure Container Registry, dans le même ou dans un autre abonnement ou locataire Azure

* Importer à partir d’un registre de conteneurs privé non-Azure

L’importation d’images dans un registre de conteneurs Azure présente les avantages suivants par rapport à l’utilisation de commandes de l’interface CLI Docker :

* Étant donné que votre environnement client ne nécessite pas d’installation Docker locale, vous pouvez importer n’importe quelle image conteneur, quel que soit le type de système d’exploitation pris en charge.

* Quand vous importez des images multi-architecture (notamment des images Docker officielles), les images pour toutes les architectures et plateformes spécifiées dans la liste de manifeste sont copiées.

* L'accès au registre source ne doit pas nécessairement utiliser le point de terminaison public du registre.

## <a name="limitations"></a>Limites

* Le nombre maximum de manifestes pour une image importée est de 50.
* La taille maximale des couches pour une image importée d'un registre public est de 2 Gio.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour importer des images conteneur, cet article nécessite que vous exécutiez l’interface Azure CLI dans Azure Cloud Shell ou localement (version 2.0.55 ou ultérieure recommandée). Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli].

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Pour importer des images de conteneur, cet article exige que vous exécutiez Azure PowerShell dans Azure Cloud Shell ou localement (version 5.9.0 ou ultérieure recommandée). Exécutez `Get-InstalledModule -Name Az` pour trouver la version. Si vous devez installer ou mettre à niveau, consultez [Installer le module PowerShell Azure Az][install-the-azure-az-powershell-module].

---

[!INCLUDE [container-registry-geo-replication-include](../../includes/container-registry-geo-replication-include.md)]

> [!IMPORTANT]
> Les modifications apportées à l’importation d’image entre deux registres de conteneurs Azure ont été introduites à partir du mois de janvier 2021 :
> * L’importation vers ou depuis un registre de conteneurs Azure avec accès restreint au réseau requiert que le registre restreint [**autorise l’accès par des services de confiance**](allow-access-trusted-services.md) pour contourner le réseau. Par défaut, le paramètre est activé, ce qui permet l’importation. Si le paramètre n’est pas activé dans un registre nouvellement créé avec un point de terminaison privé ou avec des règles de pare-feu de registre, l’importation échoue.
> * Dans un registre de conteneurs Azure avec accès restreint au réseau existant utilisé comme source d’importation ou cible, l’activation de cette fonctionnalité de sécurité réseau est facultative, mais recommandée.

## <a name="prerequisites"></a>Prérequis

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Si vous ne disposez pas d’un registre de conteneurs Azure, créez-en un. Pour connaître les étapes à suivre, voir [Démarrage rapide : Créer un registre de conteneurs privé avec Azure CLI](container-registry-get-started-azure-cli.md).

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Si vous ne disposez pas d’un registre de conteneurs Azure, créez-en un. Pour les étapes, voir [Démarrage rapide : Créer un registre de conteneurs privé à l'aide d'Azure PowerShell](container-registry-get-started-powershell.md).

---

Pour importer une image dans un registre de conteneurs Azure, votre identité doit avoir des autorisations en écriture sur le registre cible (au moins le rôle Contributeur ou un rôle personnalisé permettant l’action importimage). Consultez [Autorisations et rôles Azure Container Registry](container-registry-roles.md#custom-roles).

## <a name="import-from-a-public-registry"></a>Importer à partir d’un registre public

### <a name="import-from-docker-hub"></a>Importer à partir de Docker Hub

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Par exemple, utilisez la commande [az acr import][az-acr-import] pour importer l’image `hello-world:latest` multi-architecture à partir de Docker Hub dans un registre nommé *myregistry*. `hello-world` étant une image officielle de Docker Hub, cette image se trouve dans le dépôt `library` par défaut. Indiquez le nom du dépôt et éventuellement une étiquette dans la valeur du paramètre d’image `--source`. (Vous pouvez également identifier une image par code de hachage de manifeste et non par étiquette, ce qui garantit l’obtention d’une version particulière.)

```azurecli
az acr import \
  --name myregistry \
  --source docker.io/library/hello-world:latest \
  --image hello-world:latest
```

Pour vérifier si plusieurs manifestes sont associés à cette image, exécutez la commande `az acr repository show-manifests` :

```azurecli
az acr repository show-manifests \
  --name myregistry \
  --repository hello-world
```

Si vous avez un [compte Docker Hub](https://www.docker.com/pricing), nous vous recommandons d’utiliser les informations d’identification lors de l’importation d’une image à partir de Docker Hub. Transmettez le nom d’utilisateur et le mot de passe du Docker Hub ou un [jeton d’accès personnel](https://docs.docker.com/docker-hub/access-tokens/) en tant que paramètres à `az acr import`. L’exemple suivant importe une image publique à partir du référentiel `tensorflow` dans Docker Hub, en utilisant les informations d'identification de Docker Hub :

```azurecli
az acr import \
  --name myregistry \
  --source docker.io/tensorflow/tensorflow:latest-gpu \
  --image tensorflow:latest-gpu
  --username <Docker Hub user name>
  --password <Docker Hub token>
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Par exemple, utilisez la commande [Import-AzContainerRegistryImage][import-azcontainerregistryimage] pour importer l'`hello-world:latest`image multi-architecture de Docker Hub vers un registre nommé *myregistry*. `hello-world` étant une image officielle de Docker Hub, cette image se trouve dans le dépôt `library` par défaut. Inclure le nom du référentiel et éventuellement une balise dans la valeur du `-SourceImage`paramètre. (Vous pouvez également identifier une image par code de hachage de manifeste et non par étiquette, ce qui garantit l’obtention d’une version particulière.)

```azurepowershell
Import-AzContainerRegistryImage -RegistryName myregistry -ResourceGroupName myResourceGroup -SourceRegistryUri docker.io -SourceImage library/hello-world:latest
```

Vous pouvez vérifier que plusieurs manifestes sont associés à cette image en exécutant la `Get-AzContainerRegistryManifest` cmdlet :

```azurepowershell
Get-AzContainerRegistryManifest -RepositoryName library/hello-world -RegistryName myregistry
```

Si vous avez un [compte Docker Hub](https://www.docker.com/pricing), nous vous recommandons d’utiliser les informations d’identification lors de l’importation d’une image à partir de Docker Hub. Transmettez le nom d’utilisateur et le mot de passe du Docker Hub ou un [jeton d’accès personnel](https://docs.docker.com/docker-hub/access-tokens/) en tant que paramètres à `Import-AzContainerRegistryImage`. L’exemple suivant importe une image publique à partir du référentiel `tensorflow` dans Docker Hub, en utilisant les informations d'identification de Docker Hub :

```azurepowershell
Import-AzContainerRegistryImage -RegistryName myregistry -ResourceGroupName myResourceGroup -SourceRegistryUri docker.io -SourceImage tensorflow/tensorflow:latest-gpu -Username <Docker Hub user name> -Password <Docker Hub token>
```

---

### <a name="import-from-microsoft-container-registry"></a>Importer à partir du registre de conteneurs Microsoft

Par exemple, importez l’`ltsc2019`image Windows Server Core à partir du référentiel `windows` dans le registre de conteneurs Microsoft.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az acr import \
--name myregistry \
--source mcr.microsoft.com/windows/servercore:ltsc2019 \
--image servercore:ltsc2019
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Import-AzContainerRegistryImage -RegistryName myregistry -ResourceGroupName myResourceGroup -SourceRegistryUri mcr.microsoft.com -SourceImage windows/servercore:ltsc2019
```

---

## <a name="import-from-an-azure-container-registry-in-the-same-ad-tenant"></a>Importer à partir d'un registre de conteneurs Azure situé dans le même locataire AD

Vous pouvez importer une image à partir d'un registre de conteneurs Azure situé dans le même locataire AD grâce aux autorisations Azure Active Directory intégrées.

* Votre identité doit disposer d’autorisations Azure Active Directory pour lire à partir du registre source (rôle Lecteur) et pour écrire dans le registre cible (rôle Contributeur ou [rôle personnalisé](container-registry-roles.md#custom-roles) permettant l’action importImage).

* Le registre peut se trouver dans un abonnement Azure identique ou différent dans le même locataire Active Directory.

* L’[accès public](container-registry-access-selected-networks.md#disable-public-network-access) au registre source peut être désactivé. Si l’accès public est désactivé, spécifiez le registre source par ID de ressource plutôt que par nom de serveur de connexion au registre.

* Si le registre source et/ou le registre cible ont un point de terminaison privé ou si des règles de pare-feu de registre sont appliqués, assurez-vous que le registre restreint [permet aux services de confiance](allow-access-trusted-services.md) d’accéder au réseau.

### <a name="import-from-a-registry-in-the-same-subscription"></a>Importer à partir d’un registre dans le même abonnement

Par exemple, importez l’image `aci-helloworld:latest` à partir d’un registre source *mysourceregistry* dans *myregistry* dans le même abonnement Azure.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az acr import \
  --name myregistry \
  --source mysourceregistry.azurecr.io/aci-helloworld:latest \
  --image aci-helloworld:latest
```

L’exemple suivant importe l’image `aci-helloworld:latest` vers *myregistry* à partir d’un registre source *mysourceregistry* dans lequel l’accès au point de terminaison public du registre est désactivé. Indiquez l’ID de ressource du registre source avec le paramètre `--registry`. Notez que le paramètre `--source` spécifie uniquement le dépôt source et l’étiquette, pas le nom du serveur de connexion au registre.

```azurecli
az acr import \
  --name myregistry \
  --source aci-helloworld:latest \
  --image aci-helloworld:latest \
  --registry /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/sourceResourceGroup/providers/Microsoft.ContainerRegistry/registries/mysourceregistry
```

L’exemple suivant importe une image par code de hachage de manifeste (hachage SHA-256, représenté sous la forme `sha256:...`), et non par étiquette :

```azurecli
az acr import \
  --name myregistry \
  --source mysourceregistry.azurecr.io/aci-helloworld@sha256:123456abcdefg
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Import-AzContainerRegistryImage -RegistryName myregistry -ResourceGroupName myResourceGroup -SourceRegistryUri mysourceregistry.azurecr.io -SourceImage aci-helloworld:latest
```

L’exemple suivant importe l’image `aci-helloworld:latest` vers *myregistry* à partir d’un registre source *mysourceregistry* dans lequel l’accès au point de terminaison public du registre est désactivé. Indiquez l’ID de ressource du registre source avec le paramètre `--registry`. Notez que le paramètre `--source` spécifie uniquement le dépôt source et l’étiquette, pas le nom du serveur de connexion au registre.

```azurepowershell
Import-AzContainerRegistryImage -RegistryName myregistry -ResourceGroupName myResourceGroup -SourceRegistryResourceId '/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/sourceResourceGroup/providers/Microsoft.ContainerRegistry/registries/mysourceregistry' -SourceImage aci-helloworld:latest
```

L’exemple suivant importe une image par code de hachage de manifeste (hachage SHA-256, représenté sous la forme `sha256:...`), et non par étiquette :

```azurepowershell
Import-AzContainerRegistryImage -RegistryName myregistry -ResourceGroupName myResourceGroup -SourceRegistryUri mysourceregistry.azurecr.io -SourceImage aci-helloworld@sha256:123456abcdefg
```

---

### <a name="import-from-a-registry-in-a-different-subscription"></a>Importer à partir d’un registre dans un abonnement différent

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Dans l’exemple suivant, *mysourceregistry* est dans un autre abonnement que *myregistry*, dans le même locataire Active Directory. Indiquez l’ID de ressource du registre source avec le paramètre `--registry`. Notez que le paramètre `--source` spécifie uniquement le dépôt source et l’étiquette, pas le nom du serveur de connexion au registre.

```azurecli
az acr import \
  --name myregistry \
  --source samples/aci-helloworld:latest \
  --image aci-hello-world:latest \
  --registry /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/sourceResourceGroup/providers/Microsoft.ContainerRegistry/registries/mysourceregistry
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Dans l’exemple suivant, *mysourceregistry* est dans un autre abonnement que *myregistry*, dans le même locataire Active Directory. Indiquez l’ID de ressource du registre source avec le paramètre `--registry`. Notez que le paramètre `--source` spécifie uniquement le dépôt source et l’étiquette, pas le nom du serveur de connexion au registre.

```azurepowershell
Import-AzContainerRegistryImage -RegistryName myregistry -ResourceGroupName myResourceGroup -SourceRegistryResourceId '/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/sourceResourceGroup/providers/Microsoft.ContainerRegistry/registries/mysourceregistry' -SourceImage aci-helloworld:latest
```

---

### <a name="import-from-a-registry-using-service-principal-credentials"></a>Importer à partir d’un registre à l’aide des informations d’identification du principal de service

Pour importer à partir d'un registre auquel vous n'avez pas accès avec les autorisations Active Directory intégrées, vous pouvez utiliser les informations d'identification du principal de service (si celles-ci sont disponibles) pour le registre source. Indiquez l’appID et le mot de passe d’un [principal de service](container-registry-auth-service-principal.md) Active Directory disposant d’un accès ACRPull au registre source. L’utilisation d’un principal de service est utile pour les systèmes de génération et d’autres systèmes sans assistance qui doivent importer des images dans votre registre.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az acr import \
  --name myregistry \
  --source sourceregistry.azurecr.io/sourcerrepo:tag \
  --image targetimage:tag \
  --username <SP_App_ID> \
  --password <SP_Passwd>
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Import-AzContainerRegistryImage -RegistryName myregistry -ResourceGroupName myResourceGroup -SourceRegistryUri sourceregistry.azurecr.io -SourceImage sourcerrepo:tag -Username <SP_App_ID> -Password <SP_Passwd>
```

---

## <a name="import-from-an-azure-container-registry-in-a-different-ad-tenant"></a>Importer à partir d'un registre de conteneurs Azure situé dans un autre locataire AD

Pour importer à partir d’un registre de conteneurs Azure situé dans un autre locataire Azure Active Directory, spécifiez le registre source par nom de serveur de connexion et entrez des informations d’identification permettant un accès par tirage (pull) au registre.

### <a name="cross-tenant-import-with-username-and-password"></a>Importation entre locataires avec nom d’utilisateur et mot de passe
Par exemple, utilisez un [jeton délimité par le référentiel](container-registry-repository-scoped-permissions.md) et un mot de passe, ou l'appID et le mot de passe d'un [principal de service](container-registry-auth-service-principal.md) Active Directory disposant d'un accès ACRPull au registre source.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az acr import \
  --name myregistry \
  --source sourceregistry.azurecr.io/sourcerrepo:tag \
  --image targetimage:tag \
  --username <SP_App_ID> \
  --password <SP_Passwd>
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Import-AzContainerRegistryImage -RegistryName myregistry -ResourceGroupName myResourceGroup -SourceRegistryUri sourceregistry.azurecr.io -SourceImage sourcerrepo:tag -Username <SP_App_ID> -Password <SP_Passwd>
```

---

### <a name="cross-tenant-import-with-access-token"></a>Importation entre locataires avec jeton d’accès

Pour accéder au registre source à l’aide d’une identité dans le locataire source qui dispose d’autorisations de registre, vous pouvez obtenir un jeton d’accès :

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
# Login to Azure CLI with the identity, for example a user-assigned managed identity
az login --identity --username <identity_ID>

# Get access token returned by `az account get-access-token`
az account get-access-token
```

Dans le locataire cible, transmettez le jeton d’accès en tant que mot de passe à la commande `az acr import`. Le registre source est spécifié par le nom du serveur de connexion. Notez qu’aucun nom d’utilisateur n’est nécessaire dans cette commande :

```azurecli
az acr import \
  --name myregistry \
  --source sourceregistry.azurecr.io/sourcerrepo:tag \
  --image targetimage:tag \
  --password <access-token>
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
# Login to Azure PowerShell with the identity, for example a user-assigned managed identity
Connect-AzAccount -Identity -AccountId <identity_ID>

# Get access token returned by `Get-AzAccessToken`
Get-AzAccessToken
```

Dans le locataire cible, passez le jeton d'accès comme mot de passe à la `Import-AzContainerRegistryImage` cmdlet. Le registre source est spécifié par le nom du serveur de connexion. Notez qu’aucun nom d’utilisateur n’est nécessaire dans cette commande :

```azurepowershell
Import-AzContainerRegistryImage -RegistryName myregistry -ResourceGroupName myResourceGroup -SourceRegistryUri sourceregistry.azurecr.io -SourceImage sourcerrepo:tag -Password <access-token>
```

---

## <a name="import-from-a-non-azure-private-container-registry"></a>Importer à partir d’un registre de conteneurs privé non-Azure

Importez une image à partir d'un registre privé non Azure en spécifiant des informations d'identification permettant un accès par extraction au registre. Par exemple, tirez (pull) une image à partir d’un registre Docker privé :

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az acr import \
  --name myregistry \
  --source docker.io/sourcerepo/sourceimage:tag \
  --image sourceimage:tag \
  --username <username> \
  --password <password>
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)
```azurepowershell
Import-AzContainerRegistryImage -RegistryName myregistry -ResourceGroupName myResourceGroup -SourceRegistryUri docker.io/sourcerepo -SourceImage sourcerrepo:tag -Username <username> -Password <password>
```

---

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à importer des images conteneur dans un registre de conteneurs Azure à partir d’un registre public ou d’un autre registre privé.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

* Pour accéder à des options d’importation d’image supplémentaires, consultez les informations de référence sur la commande [az acr import][az-acr-import].

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

* Pour connaître les autres options d'importation d'images, consultez la référence du cmdlet [Import-AzContainerRegistryImage][import-azcontainerregistryimage].

---

* L’importation d’images peut vous aider à déplacer du contenu vers un registre de conteneurs dans une région Azure, un abonnement ou un locataire Azure AD différent. Pour plus d’informations, consultez [Déplacer manuellement un registre de conteneurs vers une autre région](manual-regional-move.md).

* Découvrez comment [désactiver l’exportation d’artefacts](data-loss-prevention.md) à partir d’un registre de conteneurs restreint par le réseau.


<!-- LINKS - Internal -->
[az-login]: /cli/azure/reference-index#az_login
[az-acr-import]: /cli/azure/acr#az_acr_import
[azure-cli]: /cli/azure/install-azure-cli
[install-the-azure-az-powershell-module]: /powershell/azure/install-az-ps
[import-azcontainerregistryimage]: /powershell/module/az.containerregistry/import-azcontainerregistryimage
