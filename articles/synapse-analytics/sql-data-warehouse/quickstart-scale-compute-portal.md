---
title: 'Démarrage rapide : Mettre à l’échelle le calcul pour le pool SQL Synapse (portail Azure)'
description: Vous pouvez mettre à l’échelle le calcul pour le pool SQL Synapse (entrepôt de données) à l’aide du portail Azure.
services: synapse-analytics
author: Antvgski
ms.author: anvang
manager: craigg
ms.reviewer: jrasnick
ms.date: 04/28/2020
ms.topic: quickstart
ms.service: synapse-analytics
ms.subservice: sql-dw
ms.custom: seo-lt-2019, azure-synapse, mode-portal
ms.openlocfilehash: de40b989f12b867974e14bf117a0dbb9f13e95ba
ms.sourcegitcommit: 106f5c9fa5c6d3498dd1cfe63181a7ed4125ae6d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/02/2021
ms.locfileid: "131013191"
---
# <a name="quickstart-scale-compute-for-synapse-sql-pool-with-the-azure-portal"></a>Démarrage rapide : Mettre à l’échelle le calcul pour le pool SQL Synapse à l’aide du portail Azure

Vous pouvez mettre à l’échelle le calcul pour le pool SQL Synapse (entrepôt de données) à l’aide du portail Azure. [Effectuez un scale-out du calcul](sql-data-warehouse-manage-compute-overview.md) pour améliorer les performances, ou réduisez-le pour diminuer les coûts. 

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/).

## <a name="before-you-begin"></a>Avant de commencer

Vous pouvez mettre à l’échelle un pool SQL que vous avez déjà ou utiliser la section [Démarrage rapide : Créer et connecter - Portail](create-data-warehouse-portal.md) pour créer un pool SQL nommé **mySampleDataWarehouse**. Ce guide de démarrage rapide met à l’échelle **mySampleDataWarehouse**.

>[!IMPORTANT] 
>Votre pool SQL doit être en ligne pour la mise à l’échelle. 

## <a name="scale-compute"></a>Mise à l’échelle des ressources de calcul

Les ressources de calcul de pool SQL peuvent être mises à l’échelle en augmentant ou en diminuant les unités DWU (Data Warehouse Unit). Le guide [Démarrage rapide : Créer et connecter - Portail](create-data-warehouse-portal.md) a permis de créer **mySampleDataWarehouse** et de l’initialiser avec 400 DWU. Les étapes suivantes ajustent les DWU de **mySampleDataWarehouse**.

Pour modifier les unités Data Warehouse Unit :

1. Cliquez sur **Azure Synapse Analytics (anciennement SQL DW)** dans la page de gauche du portail Azure.
2. Sélectionnez **mySampleDataWarehouse** dans la page **Azure Synapse Analytics (anciennement SQL DW)** . Le pool SQL s’ouvre.
3. Cliquez sur **Scale**.

    ![Cliquez sur Mettre à l’échelle.](./media/quickstart-scale-compute-portal/click-scale.png)

2. Dans le panneau Mettre à l’échelle, déplacez le curseur vers la gauche ou vers la droite pour modifier le paramètre DWU. Sélectionnez ensuite l’échelle.

    ![Déplacez le curseur](./media/quickstart-scale-compute-portal/scale-dwu.png)

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur le pool SQL, consultez le tutoriel [Charger des données dans un pool SQL](./load-data-from-azure-blob-storage-using-copy.md).
