---
title: Copier les fichiers nouveaux et modifiés par LastModifiedDate
description: Découvrez comment utiliser un modèle de solution pour copier les fichiers nouveaux et modifiés par LastModifiedDate avec Azure Data Factory.
author: dearandyxu
ms.author: yexu
ms.reviewer: ''
ms.service: data-factory
ms.subservice: tutorials
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 3/8/2019
ms.openlocfilehash: 601dcaa3ea402f8f6b8b8c0664b8a47cfb5f4359
ms.sourcegitcommit: 0770a7d91278043a83ccc597af25934854605e8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/13/2021
ms.locfileid: "124743522"
---
# <a name="copy-new-and-changed-files-by-lastmodifieddate-with-azure-data-factory"></a>Copier les fichiers nouveaux et modifiés par LastModifiedDate avec Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article décrit un modèle de solution permettant de copier les fichiers nouveaux et modifiés par LastModifiedDate entre un magasin basé sur des fichiers et un magasin de destination. 

## <a name="about-this-solution-template"></a>À propos de ce modèle de solution

Ce modèle commence par sélectionner les fichiers nouveaux et modifiés en fonction de leur attribut **LastModifiedDate**, puis copie les fichiers sélectionnés entre le magasin de source de données et le magasin de destination des données.

Le modèle contient une activité :
- **Copier** pour copier les fichiers nouveaux et modifiés par LastModifiedDate entre un magasin de fichiers et un magasin de destination.

Le modèle définit six paramètres :
-  *FolderPath_Source* correspond au chemin d’accès du dossier où vous pouvez lire les fichiers à partir du magasin source. Vous devez remplacer la valeur par défaut par votre propre chemin d’accès au dossier.
-  *Directory_Source* correspond au chemin du sous-dossier où vous pouvez lire les fichiers du magasin source. Vous devez remplacer la valeur par défaut par votre propre chemin de sous-dossier.
-  *FolderPath_Destination* correspond au chemin d’accès du dossier où vous souhaitez copier les fichiers vers le magasin de destination. Vous devez remplacer la valeur par défaut par votre propre chemin d’accès au dossier.
-  *Directory_Destination* correspond au chemin du dossier où vous souhaitez copier les fichiers vers le magasin de destination. Vous devez remplacer la valeur par défaut par votre propre chemin de sous-dossier.
-  *LastModified_From* est utilisé pour sélectionner les fichiers dont l’attribut LastModifiedDate est postérieur ou égal à cette valeur DateHeure.  Pour sélectionner uniquement les nouveaux fichiers (non copiés précédemment), définissez cette valeur DateHeure sur l'heure du dernier déclenchement du pipeline. Vous pouvez remplacer la valeur par défaut « 2019-02-01T00:00:00Z » par la valeur LastModifiedDate attendue dans le fuseau horaire UTC. 
-  *LastModified_To* est utilisé pour sélectionner les fichiers dont l’attribut LastModifiedDate est antérieur à cette valeur DateHeure. Pour sélectionner uniquement les nouveaux fichiers (non copiés précédemment), cette valeur DateHeure peut correspondre à l'heure actuelle.  Vous pouvez remplacer la valeur par défaut « 2019-02-01T00:00:00Z » par la valeur LastModifiedDate attendue dans le fuseau horaire UTC. 

## <a name="how-to-use-this-solution-template"></a>Utiliser ce modèle de solution

1. Accédez au modèle **Copier les nouveaux fichiers par LastModifiedDate uniquement**. Créez une **nouvelle** connexion à votre magasin de stockage source. C’est à partir de ce magasin que seront copiés les fichiers de votre choix.

    :::image type="content" source="media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate1.png" alt-text="Créer une connexion à la source":::
    
2. Créez une **nouvelle connexion** au magasin de destination. C’est à partir de ce magasin que seront copiés les fichiers de votre choix. 

    :::image type="content" source="media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate3.png" alt-text="Créer une connexion à la destination":::

3. Sélectionnez **Utiliser ce modèle**.

    :::image type="content" source="media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate4.png" alt-text="Utiliser ce modèle":::
    
4. Le pipeline apparaît comme disponible dans le panneau, comme le montre l’exemple suivant :

    :::image type="content" source="media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate5.png" alt-text="Afficher le pipeline":::

