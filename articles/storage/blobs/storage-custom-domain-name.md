---
title: Mapper un domaine personnalisé à un point de terminaison de Stockage Blob Azure
titleSuffix: Azure Storage
description: Mapper un domaine personnalisé à un point de terminaison de Stockage Blob ou un point de terminaison web dans un compte de stockage Azure.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 02/12/2021
ms.author: normesta
ms.reviewer: dineshm
ms.subservice: blobs
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e56a36947930894548a4490320c48efab0509ef1
ms.sourcegitcommit: f6e2ea5571e35b9ed3a79a22485eba4d20ae36cc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/24/2021
ms.locfileid: "128662556"
---
# <a name="map-a-custom-domain-to-an-azure-blob-storage-endpoint"></a>Mapper un domaine personnalisé à un point de terminaison de Stockage Blob Azure

Vous pouvez mapper un domaine personnalisé à un point de terminaison de service blob ou de [site web statique](storage-blob-static-website.md).

> [!NOTE]
> Ce mappage fonctionne uniquement pour des sous-domaines (par exemple, `www.contoso.com`). Si vous souhaitez que votre point de terminaison web soit disponible sur le domaine racine (par exemple, `contoso.com`), vous devez utiliser Azure CDN. Pour obtenir de l’aide, voir la section [Mapper un domaine personnalisé avec le protocole HTTPS activé](#enable-https) de cet article. Étant donné que vous accédez à cette section de l’article pour activer le domaine racine de votre domaine personnalisé, l’étape de cette section visant à activer le protocole HTTPS est facultative.

<a id="enable-http"></a>

## <a name="map-a-custom-domain-with-only-http-enabled"></a>Mapper un domaine personnalisé avec uniquement le protocole HTTP activé

Cette approche est plus simple mais active uniquement l’accès HTTP. Si le compte de stockage est configuré pour [exiger un transfert sécurisé](../common/storage-require-secure-transfer.md) sur HTTPS, vous devez activer l’accès HTTPS pour votre domaine personnalisé.

Pour activer l’accès HTTPS, voir la section [Mapper un domaine personnalisé avec le protocole HTTPS activé](#enable-https) de cet article.

<a id="map-a-domain"></a>

### <a name="map-a-custom-domain"></a>Mapper un domaine personnalisé

> [!IMPORTANT]
> Votre domaine personnalisé est brièvement indisponible pour les utilisateurs pendant que vous effectuez la configuration. Si votre domaine prend actuellement en charge une application adossée à un contrat de niveau de service (SLA) interdisant tout temps d’arrêt, suivez les étapes décrites dans la section [Mapper un domaine personnalisé sans temps d’arrêt](#zero-down-time) de cet article pour vous assurer que les utilisateurs puissent accéder à votre domaine pendant l’exécution du mappage DNS.

Si vous ne vous souciez pas du fait que le domaine soit brièvement indisponible pour vos utilisateurs, procédez comme suit.

:heavy_check_mark: Étape 1 : Récupérez le nom d’hôte de votre point de terminaison de stockage.

:heavy_check_mark: Étape 2 : Créez un enregistrement de nom canonique (CNAME) auprès de votre fournisseur de domaine.

:heavy_check_mark: Étape 3 : Inscrivez le domaine personnalisé auprès d’Azure.

:heavy_check_mark: Étape 4 : Testez votre domaine personnalisé.

<a id="endpoint"></a>

#### <a name="step-1-get-the-host-name-of-your-storage-endpoint"></a>Étape 1 : Récupérez le nom d’hôte de votre point de terminaison de stockage.

Le nom d’hôte est l’URL du point de terminaison de stockage sans l’identificateur de protocole et la barre oblique finale.

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre compte de stockage.

2. Dans le volet du menu, sous **Paramètres**, sélectionnez **Points de terminaison**.

3. Copiez la valeur du point de terminaison **Service Blob** ou du point de terminaison **Site web statique** dans un fichier texte.

   > [!NOTE]
   > Le point de terminaison de stockage Data Lake n’est pas pris en charge (par exemple, `https://mystorageaccount.dfs.core.windows.net/`).

4. Supprimez de cette chaîne l’identificateur de protocole (par exemple, `HTTPS`) et la barre oblique finale. Le tableau suivant contient des exemples.

   | Type de point de terminaison |  endpoint | nom de l’hôte |
   |------------|-----------------|-------------------|
   |service blob  | `https://mystorageaccount.blob.core.windows.net/` | `mystorageaccount.blob.core.windows.net` |
   |site web statique  | `https://mystorageaccount.z5.web.core.windows.net/` | `mystorageaccount.z5.web.core.windows.net` |

   Mettez cette valeur de côté pour plus tard.

<a id="create-cname-record"></a>

#### <a name="step-2-create-a-canonical-name-cname-record-with-your-domain-provider"></a>Étape 2 : Créez un enregistrement de nom canonique (CNAME) auprès de votre fournisseur de domaine.

Créez un enregistrement CNAME pointant vers votre nom d’hôte. Un enregistrement CNAME est un type d’enregistrement DNS qui mappe un nom de domaine source à un nom de domaine de destination.

1. Connectez-vous au site web de votre bureau d’enregistrement de domaines, puis accédez à la page de gestion du paramètre DNS.

   Vous pourrez trouver cette page dans une section intitulée **Domain Name**, **DNS** ou **Name Server Management**.

2. Recherchez la section dédiée à la gestion des enregistrements CNAME.

   Pour cela, accédez à une page de paramètres avancés et recherchez **CNAME**, **Alias** ou **Sous-domaines**.

3. Créez un enregistrement CNAME. Dans le cadre de cet enregistrement, fournissez les éléments suivants :

   - L’alias de sous-domaine tel que `www` ou `photos`. Le sous-domaine est requis, les domaines racines ne sont pas pris en charge.

   - Le nom d’hôte que vous avez obtenu dans le cadre de la section [Récupérez le nom d’hôte de votre point de terminaison de stockage](#endpoint) plus haut dans cet article.

<a id="register"></a>

#### <a name="step-3-register-your-custom-domain-with-azure"></a>Étape 3 : Inscrivez votre domaine personnalisé auprès d’Azure.

##### <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre compte de stockage.

2. Dans le volet du menu, sous **Sécurité + mise en réseau**, sélectionnez **Mise en réseau**.

3. Dans la page **Réseau**, choisissez l’onglet **Domaine personnalisé**.

   > [!NOTE]
   > Cette option ne figure pas dans les comptes pour lesquels la fonctionnalité d’espace de noms hiérarchique est activée. Avec ces comptes, utilisez PowerShell ou Azure CLI pour effectuer cette étape.

3. Dans la zone de texte **Nom de domaine**, entrez le nom de votre domaine personnalisé, en incluant le sous-domaine.

   Par exemple, si votre domaine est *contoso.com* et votre alias de sous-domaine *www*, entrez `www.contoso.com`. Si votre sous-domaine est *photos*, entrez `photos.contoso.com`.

4. Pour inscrire le domaine personnalisé, choisissez le bouton **Enregistrer**.

   Une fois l’enregistrement CNAME propagé via les serveurs de noms de domaine (DNS), vos utilisateurs disposant des autorisations appropriées peuvent afficher les données blob à l’aide du domaine personnalisé.

##### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Exécutez la commande PowerShell suivante :

```powershell
Set-AzStorageAccount -ResourceGroupName <resource-group-name> -Name <storage-account-name> -CustomDomainName <custom-domain-name> -UseSubDomain $false
```

- Remplacez l’espace réservé `<resource-group-name>` par le nom du groupe de ressources.

- Remplacez l’espace réservé `<storage-account-name>` par le nom du compte de stockage.

- Remplacez l’espace réservé `<custom-domain-name>` par le nom de votre domaine personnalisé, y compris le sous-domaine.

  Par exemple, si votre domaine est *contoso.com* et votre alias de sous-domaine *www*, entrez `www.contoso.com`. Si votre sous-domaine est *photos*, entrez `photos.contoso.com`.

Une fois l’enregistrement CNAME propagé via les serveurs de noms de domaine (DNS), vos utilisateurs disposant des autorisations appropriées peuvent afficher les données blob à l’aide du domaine personnalisé.

##### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Exécutez la commande PowerShell suivante :

```azurecli
az storage account update \
   --resource-group <resource-group-name> \ 
   --name <storage-account-name> \
   --custom-domain <custom-domain-name> \
   --use-subdomain false
  ```

- Remplacez l’espace réservé `<resource-group-name>` par le nom du groupe de ressources.

- Remplacez l’espace réservé `<storage-account-name>` par le nom du compte de stockage.

- Remplacez l’espace réservé `<custom-domain-name>` par le nom de votre domaine personnalisé, y compris le sous-domaine.

  Par exemple, si votre domaine est *contoso.com* et votre alias de sous-domaine *www*, entrez `www.contoso.com`. Si votre sous-domaine est *photos*, entrez `photos.contoso.com`.

Une fois l’enregistrement CNAME propagé via les serveurs de noms de domaine (DNS), vos utilisateurs disposant des autorisations appropriées peuvent afficher les données blob à l’aide du domaine personnalisé.

---

#### <a name="step-4-test-your-custom-domain"></a>Étape 4 : Tester votre domaine personnalisé

Pour confirmer que votre domaine personnalisé est mappé à votre point de terminaison de service blob, créez un objet blob dans un conteneur public au sein de votre compte de stockage. Dans un navigateur web, utilisez un URI au format suivant pour accéder à l’objet blob : `http://<subdomain.customdomain>/<mycontainer>/<myblob>`

Pour accéder à un formulaire web dans le conteneur `myforms` dans le sous-domaine personnalisé *photos.contoso.com*, vous pouvez par exemple utiliser l’URI suivant : `http://photos.contoso.com/myforms/applicationform.htm`

<a id="zero-down-time"></a>

### <a name="map-a-custom-domain-with-zero-downtime"></a>Mapper un domaine personnalisé sans temps d’arrêt

> [!NOTE]
> Si vous ne vous souciez pas du fait que le domaine soit brièvement indisponible pour vos utilisateurs, vous pouvez suivre les étapes décrites dans la section [Mapper un domaine personnalisé](#map-a-domain) de cet article. Il s’agit d’une approche plus simple, nécessitant moins d’étapes.

Si votre domaine prend actuellement en charge une application adossée à un contrat de niveau de service (SLA) interdisant tout temps d’arrêt, suivez ces étapes pour vous assurer que les utilisateurs puissent accéder à votre domaine pendant l’exécution du mappage DNS.

:heavy_check_mark: Étape 1 : Récupérez le nom d’hôte de votre point de terminaison de stockage.

:heavy_check_mark: Étape 2 : Créez un enregistrement de nom canonique (CNAME) intermédiaire auprès de votre fournisseur de domaine.

:heavy_check_mark: Étape 3 : Pré-inscrivez le domaine personnalisé auprès d’Azure.

:heavy_check_mark: Étape 4 : Créez un enregistrement CNAME auprès de votre fournisseur de domaine.

:heavy_check_mark: Étape 5 : Testez votre domaine personnalisé.

<a id="endpoint-2"></a>

#### <a name="step-1-get-the-host-name-of-your-storage-endpoint"></a>Étape 1 : Récupérez le nom d’hôte de votre point de terminaison de stockage.

Le nom d’hôte est l’URL du point de terminaison de stockage sans l’identificateur de protocole et la barre oblique finale.

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre compte de stockage.

2. Dans le volet du menu, sous **Paramètres**, sélectionnez **Points de terminaison**.

3. Copiez la valeur du point de terminaison **Service Blob** ou du point de terminaison **Site web statique** dans un fichier texte.

   > [!NOTE]
   > Le point de terminaison de stockage Data Lake n’est pas pris en charge (par exemple, `https://mystorageaccount.dfs.core.windows.net/`).

4. Supprimez de cette chaîne l’identificateur de protocole (par exemple, `HTTPS`) et la barre oblique finale. Le tableau suivant contient des exemples.

   | Type de point de terminaison |  endpoint | nom de l’hôte |
   |------------|-----------------|-------------------|
   |service blob  | `https://mystorageaccount.blob.core.windows.net/` | `mystorageaccount.blob.core.windows.net` |
   |site web statique  | `https://mystorageaccount.z5.web.core.windows.net/` | `mystorageaccount.z5.web.core.windows.net` |

   Mettez cette valeur de côté pour plus tard.

#### <a name="step-2-create-an-intermediary-canonical-name-cname-record-with-your-domain-provider"></a>Étape 2 : Créer un enregistrement de nom canonique (CNAME) intermédiaire auprès de votre fournisseur de domaine

Créez un enregistrement CNAME temporaire pointant vers votre nom d’hôte. Un enregistrement CNAME est un type de fonctionnalité DNS (Domain Name System) qui mappe un nom de domaine source à un nom de domaine de destination.

1. Connectez-vous au site web de votre bureau d’enregistrement de domaines, puis accédez à la page de gestion du paramètre DNS.

   Vous pourrez trouver cette page dans une section intitulée **Domain Name**, **DNS** ou **Name Server Management**.

2. Recherchez la section dédiée à la gestion des enregistrements CNAME.

   Pour cela, accédez à une page de paramètres avancés et recherchez **CNAME**, **Alias** ou **Sous-domaines**.

3. Créez un enregistrement CNAME. Dans le cadre de cet enregistrement, fournissez les éléments suivants :

   - L’alias de sous-domaine tel que `www` ou `photos`. Le sous-domaine est requis, les domaines racines ne sont pas pris en charge.

     Ajoutez le sous-domaine `asverify` à l’alias. Par exemple, `asverify.www` ou `asverify.photos`.

   - Le nom d’hôte que vous avez obtenu dans le cadre de la section [Récupérez le nom d’hôte de votre point de terminaison de stockage](#endpoint) plus haut dans cet article.

     Ajoutez le sous-domaine `asverify` au nom d’hôte. Par exemple : `asverify.mystorageaccount.blob.core.windows.net`.

#### <a name="step-3-pre-register-your-custom-domain-with-azure"></a>Étape 3 : Pré-inscrivez votre domaine personnalisé auprès d’Azure.

Lorsque vous pré-enregistrez votre domaine personnalisé auprès d’Azure, vous permettez à Azure de reconnaître votre domaine personnalisé sans avoir à modifier l’enregistrement DNS du domaine. Ainsi, lorsque vous modifiez l’enregistrement DNS pour le domaine, il est mappé au point de terminaison d’objet blob sans temps d’arrêt.

##### <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre compte de stockage.

2. Dans le volet du menu, sous **Sécurité + mise en réseau**, sélectionnez **Mise en réseau**.

3. Dans la page **Réseau**, choisissez l’onglet **Domaine personnalisé**.

   > [!NOTE]
   > Cette option ne figure pas dans les comptes pour lesquels la fonctionnalité d’espace de noms hiérarchique est activée. Avec ces comptes, utilisez PowerShell ou Azure CLI pour effectuer cette étape.

3. Dans la zone de texte **Nom de domaine**, entrez le nom de votre domaine personnalisé, en incluant le sous-domaine.

   Par exemple, si votre domaine est *contoso.com* et votre alias de sous-domaine *www*, entrez `www.contoso.com`. Si votre sous-domaine est *photos*, entrez `photos.contoso.com`.

4. Cochez la case **Utiliser la validation CNAME indirecte**.

5. Pour inscrire le domaine personnalisé, choisissez le bouton **Enregistrer**.

   Si l’inscription réussit, une notification de portail apparaît, indiquant que votre compte de stockage a été correctement mis à jour. Votre domaine personnalisé a été vérifié par Azure, toutefois, le trafic en direction de votre domaine ne sera routé vers votre compte de stockage que lorsque vous aurez créé un enregistrement CNAME avec votre fournisseur de domaine. Vous le ferez dans la section suivante.

##### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Exécutez la commande PowerShell suivante :

```powershell
Set-AzStorageAccount -ResourceGroupName <resource-group-name> -Name <storage-account-name> -CustomDomainName <custom-domain-name> -UseSubDomain $true
```

- Remplacez l’espace réservé `<resource-group-name>` par le nom du groupe de ressources.

- Remplacez l’espace réservé `<storage-account-name>` par le nom du compte de stockage.

- Remplacez l’espace réservé `<custom-domain-name>` par le nom de votre domaine personnalisé, y compris le sous-domaine.

  Par exemple, si votre domaine est *contoso.com* et votre alias de sous-domaine *www*, entrez `www.contoso.com`. Si votre sous-domaine est *photos*, entrez `photos.contoso.com`.

Le trafic en direction de votre domaine ne sera routé vers votre compte de stockage que lorsque vous aurez créé un enregistrement CNAME avec votre fournisseur de domaine. Vous le ferez dans la section suivante.

##### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Exécutez la commande PowerShell suivante :

```azurecli
az storage account update \
   --resource-group <resource-group-name> \ 
   --name <storage-account-name> \
   --custom-domain <custom-domain-name> \
   --use-subdomain true
  ```

- Remplacez l’espace réservé `<resource-group-name>` par le nom du groupe de ressources.

- Remplacez l’espace réservé `<storage-account-name>` par le nom du compte de stockage.

- Remplacez l’espace réservé `<custom-domain-name>` par le nom de votre domaine personnalisé, y compris le sous-domaine.

  Par exemple, si votre domaine est *contoso.com* et votre alias de sous-domaine *www*, entrez `www.contoso.com`. Si votre sous-domaine est *photos*, entrez `photos.contoso.com`.

Le trafic en direction de votre domaine ne sera routé vers votre compte de stockage que lorsque vous aurez créé un enregistrement CNAME avec votre fournisseur de domaine. Vous le ferez dans la section suivante.

---

#### <a name="step-4-create-a-cname-record-with-your-domain-provider"></a>Étape 4 : Créez un enregistrement CNAME auprès de votre fournisseur de domaine.

Créez un enregistrement CNAME temporaire pointant vers votre nom d’hôte.

1. Connectez-vous au site web de votre bureau d’enregistrement de domaines, puis accédez à la page de gestion du paramètre DNS.

   Vous pourrez trouver cette page dans une section intitulée **Domain Name**, **DNS** ou **Name Server Management**.

2. Recherchez la section dédiée à la gestion des enregistrements CNAME.

   Pour cela, accédez à une page de paramètres avancés et recherchez **CNAME**, **Alias** ou **Sous-domaines**.

3. Créez un enregistrement CNAME. Dans le cadre de cet enregistrement, fournissez les éléments suivants :

   - L’alias de sous-domaine tel que `www` ou `photos`. Le sous-domaine est requis, les domaines racines ne sont pas pris en charge.

   - Le nom d’hôte que vous avez obtenu dans le cadre de la section [Récupérez le nom d’hôte de votre point de terminaison de stockage](#endpoint-2) plus haut dans cet article.

#### <a name="step-5-test-your-custom-domain"></a>Étape 5 : Tester votre domaine personnalisé

Pour confirmer que votre domaine personnalisé est mappé à votre point de terminaison de service blob, créez un objet blob dans un conteneur public au sein de votre compte de stockage. Dans un navigateur web, utilisez un URI au format suivant pour accéder à l’objet blob : `http://<subdomain.customdomain>/<mycontainer>/<myblob>`

Pour accéder à un formulaire web dans le conteneur `myforms` dans le sous-domaine personnalisé *photos.contoso.com*, vous pouvez par exemple utiliser l’URI suivant : `http://photos.contoso.com/myforms/applicationform.htm`

### <a name="remove-a-custom-domain-mapping"></a>Supprimer un mappage de domaine personnalisé

Pour supprimer un mappage de domaine personnalisé, annulez l’inscription du domaine personnalisé. Suivez l’une des procédures suivantes.

#### <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre compte de stockage.

2. Dans le volet du menu, sous **Sécurité + mise en réseau**, sélectionnez **Mise en réseau**.

3. Dans la page **Réseau**, choisissez l’onglet **Domaine personnalisé**.

4. Effacez le contenu de la zone de texte contenant le nom de votre domaine personnalisé.

5. Sélectionnez le bouton **Enregistrer**.

Une fois le domaine personnalisé supprimé, une notification de portail apparaît, indiquant que votre compte de stockage a été correctement mis à jour.

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Pour supprimer une inscription de domaine personnalisé, utilisez l’applet de commande PowerShell [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount), puis spécifiez une chaîne vide (`""`) pour la valeur d’argument `-CustomDomainName`.

- Format de commande :

  ```powershell
  Set-AzStorageAccount `
      -ResourceGroupName "<resource-group-name>" `
      -AccountName "<storage-account-name>" `
      -CustomDomainName ""
  ```

- Exemple de commande :

  ```powershell
  Set-AzStorageAccount `
      -ResourceGroupName "myresourcegroup" `
      -AccountName "mystorageaccount" `
      -CustomDomainName ""
  ```

#### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour supprimer une inscription de domaine personnalisé, utilisez la commande d’interface de ligne de commande [az storage account update](/cli/azure/storage/account) et spécifiez une chaîne vide (`""`) pour la valeur d’argument `--custom-domain`.

- Format de commande :

  ```azurecli
  az storage account update \
      --name <storage-account-name> \
      --resource-group <resource-group-name> \
      --custom-domain ""
  ```

- Exemple de commande :

  ```azurecli
  az storage account update \
      --name mystorageaccount \
      --resource-group myresourcegroup \
      --custom-domain ""
  ```

---

<a id="enable-https"></a>

## <a name="map-a-custom-domain-with-https-enabled"></a>Mapper un domaine personnalisé avec le protocole HTTPS activé

Cette approche implique davantage d’étapes, mais active l’accès HTTPS.

Si vous n’avez pas besoin que les utilisateurs accèdent à votre objet blob ou à votre contenu web en utilisant le protocole HTTPS, consultez la section [Mapper un domaine personnalisé avec uniquement le protocole HTTP activé](#enable-http) de cet article.

1. Activez [Azure CDN](../../cdn/cdn-overview.md) sur votre objet blob ou votre point de terminaison web.

   Pour un point de terminaison de Stockage Blob, voir [Intégrer un compte de stockage Azure avec Azure CDN](../../cdn/cdn-create-a-storage-account-with-cdn.md).

   Pour un point de terminaison de site web statique, voir [Intégrer un site web statique avec Azure CDN](static-website-content-delivery-network.md).

2. [Mapper du contenu CDN Azure avec un domaine personnalisé](../../cdn/cdn-map-content-to-custom-domain.md).

3. [Activer le protocole HTTPS sur un domaine personnalisé CDN Azure](../../cdn/cdn-custom-ssl.md).

   > [!NOTE]
   > Lorsque vous mettez à jour votre site web statique, veillez à effacer le contenu mis en cache sur les serveurs Edge du CDN en vidant le point de terminaison CDN. Pour plus d’informations, consultez [Purger un point de terminaison CDN Azure](../../cdn/cdn-purge-endpoint.md).

4. (Facultatif) Passez en revue les instructions suivantes :

   - [Jetons de signature d’accès partagé (SAP) avec Azure CDN](../../cdn/cdn-storage-custom-domain-https.md#shared-access-signatures).

   - [Redirection HTTP vers HTTPS avec Azure CDN](../../cdn/cdn-storage-custom-domain-https.md#http-to-https-redirection).

   - [Tarification et facturation lors de l’utilisation de Stockage Blob avec Azure CDN](../../cdn/cdn-storage-custom-domain-https.md#pricing-and-billing).

## <a name="feature-support"></a>Prise en charge des fonctionnalités

Ce tableau montre comment cette fonctionnalité est prise en charge dans votre compte ainsi que l’impact sur la prise en charge lorsque vous activez certaines fonctionnalités.

| Type de compte de stockage                | Stockage Blob (prise en charge par défaut)   | Data Lake Storage Gen2 <sup>1</sup>                        | NFS 3.0 <sup>1</sup>
|-----------------------------|---------------------------------|------------------------------------|--------------------------------------------------|
| Usage général v2 Standard | ![Oui](../media/icons/yes-icon.png) | ![Oui](../media/icons/yes-icon.png)  <sup>2</sup> | ![Oui](../media/icons/yes-icon.png)  <sup>2</sup> |
| Objets blob de blocs Premium          | ![Oui](../media/icons/yes-icon.png) | ![Oui](../media/icons/yes-icon.png)  <sup>2</sup> | ![Oui](../media/icons/yes-icon.png)  <sup>2</sup> |

<sup>1</sup>    Data Lake Storage Gen2 et le protocole NFS (Network File System) 3.0 requièrent tous deux un compte de stockage avec un espace de noms hiérarchique activé.

<sup>2</sup> La fonctionnalité est prise en charge dans la préversion.

## <a name="next-steps"></a>Étapes suivantes

- [Découvrir l’hébergement de site web statique dans Stockage Blob Azure](storage-blob-static-website.md)
