---
title: Personnaliser un modèle de marques avec l’API Azure Video Analyzer for Media (anciennement Video Indexer)
titleSuffix: Azure Video Analyzer for Media
description: Découvrez comment personnaliser un modèle de marques avec l’API Azure Video Analyzer for Media (anciennement Video Indexer).
services: azure-video-analyzer
author: anikaz
manager: johndeu
ms.topic: article
ms.subservice: azure-video-analyzer-media
ms.date: 01/14/2020
ms.author: kumud
ms.openlocfilehash: cf693036fb31613c171bf7618e45e3b1e57970e4
ms.sourcegitcommit: 0af634af87404d6970d82fcf1e75598c8da7a044
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/15/2021
ms.locfileid: "112123301"
---
# <a name="customize-a-brands-model-with-the-video-analyzer-for-media-api"></a>Personnalisation du modèle de marques à l’aide de l’API Video Analyzer for Media

Azure Video Analyzer for Media (anciennement Video Indexer) prend en charge la détection de marques dans les messages vocaux et visuels lors de l’indexation et de la réindexation de contenu vidéo et audio. La fonctionnalité de détection de marques identifie les noms de produits, services et entreprises suggérés par la base de données de marques Bing. Par exemple, si le nom de Microsoft est mentionné dans du contenu vidéo ou audio, ou s’il apparaît sous forme de texte visuel dans une vidéo, Video Analyzer for Media le détecte et l’interprète comme un nom de marque dans le contenu. Un modèle de marques personnalisé permet d’exclure certaines marques de la détection et d’inclure les marques devant faire partie de votre modèle qui peuvent ne pas se trouver dans la base de données de marques de Bing. Pour plus d’informations, consultez [What is Content Moderator?](customize-brands-model-overview.md) (Présentation de Content Moderator).

> [!NOTE]
> Si votre vidéo a été indexée avant l’ajout d’une marque, vous devez la réindexer.

Vous pouvez utiliser l’API Video Analyzer for Media pour créer, utiliser et modifier des modèles personnalisés de marques détectées dans une vidéo, comme décrit dans cette rubrique. Vous pouvez également utiliser le site web Video Analyzer for Media comme décrit dans [Personnaliser un modèle de marques avec le site web Video Analyzer for Media](customize-brands-model-with-api.md).

## <a name="create-a-brand"></a>Créer une marque

Le [créer une marque](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Create-Brand) API crée une marque personnalisée et l’ajoute au modèle de marques personnalisées pour le compte spécifié.

> [!NOTE]
> L’affectation de la valeur true à `enabled` (dans le corps) a pour effet de placer la marque dans la liste *Include* pour permettre à Video Analyzer for Media de la détecter. L’affectation de la valeur false à `enabled` a pour effet de placer la marque dans la liste *Exclude* pour empêcher Video Analyzer for Media de la détecter.

D’autres paramètres que vous pouvez définir dans le corps :

* La valeur `referenceUrl` peut correspondre à n’importe quel site web de référence de la marque, comme un lien vers sa page Wikipédia.
* La valeur `tags` est une liste d’étiquettes pour la marque. Elle apparaît dans le champ *Catégorie* de la marque sur le site web Video Analyzer for Media. Par exemple, la marque « Azure » peut être étiquetée ou classée dans la catégorie « Cloud ».

### <a name="response"></a>response

La réponse fournit des informations sur la marque que vous venez de créer au format de l’exemple ci-dessous.

```json
{
  "referenceUrl": "https://en.wikipedia.org/wiki/Example",
  "id": 97974,
  "name": "Example",
  "accountId": "SampleAccountId",
  "lastModifierUserName": "SampleUserName",
  "created": "2018-04-25T14:59:52.7433333",
  "lastModified": "2018-04-25T14:59:52.7433333",
  "enabled": true,
  "description": "This is an example",
  "tags": [
    "Tag1",
    "Tag2"
  ]
}
```

## <a name="delete-a-brand"></a>Supprimer une marque

L’ [supprimer une marque](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Delete-Brand) API supprime une marque du modèle de marques personnalisées pour le compte spécifié. Le compte est spécifié dans le paramètre `accountId`. Une fois le paramètre appelé, la marque disparaît des listes *Include* ou *Exclude*.

### <a name="response"></a>response

Aucun contenu n’est retourné en cas de suppression effective de la marque.

## <a name="get-a-specific-brand"></a>Obtenir une marque spécifique

Les[obtiennent une marque](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Brand) API vous permet de rechercher les détails d’une marque dans le modèle de marques personnalisées pour le compte spécifié à l’aide de l’identifiant de marque.

### <a name="response"></a>response

La réponse fournit des informations sur la marque que vous avez recherchée (à l’aide de l’ID de marque) au format de l’exemple ci-dessous.

