---
title: Fichier Include
description: Fichier Include
author: lrtoyou1223
ms.service: data-factory
ms.subservice: integration-runtime
ms.topic: include
ms.date: 10/09/2019
ms.author: lle
ms.openlocfilehash: 09f61060f9a76036f5e2d58512ddde9248402bbd
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122642020"
---
| Noms de domaine                                          | Ports sortants | Description                |
| ----------------------------------------------------- | -------------- | ---------------------------|
| Cloud public : `*.servicebus.windows.net` <br> Azure Government : `*.servicebus.usgovcloudapi.net` <br> Chine : `*.servicebus.chinacloudapi.cn`   | 443            | Requis par le runtime d’intégration auto-hébergé pour la création interactive. |
| Cloud public : `{datafactory}.{region}.datafactory.azure.net`<br> ou `*.frontend.clouddatahub.net` <br> Azure Government : `{datafactory}.{region}.datafactory.azure.us` <br> Chine : `{datafactory}.{region}.datafactory.azure.cn` | 443            | Requis par le runtime d’intégration auto-hébergé pour se connecter au service Data Factory. <br>Pour les nouvelles fabriques Data Factory créées dans un cloud public, recherchez le nom de domaine complet à partir de votre clé de runtime d’intégration auto-hébergé dont le format est {fabrique_de_données}.{région}.datafactory.azure.net. Pour une ancienne fabrique de données, si vous ne voyez pas le nom de domaine complet dans votre clé d’intégration auto-hébergée, utilisez *.frontend.clouddatahub.net à la place. |
| `download.microsoft.com`    | 443            | Exigé par le runtime d’intégration auto-hébergé pour télécharger les mises à jour. Si vous avez désactivé la mise à jour automatique, vous pouvez ignorer la configuration de ce domaine. |
| URL du coffre de clés | 443           | Requis par Azure Key Vault si vous stockez les informations d’identification dans Key Vault. |
