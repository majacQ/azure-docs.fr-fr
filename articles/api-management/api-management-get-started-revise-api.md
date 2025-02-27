---
title: 'Tutoriel : Utiliser des révisions dans Gestion des API pour apporter aux API des changements non cassants de manière sécurisée'
titleSuffix: Azure API Management
description: Suivez les étapes de ce didacticiel pour apprendre à apporter des modifications sans rupture à l’aide de révisions dans Gestion des API.
services: api-management
documentationcenter: ''
author: dlepow
ms.service: api-management
ms.custom: mvc, devx-track-azurecli
ms.topic: tutorial
ms.date: 02/09/2021
ms.author: danlep
ms.openlocfilehash: 34ffc28e6c8e109252b68be269c1918beabfd5af
ms.sourcegitcommit: f6e2ea5571e35b9ed3a79a22485eba4d20ae36cc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/24/2021
ms.locfileid: "128676596"
---
# <a name="tutorial-use-revisions-to-make-non-breaking-api-changes-safely"></a>Tutoriel : Utiliser des révisions pour apporter aux API des changements non cassants de manière sécurisée
Une fois que votre API est prête et commence à être utilisée par les développeurs, vous devez finir par lui apporter des modifications sans interrompre les appelants de l’API. Il est également utile d’informer les développeurs des modifications apportées. 

Dans Gestion des API Azure, utilisez des *révisions* pour apporter aux API des changements non cassants afin de pouvoir modéliser et tester ces derniers de manière sécurisée. Quand vous êtes prêt, vous pouvez rendre actuelle une révision et remplacer votre API actuelle. 

Pour plus d’informations générales, consultez [Versions et révisions](https://azure.microsoft.com/blog/versions-revisions/) et [Gestion des versions d’API avec la Gestion des API Azure](https://azure.microsoft.com/blog/api-versioning-with-azure-api-management/).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Ajouter une révision
> * Apporter des modifications sans rupture à une révision
> * Rendre cette révision actuelle et ajouter une entrée au journal des modifications
> * Parcourir le portail des développeurs pour voir les modifications et le journal des modifications

:::image type="content" source="media/api-management-getstarted-revise-api/azure-portal.png" alt-text="Révisions d’API dans le portail Azure":::

## <a name="prerequisites"></a>Prérequis

+ Apprenez la [terminologie relative à Gestion des API Azure](api-management-terminology.md).
+ Suivez ce guide de démarrage rapide : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md).
+ Suivez également le didacticiel suivant : [Importer et publier votre première API](import-and-publish.md).

## <a name="add-a-new-revision"></a>Ajouter une révision

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre instance de Gestion des API.
1. Sélectionnez **API**.
2. Dans la liste des API, sélectionnez **API de conférence de démonstration** (ou une autre API à laquelle vous souhaitez ajouter des révisions).
3. Sélectionnez l’onglet **Révisions**.
4. Sélectionnez **+ Ajouter une révision**.

   :::image type="content" source="media/api-management-getstarted-revise-api/07-add-revisions-01-add-new-revision.png" alt-text="Ajouter une révision de l’API":::

    > [!TIP]
    > Vous pouvez également sélectionner **Ajouter une révision** dans le menu contextuel ( **...** ) de l’API.

5. Indiquez une description pour la nouvelle révision qui vous aidera à vous souvenir à quoi elle sert.
6. Sélectionnez **Créer**.
7. Vous venez de créer une nouvelle révision.

    > [!NOTE]
    > Votre API d’origine reste dans **Revision 1**. Il s’agit de la révision que vos utilisateurs continuent à appeler, jusqu’à ce que vous choisissiez d’actualiser une autre révision.

## <a name="make-non-breaking-changes-to-your-revision"></a>Apporter des modifications sans rupture à une révision

1. Dans la liste des API, sélectionnez **API de conférence de démonstration**.
1. Sélectionnez l’onglet **Conception** situé près du haut de l’écran.
1. Notez que dans le **sélecteur de révision** (juste au-dessus de l’onglet Conception), **Revision 2** est sélectionné.

    > [!TIP]
    > Utilisez le sélecteur de révision pour basculer entre les révisions sur lesquelles vous souhaitez travailler.
1. Sélectionnez **+ Ajouter une opération**.
1. Définissez votre opération sur **POST**, puis attribuez au nom, au nom d’affichage et à l’URL de l’opération la valeur **test**.
1. **Enregistrez** votre nouvelle opération.

   :::image type="content" source="media/api-management-getstarted-revise-api/07-add-revisions-02-make-changes.png" alt-text="Modifier une révision":::
