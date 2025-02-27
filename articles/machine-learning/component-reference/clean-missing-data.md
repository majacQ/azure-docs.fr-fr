---
title: 'Nettoyer des données manquantes : référence du composant'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le composant Nettoyer des données manquantes dans Azure Machine Learning pour supprimer, remplacer ou inférer des valeurs manquantes.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/11/2020
ms.openlocfilehash: 7e75506d68f35e7b3a29e36af97d01e1174723c3
ms.sourcegitcommit: e41827d894a4aa12cbff62c51393dfc236297e10
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2021
ms.locfileid: "131566876"
---
# <a name="clean-missing-data-component"></a>Composant Nettoyer des données manquantes

Cet article décrit un composant dans le concepteur Azure Machine Learning.

Utilisez ce composant pour supprimer, remplacer ou inférer des valeurs manquantes. 

Les scientifiques des données vérifient souvent les données pour détecter les valeurs manquantes et effectuer ensuite diverses opérations pour corriger les données ou en ajouter de nouvelles. L’objectif de ces opérations de nettoyage est d’éviter les problèmes causés par des données manquantes pouvant survenir lors de l’apprentissage d’un modèle. 

Ce composant prend en charge plusieurs types d’opérations pour le « nettoyage » de valeurs manquantes, à savoir :

+ Le remplacement des valeurs manquantes par un espace réservé, une moyenne ou une autre valeur
+ La suppression totale des lignes et des colonnes ayant des valeurs manquantes
+ La déduction de valeurs basées sur des méthodes statistiques


L’utilisation de ce composant ne modifie pas votre jeu de données source. Au lieu de cela, il crée un nouveau jeu de données dans votre espace de travail, que vous pouvez utiliser dans le flux de travail suivant. Vous pouvez également enregistrer le nouveau jeu de données nettoyé pour une nouvelle utilisation.

Ce composant génère également une définition de la transformation utilisée pour nettoyer les valeurs manquantes. Vous pouvez réutiliser cette transformation sur d’autres jeux de données qui ont le même schéma, à l’aide du composant [Appliquer une transformation](./apply-transformation.md).  

## <a name="how-to-use-clean-missing-data"></a>Comment utiliser le nettoyage des données manquantes

Ce composant vous permet de définir une opération de nettoyage. Vous pouvez également enregistrer l’opération de nettoyage afin de pouvoir l’appliquer ultérieurement à de nouvelles données. Consultez les sections suivantes pour savoir comment créer et enregistrer un processus de nettoyage : 
 
