---
title: Déclarer des ressources dans des modèles
description: Explique comment déclarer des ressources à déployer dans un modèle Azure Resource Manager (modèle ARM).
ms.topic: conceptual
ms.date: 05/11/2021
ms.openlocfilehash: 7d7bae8adec81aa3344c5c571f0e40556928a1a9
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/10/2021
ms.locfileid: "111960127"
---
# <a name="resource-declaration-in-arm-templates"></a>Déclaration de ressource dans des modèles ARM

Pour déployer une ressource via un modèle Azure Resource Manager (modèle ARM), vous ajoutez une déclaration de ressource. Utilisez le tableau `resources` dans un modèle JSON.

## <a name="set-resource-type-and-version"></a>Définir le type et la version d’une ressource

Lorsque vous ajoutez une ressource à votre modèle, commencez par définir le type de la ressource et la version de l’API. Ces valeurs déterminent les autres propriétés disponibles pour la ressource.

L’exemple suivant montre comment définir le type de ressource et la version de l’API pour un compte de stockage. L’exemple n’affiche pas la déclaration de ressource complète.

```json
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2019-06-01",
    ...
  }
]
```

## <a name="set-resource-name"></a>Définit un nom de ressource

Chaque ressource a un nom. Lorsque vous définissez le nom de la ressource, soyez attentif aux [règles et restrictions applicables aux noms de ressources](../management/resource-name-rules.md).

```json
"parameters": {
  "storageAccountName": {
    "type": "string",
    "minLength": 3,
    "maxLength": 24
  }
},
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2019-06-01",
    "name": "[parameters('storageAccountName')]",
    ...
  }
]
```

## <a name="set-location"></a>Définissez un emplacement

De nombreuses ressources nécessitent un emplacement. Vous pouvez déterminer si la ressource a besoin d’un emplacement via IntelliSense ou une [référence de modèle](/azure/templates/). L’exemple suivant ajoute un paramètre d’emplacement utilisé pour le compte de stockage.

```json
"parameters": {
  "storageAccountName": {
    "type": "string",
    "minLength": 3,
    "maxLength": 24
  },
  "location": {
    "type": "string",
    "defaultValue": "[resourceGroup().location]"
  }
},
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2019-06-01",
    "name": "[parameters('storageAccountName')]",
    "location": "[parameters('location')]",
    ...
  }
]
```

Pour plus d’informations, consultez [Définir l’emplacement des ressources dans un modèle Resource Manager](resource-location.md).

## <a name="set-tags"></a>Définir des étiquettes

Vous pouvez appliquer des étiquettes à une ressource pendant le déploiement. Les étiquettes vous aident à organiser logiquement vos ressources déployées. Pour obtenir des exemples de différentes façons de spécifier les étiquettes, consultez [Étiquettes de modèle ARM](../management/tag-resources.md#arm-templates).

## <a name="set-resource-specific-properties"></a>Définir des propriétés spécifiques d’une ressource

Les propriétés précédentes sont génériques pour la plupart des types de ressources. Après avoir défini ces valeurs, vous devez définir les propriétés qui sont spécifiques du type de ressource que vous déployez.

Utilisez IntelliSense ou une [référence de modèle](/azure/templates/) pour déterminer les propriétés disponibles et celles qui sont requises. L’exemple suivant définit les propriétés restantes pour un compte de stockage.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string",
      "minLength": 3,
      "maxLength": 24
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "functions": [],
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
      },
      "kind": "StorageV2",
      "properties": {
        "accessTier": "Hot"
      }
    }
  ]
}
```

## <a name="next-steps"></a>Étapes suivantes

* Pour déployer une ressource de manière conditionnelle, consultez [Déploiement conditionnel dans des modèles ARM](conditional-resource-deployment.md).
* Pour définir des dépendances de ressources, consultez [Définir l’ordre de déploiement des ressources dans les modèles ARM](./resource-dependency.md).