5. Sélectionnez **Déboguer**, entrez la valeur des **Paramètres** et sélectionnez **Terminer**.  Comme illustré ci-après, nous définissons les paramètres comme suit.
   - **FolderPath_Source** = sourcefolder
   - **Directory_Source** = subfolder
   - **FolderPath_Destination** = destinationfolder
   - **Directory_Destination** = subfolder
   - **LastModified_From** =  2019-02-01T00:00:00Z
   - **LastModified_To** = 2019-03-01T00:00:00Z
    
    L’exemple indique que les fichiers qui ont été modifiés au cours de la période (**2019-02-01T00:00:00Z** à **2019-03-01T00:00:00Z**) seront copiés à partir du chemin source **dossier_source/sous_dossier** vers le chemin de destination **dossier_destination/sous_dossier**.  Vous pouvez remplacer ces valeurs par vos propres paramètres.

    :::image type="content" source="media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate6.png" alt-text="Exécuter le pipeline":::

6. Vérifiez le résultat. Seuls les derniers fichiers modifiés dans l'intervalle de temps configuré ont été copiés dans le magasin de destination.

    :::image type="content" source="media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate7.png" alt-text="Vérifier le résultat":::
    
7. Vous pouvez maintenant ajouter un déclencheur de fenêtres bascules pour automatiser ce pipeline et lui permettre de copier systématiquement les fichiers nouveaux et modifiés par LastModifiedDate et ce, de manière régulière.  Sélectionnez **Ajouter un déclencheur**, puis **Nouveau/Modifier**.

    :::image type="content" source="media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate8.png" alt-text="Capture d’écran mettant en surbrillance l’option de menu Nouveau/Modifier qui s’affiche quand vous sélectionnez Ajouter un déclencheur.":::
    
8. Dans la fenêtre **Ajouter des déclencheurs**, sélectionnez **+ Nouveau**.

9. Sélectionnez **Fenêtre bascule** pour le type de déclencheur, définissez **Toutes les 15 minutes** en tant que périodicité (vous pouvez opter pour l’intervalle de temps de votre choix). Sélectionnez **Oui** pour la zone Activé, puis sélectionnez **OK**.

    :::image type="content" source="media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate10.png" alt-text="Créer le déclencheur"::: 
    
10. Définissez la valeur correspondant aux **Paramètres d’exécution du déclencheur** comme suit, puis sélectionnez **Terminer**.
    - **FolderPath_Source** = **sourcefolder**.  Vous pouvez remplacer ce dossier par votre dossier dans le magasin de données sources.
    - **Directory_Source** = **subfolder**.  Vous pouvez remplacer ce dossier par votre sous-dossier dans le magasin de données source.
    - **FolderPath_Destination** = **destinationfolder**.  Vous pouvez remplacer ce dossier par votre dossier dans le magasin de données de destination.
    - **Directory_Destination** = **subfolder**.  Vous pouvez remplacer ce dossier par votre sous-dossier dans le magasin de données de destination.
    - **LastModified_From** =   **\@trigger().outputs.windowStartTime**.  Cette variable système issue du déclencheur détermine l'heure du dernier déclenchement du pipeline.
    - **LastModified_To** =  **\@trigger().outputs.windowEndTime**.  Cette variable système issue du déclencheur détermine l'heure du déclenchement du pipeline actuel.
    
    :::image type="content" source="media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate11.png" alt-text="Paramètres d'entrée":::
    
11. Sélectionnez **Publier tout**.
    
    :::image type="content" source="media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate12.png" alt-text="Publier tout":::

12. Créer des fichiers dans le dossier source du magasin source de données.  Vous attendez à présent que le pipeline se déclenche automatiquement, et seuls les nouveaux fichiers seront copiés dans le magasin de destination.

13. Sélectionnez l’onglet **Surveiller** dans le volet de navigation gauche, puis patientez environ 15 minutes si la périodicité du déclencheur a été définie sur Toutes les 15 minutes. 

14. Vérifiez le résultat. Votre pipeline sera déclenché automatiquement toutes les 15 minutes, et seuls les fichiers nouveaux et modifiés à partir du magasin source seront copiés vers le magasin de destination à chaque exécution du pipeline.

    :::image type="content" source="media/solution-template-copy-new-files-lastmodifieddate/copy-new-files-lastmodifieddate15.png" alt-text="Capture d’écran montrant les résultats retournés lors du déclenchement du pipeline.":::
    
## <a name="next-steps"></a>Étapes suivantes

- [Présentation du service Azure Data Factory](introduction.md)
