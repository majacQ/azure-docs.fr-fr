---
title: Stratégie de gestion des API Azure - Envoyer les erreurs à Stackify à des fins de journalisation
titleSuffix: Azure API Management
description: Exemple de stratégie de gestion des API Azure - Démontre comment ajouter une stratégie de journalisation des erreurs pour envoyer les erreurs à Stackify à des fins de journalisation.
services: api-management
documentationcenter: ''
author: dlepow
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 10/13/2017
ms.author: danlep
ms.openlocfilehash: dae1ce05df06316aebcfab37b22284ee67813d16
ms.sourcegitcommit: f6e2ea5571e35b9ed3a79a22485eba4d20ae36cc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/24/2021
ms.locfileid: "128644711"
---
# <a name="send-errors-to-stackify-for-logging"></a>Envoyer les erreurs à Stackify à des fins de journalisation

Cet article représente un exemple de stratégie de gestion des API Azure qui explique comment ajouter une stratégie de journalisation des erreurs pour envoyer des erreurs à Stackify à des fins de journalisation. Pour définir ou modifier un code de stratégie, suivez les étapes décrites dans [Définir ou modifier une stratégie](../set-edit-policies.md). Pour voir d’autres exemples, consultez [Exemples de stratégie](../policy-reference.md).

## <a name="policy"></a>Policy

Collez le code dans le bloc **on-error**.

[!code-xml[Main](../../../api-management-policy-samples/examples/Log errors to Stackify.policy.xml)]

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les stratégies APIM :

+ [Stratégies de transformation](../api-management-transformation-policies.md)
+ [Exemples de stratégie](../policy-reference.md)