+ [Pour remplacer des valeurs manquantes](#replace-missing-values)
  
+ [Pour appliquer une transformation de nettoyage à de nouvelles données](#apply-a-saved-cleaning-operation-to-new-data)
 
> [!IMPORTANT]
> La méthode de nettoyage que vous utilisez pour gérer des valeurs manquantes peut affecter considérablement vos résultats. Nous vous recommandons de tester plusieurs méthodes. Prenez en compte la justification d’une utilisation d’une méthode particulière et la qualité des résultats.

### <a name="replace-missing-values"></a>Remplacer des valeurs manquantes  

Chaque fois que vous appliquez le composant [Nettoyer des données manquantes](./clean-missing-data.md) à un jeu de données, la même opération de nettoyage est appliquée à toutes les colonnes que vous sélectionnez. Par conséquent, si vous avez besoin de nettoyer des colonnes différentes à l’aide de différentes méthodes, utilisez des instances distinctes du composant.

1.  Ajoutez le composant [Nettoyer des données manquantes](./clean-missing-data.md) à votre pipeline et connectez le jeu de données auquel manquent des valeurs.  
  
2.  Pour **Colonnes à nettoyer**, choisissez les colonnes qui contiennent des valeurs manquantes et que vous souhaitez modifier. Vous pouvez choisir plusieurs colonnes, mais vous devez utiliser la même méthode de remplacement dans toutes les colonnes sélectionnées. Par conséquent, en général, vous devez nettoyer les colonnes de type chaîne et les colonnes numériques séparément.

    Par exemple, pour rechercher les valeurs manquantes dans toutes les colonnes numériques :

    1. Sélectionnez le composant **Nettoyer des données manquantes**, puis cliquez sur **Modifier la colonne** dans le volet droit du composant.

    3. Pour **Inclure**, sélectionnez **Types de colonnes** dans la liste déroulante, puis sélectionnez **Numérique**. 
  
    N’importe quelle méthode de nettoyage ou de remplacement que vous choisissez doit être applicable à **toutes** les colonnes sélectionnées. Si les données dans n’importe quelle colonne sont incompatibles avec l’opération spécifiée, le composant renvoie une erreur et arrête le pipeline.
  
3.  Pour **rapport minimal des valeurs manquantes**, spécifiez le nombre minimal de valeurs manquantes requis pour l’opération à effectuer.  
  
    Vous utilisez cette option conjointement avec le **rapport maximal des valeurs manquantes** pour définir les conditions sous lesquelles une opération de nettoyage est effectuée sur le jeu de données. S’il existe trop ou trop peu de lignes qui sont des valeurs manquantes, l’opération ne peut pas être effectuée. 
  
    Le nombre que vous entrez représente le **rapport** des valeurs manquantes pour toutes les valeurs dans la colonne. Par défaut, la propriété **rapport minimal des valeurs manquantes** est définie sur 0. Cela signifie que les valeurs manquantes sont nettoyées même s’il n’y a qu’une seule valeur manquante. 

    > [!WARNING]
    > Cette condition doit être remplie par chaque colonne afin que l’opération spécifiée soit appliquée. Par exemple, supposons que vous avez sélectionné trois colonnes et défini le rapport minimal des valeurs manquantes sur 0,2 (20 %), mais qu’une seule colonne possède 20 % de valeurs manquantes. Dans ce cas, l’opération de nettoyage s’appliquera uniquement à la colonne ayant plus de 20 % de valeurs manquantes. Par conséquent, les autres colonnes resteront inchangées.
    > 
    > Si vous avez un doute à propos de valeurs manquantes qui auraient été modifiées ou non, sélectionnez l'option **Générer la colonne d'indicateur des valeurs manquantes**. Une colonne est ajoutée au jeu de données pour indiquer si chaque colonne répond ou non aux critères spécifiés pour les plages minimales et maximales.  
  
4. Pour **rapport maximal des valeurs manquantes**, spécifiez le nombre maximal de valeurs manquantes requis pour l’opération à effectuer.   
  
    Par exemple, vous souhaitez peut-être effectuer la substitution des valeurs manquantes uniquement si 30 % ou moins des lignes contiennent des valeurs manquantes, tout en souhaitant laisser les valeurs si plus de 30 % des lignes ont des valeurs manquantes.  
  
    Vous définissez le nombre comme le rapport des valeurs manquantes à toutes les valeurs dans la colonne. Par défaut, le **rapport maximal des valeurs manquantes** est défini sur 1. Cela signifie que les valeurs manquantes sont nettoyées même si 100 % des valeurs dans la colonne sont manquantes.  
  
   
  
5. Comme **Mode de nettoyage**, sélectionnez l’une des options suivantes pour le remplacement ou la suppression des valeurs manquantes :  
  
  
    + **Valeur de substitution personnalisée** : utilisez cette option pour spécifier une valeur d’espace réservé (par exemple, la valeur 0 ou N/D) qui s’applique à toutes les valeurs manquantes. La valeur que vous spécifiez comme remplacement doit être compatible avec le type de données de la colonne.
  
    + **Remplacer par moyenne** : calcule la moyenne de la colonne et l’utilise comme valeur de remplacement pour chaque valeur manquante dans la colonne.  
  
        S’applique uniquement aux colonnes ayant des types de données Entier, Double ou Booléen.  
  
    + **Remplacer par médiane** : calcule la valeur médiane de la colonne et l’utilise comme remplacement de toute valeur manquante dans la colonne.  
  
        S’applique uniquement aux colonnes ayant des types de données Entier ou Double. 
  
    + **Remplacer par mode** : calcule le mode de la colonne et l’utilise comme valeur de remplacement de toute valeur manquante dans la colonne.  
  
        S’applique aux colonnes ayant des types de données Entier, Double, Booléen ou Catégorique. 
  
    + **Supprimer une ligne entière** : supprime complètement n’importe quelle ligne du jeu de données comportant une ou plusieurs valeurs manquantes. Cela est utile si la valeur manquante peut être considérée comme manquante de façon aléatoire.  
  
    + **Supprimer la colonne entière** : supprime complètement toute colonne dans le jeu de données avec une ou plusieurs valeurs manquantes.  
  
    
  
6. L’option **Valeur de remplacement** est disponible si vous avez sélectionné l’option **Valeur de substitution personnalisée**. Tapez une nouvelle valeur à utiliser comme valeur de remplacement pour toutes les valeurs manquantes dans la colonne.  
  
    Notez que vous pouvez utiliser cette option uniquement dans les colonnes ayant des types de données Entier, Double, Booléen ou Chaîne.
  
7. **Générer une colonne d’indicateur de valeurs manquantes** : sélectionnez cette option si vous voulez indiquer si les valeurs de la colonne répondent ou non aux critères de nettoyage des valeurs manquantes. Cette option est particulièrement utile lorsque vous mettez en place une nouvelle opération de nettoyage et que vous souhaitez vous assurer qu’il fonctionne comme prévu.
  
8. Envoyez le pipeline.

### <a name="results"></a>Résultats

Le composant retourne deux sorties :  

-   **Jeu de données nettoyées** : un jeu de données composé des colonnes sélectionnées, avec des valeurs manquantes gérées comme spécifié, ainsi qu’une colonne d’indicateur, si vous avez sélectionné cette option.  

    Les colonnes non sélectionnées pour le nettoyage sont également « transmises ».  
  
-  **Transformation du nettoyage** : une transformation de données utilisée pour le nettoyage, qui peut être enregistrée dans votre espace de travail et appliquée ultérieurement à de nouvelles données.

### <a name="apply-a-saved-cleaning-operation-to-new-data"></a>Appliquer une opération de nettoyage enregistrée à de nouvelles données  

Si vous devez répéter les opérations de nettoyage souvent, nous vous recommandons d’enregistrer votre recette pour le nettoyage des données comme une *transformation*, pour le réutiliser avec le même jeu de données. L’enregistrement d’une transformation de nettoyage est particulièrement utile si vous devez réimporter et nettoyer fréquemment des données ayant le même schéma.  
      
1.  Ajoutez le composant [Appliquer une transformation](./apply-transformation.md) à votre pipeline.  
  
2.  Ajoutez le jeu de données que vous souhaitez nettoyer et connectez le jeu de données au port d’entrée droit.  
  
3.  Développez le groupe **Transformations** dans le volet gauche du concepteur. Recherchez la transformation enregistrée et faites-la glisser vers le pipeline.  

4.  Connectez la transformation enregistrée au port d’entrée gauche de [Appliquer une transformation](./apply-transformation.md). 

    Lorsque vous appliquez une transformation enregistrée, vous ne pouvez pas sélectionner les colonnes auxquelles elle est appliquée. C’est parce que la transformation a déjà été définie et s’applique automatiquement aux colonnes spécifiées dans l’opération d’origine.

    Toutefois, supposons que vous avez créé une transformation sur un sous-ensemble de colonnes numériques. Vous pouvez appliquer cette transformation pour un jeu de données des types de colonne mixtes sans déclencher d’erreur, car les valeurs manquantes sont modifiés uniquement dans les colonnes numériques correspondantes.

6.  Envoyez le pipeline.  

## <a name="next-steps"></a>Étapes suivantes

Consultez les [composants disponibles](component-reference.md) pour Azure Machine Learning. 