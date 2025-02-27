---
title: Tutoriel - Supprimer un cluster Azure Red Hat OpenShift
description: Dans ce tutoriel, découvrez comment supprimer un cluster Azure Red Hat OpenShift à l’aide d’Azure CLI
author: sakthi-vetrivel
ms.custom: fasttrack-edit, devx-track-azurecli
ms.author: suvetriv
ms.topic: tutorial
ms.service: azure-redhat-openshift
ms.date: 04/24/2020
ms.openlocfilehash: e53c8caed2cf3c72bf416b1416183eb0c654582e
ms.sourcegitcommit: 5fabdc2ee2eb0bd5b588411f922ec58bc0d45962
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/23/2021
ms.locfileid: "112541119"
---
# <a name="tutorial-delete-an-azure-red-hat-openshift-4-cluster"></a>Tutoriel : Supprimer un cluster Azure Red Hat OpenShift 4

Dans ce tutoriel, en trois parties, un cluster Azure Red Hat OpenShift exécutant OpenShift 4 est supprimé. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Supprimer un cluster Azure Red Hat OpenShift


## <a name="before-you-begin"></a>Avant de commencer

Dans les tutoriels précédents, un cluster Azure Red Hat OpenShift a été créé et rendu accessible au moyen de la console web OpenShift. Si vous n’avez pas effectué ces étapes et que vous souhaitez suivre cette procédure, commencez par le [Tutoriel 1 – Créer un cluster Azure Red Hat OpenShift 4](tutorial-create-cluster.md).

Si vous choisissez d’installer et d’utiliser l’interface CLI localement, ce tutoriel demande au minimum la version 2.6.0 d’Azure CLI. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Si vous utilisez Azure CLI localement, exécutez `az login` pour vous connecter à Azure.

```bash
az login
```

Si vous avez accès à plusieurs abonnements, exécutez `az account set -s {subscription ID}` en remplaçant `{subscription ID}` par l’abonnement que vous souhaitez utiliser.

## <a name="delete-the-cluster"></a>Supprimer le cluster

Dans les tutoriels précédents, les variables suivantes ont été définies.

```bash
CLUSTER=yourclustername
RESOURCEGROUP=yourresourcegroup
```

À l’aide de ces valeurs, supprimez votre cluster :

```bash
az aro delete --resource-group $RESOURCEGROUP --name $CLUSTER
```

Vous serez ensuite invité à confirmer la suppression du cluster. Après que vous aurez confirmé avec `y`, la suppression du cluster prendra plusieurs minutes. Quand la commande est terminée, l’ensemble du groupe de ressources et toutes les ressources qu’il contient, notamment le cluster, sont supprimés.

## <a name="next-steps"></a>Étapes suivantes

Dans cette partie du tutoriel, vous avez appris à :
> [!div class="checklist"]
> * Supprimer un cluster Azure Red Hat OpenShift 4

En savoir plus sur l’utilisation d’OpenShift avec la [documentation Red Hat OpenShift](https://docs.openshift.com/container-platform/4.6/welcome/index.html) officielle
