---
title: Accéder aux journaux des requêtes lentes - Azure CLI - Azure Database pour MySQL
description: Cet article décrit comment accéder aux journaux des requêtes lentes dans Azure Database pour MySQL à l’aide de l’interface Azure CLI.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 4/13/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: f1ab084667f073f7cd43dd986ff4cb8b3f039c75
ms.sourcegitcommit: 8b38eff08c8743a095635a1765c9c44358340aa8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/30/2021
ms.locfileid: "122641128"
---
# <a name="configure-and-access-slow-query-logs-by-using-azure-cli"></a>Configurer et consulter les journaux des requêtes lentes à l’aide d’Azure CLI

[!INCLUDE[applies-to-mysql-single-server](includes/applies-to-mysql-single-server.md)]
Vous pouvez télécharger les journaux des requêtes lentes Azure Database pour MySQL à l’aide d’Azure CLI, l’utilitaire en ligne de commande Azure.

## <a name="prerequisites"></a>Prérequis
Pour parcourir ce guide pratique, vous avez besoin des éléments suivants :
- [Serveur Azure Database pour MySQL](quickstart-create-mysql-server-database-using-azure-cli.md)
- [Azure CLI](/cli/azure/install-azure-cli) ou Azure Cloud Shell dans le navigateur

## <a name="configure-logging"></a>Configuration de la journalisation
Vous pouvez configurer le serveur afin d’accéder au journal des requêtes lentes MySQL, comme suit :
1. Activez la journalisation des requêtes lentes en définissant le paramètre **slow\_query\_log** sur ON.
2. Sélectionnez l’emplacement de sortie des journaux avec **journal\_sortie**. Pour envoyer des journaux vers le stockage local et vers les journaux de diagnostic Azure Monitor, sélectionnez **Fichier**. Pour envoyer des journaux uniquement aux journaux de Azure Monitor, sélectionnez **Aucun**
3. Réglez les autres paramètres tels que **durée\_longue\_d’une requête** et **instructions\_admin\_lentes\_du journal**.

Pour savoir comment définir la valeur de ces paramètres via Azure CLI, consultez [Comment configurer les paramètres du serveur](howto-configure-server-parameters-using-cli.md).

Par exemple, la commande CLI suivante active le journal des requêtes lentes, définit la durée de requête longue sur 10 secondes et désactive la journalisation de l’instruction admin lente. Enfin, elle répertorie les options de configuration à vérifier.
```azurecli-interactive
az mysql server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
az mysql server configuration set --name log_output --resource-group myresourcegroup --server mydemoserver --value FILE
az mysql server configuration set --name long_query_time --resource-group myresourcegroup --server mydemoserver --value 10
az mysql server configuration set --name log_slow_admin_statements --resource-group myresourcegroup --server mydemoserver --value OFF
az mysql server configuration list --resource-group myresourcegroup --server mydemoserver
```

## <a name="list-logs-for-azure-database-for-mysql-server"></a>Répertorier les journaux d’activité pour le serveur Azure Database pour MySQL
Si **log_output** est configurée sur « fichier », vous pouvez accéder aux journaux directement à partir du stockage local du serveur. Pour lister les fichiers journaux des requêtes lentes disponibles pour votre serveur, exécutez la commande [az mysql server-logs list](/cli/azure/mysql/server-logs#az_mysql_server_logs_list).

Vous pouvez répertorier les fichiers journaux pour le serveur **mydemoserver.mysql.database.azure.com** sous le groupe de ressources **myresourcegroup**. Puis, dirigez la liste des fichiers journaux vers un fichier texte appelé **log\_files\_list.txt**.
```azurecli-interactive
az mysql server-logs list --resource-group myresourcegroup --server mydemoserver > log_files_list.txt
```
## <a name="download-logs-from-the-server"></a>Télécharger des journaux d’activité à partir du serveur
Si **log_output** est configurée sur « Fichier », vous pouvez télécharger des fichiers journaux individuels à partir de votre serveur avec la commande [az mysql server-logs download](/cli/azure/mysql/server-logs#az_mysql_server_logs_download).

Utilisez l’exemple suivant pour télécharger le fichier journal spécifique pour le serveur **mydemoserver.mysql.database.azure.com** sous le groupe de ressources **myresourcegroup** dans votre environnement local.
```azurecli-interactive
az mysql server-logs download --name 20170414-mydemoserver-mysql.log --resource-group myresourcegroup --server mydemoserver
```

## <a name="next-steps"></a>Étapes suivantes
- Découvrez les [journaux des requêtes lentes dans Azure Database pour MySQL](concepts-server-logs.md).
