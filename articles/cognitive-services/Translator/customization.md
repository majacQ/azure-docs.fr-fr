---
title: Personnalisation de la traduction – Translator
titleSuffix: Azure Cognitive Services
description: Utilisez Microsoft Translator Hub pour générer votre propre système de traduction automatique à l’aide de la terminologie et du style de votre choix.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 05/26/2020
ms.author: lajanuar
ms.openlocfilehash: 68ae3735fc1f58ebbf8d7afd6a098e7520880058
ms.sourcegitcommit: 92dd25772f209d7d3f34582ccb8985e1a099fe62
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/15/2021
ms.locfileid: "114228753"
---
# <a name="customize-your-text-translations"></a>Personnaliser vos traductions de texte

Custom Translator est une fonctionnalité du service Microsoft Translator qui permet aux utilisateurs de personnaliser la traduction automatique neuronale avancée de Microsoft Translator pour la traduction de texte à l’aide de Translator (version 3 uniquement).

Cette fonctionnalité peut également être utilisée pour personnaliser la traduction vocale quand elle est utilisée avec [Cognitive Services Speech](../speech-service/index.yml).

## <a name="custom-translator"></a>Custom Translator

Avec Custom Translator, vous pouvez générer des systèmes de traduction neuronaux qui comprennent la terminologie utilisée dans votre entreprise et secteur. Le système de traduction personnalisé s’intègre aux applications, sites web et flux de travail existants.

### <a name="how-does-it-work"></a>Comment cela fonctionne-t-il ?

Utilisez vos documents traduits précédemment (dépliants, pages web, documentation, etc.) pour générer un système de traduction qui reflète la terminologie et le style propres à un domaine, plutôt qu’un système de traduction général. Les utilisateurs peuvent charger des documents TMX, XLIFF, TXT, DOCX et XLSX.  

Le système accepte également des données parallèles au niveau du document, mais pas encore alignées au niveau de la phrase. Si les utilisateurs ont accès à des versions du même contenu dans plusieurs langues, mais dans des documents distincts, Custom Translator est capable de faire correspondre automatiquement les phrases dans les documents.  Le système peut utiliser également des données unilingues dans une ou les deux langages pour compléter les données d’apprentissage parallèles et améliorer les traductions.

Le système personnalisé est alors disponible via un appel normal de Translator via le paramètre de catégorie.

Étant donné le type approprié et la quantité de données d’apprentissage, il n’est pas rare d’attendre des gains compris entre 5 et 10, ou encore plus de points BLEU sur la qualité de la traduction grâce à Custom Translator.

Vous trouverez plus de détails sur les différents niveaux de personnalisation en fonction des données disponibles dans le [guide de l’utilisateur de Custom Translator](./custom-translator/overview.md).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Configurer un système de langue personnalisé à l’aide de Custom Translator](./custom-translator/overview.md)