```json
{
  "referenceUrl": "https://en.wikipedia.org/wiki/Example",
  "id": 128846,
  "name": "Example",
  "accountId": "SampleAccountId",
  "lastModifierUserName": "SampleUserName",
  "created": "2018-01-06T13:51:38.3666667",
  "lastModified": "2018-01-11T13:51:38.3666667",
  "enabled": true,
  "description": "This is an example",
  "tags": [
    "Tag1",
    "Tag2"
  ]
}
```

> [!NOTE]
> Si `enabled` est défini sur `true`, cela signifie que la marque figure dans la liste *Include* de sorte que Video Analyzer for Media la détecte, et si `enabled` est défini sur *, cela signifie que la marque figure dans la liste* Exclude de sorte que Video Analyzer for Media ne la détecte pas.

## <a name="update-a-specific-brand"></a>Mettre à jour une marque spécifique

Les[mettent à jour une marque](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Update-Brand) API vous permet de rechercher les détails d’une marque dans le modèle de marques personnalisées pour le compte spécifié à l’aide de l’identifiant de marque.

### <a name="response"></a>response

La réponse fournit les informations actualisées sur la marque mise à jour au format de l’exemple ci-dessous.

```json
{
  "referenceUrl": null,
  "id": 97974,
  "name": "Example",
  "accountId": "SampleAccountId",
  "lastModifierUserName": "SampleUserName",
  "Created": "2018-04-25T14:59:52.7433333",
  "lastModified": "2018-04-25T15:37:50.67",
  "enabled": false,
  "description": "This is an update example",
  "tags": [
    "Tag1",
    "NewTag2"
  ]
}
```

## <a name="get-all-of-the-brands"></a>Obtenir toutes les marques

Les [obtiennent toutes les marques](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Brands) API retourne toutes les marques dans le modèle de marques personnalisées pour le compte spécifié, que la marque soit ou non dans la liste de marques à *inclure* ou à *exclure*.

### <a name="response"></a>response

La réponse renvoie une liste de toutes les marques de votre compte et les détails qui les concernent, au format de l’exemple ci-dessous.

```json
[
    {
        "ReferenceUrl": null,
        "id": 97974,
        "name": "Example",
        "accountId": "AccountId",
        "lastModifierUserName": "UserName",
        "Created": "2018-04-25T14:59:52.7433333",
        "LastModified": "2018-04-25T14:59:52.7433333",
        "enabled": true,
        "description": "This is an example",
        "tags": ["Tag1", "Tag2"]
    },
    {
        "ReferenceUrl": null,
        "id": 97975,
        "name": "Example2",
        "accountId": "AccountId",
        "lastModifierUserName": "UserName",
        "Created": "2018-04-26T14:59:52.7433333",
        "LastModified": "2018-04-26T14:59:52.7433333",
        "enabled": false,
        "description": "This is another example",
        "tags": ["Tag1", "Tag2"]
    },
]
```

> [!NOTE]
> La marque *Example* figure dans la liste *Include* pour que Video Analyzer for Media puisse la détecter, et la marque *Example2* figure dans la liste *Exclude* pour que Video Analyzer for Media ne puisse pas la détecter.

## <a name="get-brands-model-settings"></a>Obtenir les paramètres du modèle de marques

Les [ obtiennent des paramètres de marques](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Get-Brands) API retourne les paramètres du modèle de marques dans le compte spécifié. Les paramètres du modèle de marques indiquent si la détection depuis la base de données de marques de Bing est activée ou non. Si les marques de Bing ne sont pas activées, Video Analyzer for Media détecte uniquement les marques définies dans le modèle de marques personnalisé du compte spécifié.

### <a name="response"></a>response

La réponse indique si les marques de Bing sont activées au format de l’exemple ci-dessous.

```json
{
  "state": true,
  "useBuiltIn": true
}
```

> [!NOTE]
> Si `useBuiltIn` est défini sur true, les marques de Bing sont activées. Si `useBuiltin` est défini sur false, les marques de Bing sont désactivées. Il se peut que la valeur `state` soit ignorée parce qu’elle est déconseillée.

## <a name="update-brands-model-settings"></a>Mettre à jour les paramètres du modèle de marques

L’API [mises à jour de marques](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Update-Brands-Model-Settings) met à jour les paramètres du modèle de marques dans le compte spécifié. Les paramètres du modèle de marques indiquent si la détection depuis la base de données de marques de Bing est activée ou non. Si les marques de Bing ne sont pas activées, Video Analyzer for Media détecte uniquement les marques définies dans le modèle de marques personnalisé du compte spécifié.

L’indicateur `useBuiltIn` défini sur true indique que les marques Bing sont activées. Si `useBuiltin` est défini sur false, les marques Bing sont désactivées.

### <a name="response"></a>response

Aucun contenu n’est retourné en cas de mise à jour effective des paramètres du modèle de marques.

## <a name="next-steps"></a>Étapes suivantes

[Personnaliser le modèle de marques à l’aide du site web](customize-brands-model-with-website.md)