1. Vous avez modifié **Revision 2**. Utilisez le **sélecteur de révision** situé près du haut de la page pour revenir à **Revision 1**.
1. Notez que votre nouvelle opération n’apparaît pas dans **Revision 1**. 

## <a name="make-your-revision-current-and-add-a-change-log-entry"></a>Rendre cette révision actuelle et ajouter une entrée au journal des modifications

### <a name="portal"></a>[Portail](#tab/azure-portal)

1. Sélectionnez l’onglet **Révisions** dans le menu situé en haut de la page.
1. Ouvrez le menu contextuel (**...**) pour **Revision 2**.
1. Sélectionnez **Rendre actuel**.
1. Cochez la case **Publier sur le journal des modifications public de cette API**, si vous souhaitez publier les remarques relatives à cette modification. Fournissez une description de votre modification destinée aux développeurs, par exemple : **Test des révisions. Nouvelle opération « test » ajoutée.**
1. **Revision 2** est maintenant la révision actuelle.

    :::image type="content" source="media/api-management-getstarted-revise-api/revisions-menu.png" alt-text="Menu de révision dans la fenêtre Révisions":::

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour commencer à utiliser Azure CLI :

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Utilisez cette procédure pour créer et mettre à jour une version.

1. Exécutez la commande [az apim api list](/cli/azure/apim/api#az_apim_api_list) pour voir vos ID d’API :

   ```azurecli
   az apim api list --resource-group apim-hello-word-resource-group \
       --service-name apim-hello-world --output table
   ```

   L’ID d’API à utiliser dans la commande suivante est la valeur `Name`. La révision de l’API se trouve dans la colonne `ApiRevision`.

1. Pour créer la version, avec une note de publication, exécutez la commande [az apim api release create](/cli/azure/apim/api/release#az_apim_api_release_create) :

   ```azurecli
   az apim api release create --resource-group apim-hello-word-resource-group \
       --api-id demo-conference-api --api-revision 2 --service-name apim-hello-world \
       --notes 'Testing revisions. Added new "test" operation.'
   ```

   La révision que vous publiez devient la révision actuelle.

1. Pour voir vos versions, utilisez la commande [az apim api release list](/cli/azure/apim/api/release#az_apim_api_release_list) :

   ```azurecli
   az apim api release list --resource-group apim-hello-word-resource-group \
       --api-id echo-api --service-name apim-hello-world --output table
   ```

   Les notes que vous spécifiez s’affichent dans le journal des modifications. Vous pouvez les voir dans la sortie de la commande précédente.

1. Lorsque vous créez une version, le paramètre `--notes` est facultatif. Vous pouvez ajouter ou modifier ultérieurement les notes à l’aide de la commande [az apim api release update](/cli/azure/apim/api/release#az_apim_api_release_update) :

   ```azurecli
   az apim api release update --resource-group apim-hello-word-resource-group \
       --api-id demo-conference-api --release-id 00000000000000000000000000000000 \
       --service-name apim-hello-world --notes "Revised notes."
   ```

   Utilisez la valeur de la colonne `Name` pour l’ID de version.

Vous pouvez supprimer n’importe quelle version en exécutant la commande [az apim api release delete](/cli/azure/apim/api/release#az_apim_api_release_delete) :

```azurecli
az apim api release delete --resource-group apim-hello-word-resource-group \
    --api-id demo-conference-api --release-id 00000000000000000000000000000000 
    --service-name apim-hello-world
```

---

## <a name="browse-the-developer-portal-to-see-changes-and-change-log"></a>Parcourir le portail des développeurs pour voir les modifications et le journal des modifications

Si vous avez essayé le [portail des développeurs](api-management-howto-developer-portal-customize.md), vous pouvez passer en revue les modifications apportées à l’API et y changer le journal.

1. Dans le Portail Azure, sélectionnez **API**.
1. Sélectionnez **Portail des développeurs** dans le menu supérieur.
1. Dans le portail des développeurs, sélectionnez **API**, puis **API de conférence de démonstration**.
1. Notez que votre nouvelle opération **test** est désormais disponible.
1. Sélectionnez **Journal des modifications** en regard du nom de l’API.
1. Notez que l’entrée du journal des modifications apparaît dans cette liste.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Ajouter une révision
> * Apporter des modifications sans rupture à une révision
> * Rendre cette révision actuelle et ajouter une entrée au journal des modifications
> * Parcourir le portail des développeurs pour voir les modifications et le journal des modifications

Passez au tutoriel suivant :

> [!div class="nextstepaction"]
> [Publier plusieurs versions de votre API](api-management-get-started-publish-versions.md)
