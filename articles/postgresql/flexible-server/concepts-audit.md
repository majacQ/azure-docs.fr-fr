---
title: Journalisation d’audit - Azure Database pour PostgreSQL - Serveur flexible
description: Concepts relatifs à la journalisation d’audit pgAudit dans Azure Database pour PostgreSQL - Serveur flexible.
author: niklarin
ms.author: nlarin
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: d4659e44475c09a1a42c06041e3f180357af9ee2
ms.sourcegitcommit: f6e2ea5571e35b9ed3a79a22485eba4d20ae36cc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/24/2021
ms.locfileid: "128556024"
---
# <a name="audit-logging-in-azure-database-for-postgresql---flexible-server"></a>Journalisation d’audit dans Azure Database pour PostgreSQL - Serveur flexible

La journalisation d’audit des activités de base de données dans Azure Database pour PostgreSQL - Serveur flexible est disponible via l’extension d’audit PostgreSQL nommée [pgAudit](https://www.pgaudit.org/). pgAudit permet une journalisation d’audit détaillée des sessions et/ou des objets.

> [!IMPORTANT]
> Azure Database pour PostgreSQL - Serveur flexible est en préversion

Si vous souhaitez des journaux de niveau ressource Azure pour des opérations telles que la mise à l’échelle du stockage et du calcul, consultez le [journal d’activité Azure](../../azure-monitor/essentials/platform-logs-overview.md).

## <a name="usage-considerations"></a>Considérations sur l’utilisation
Par défaut, les instructions de journal pgAudit sont émises en même temps que vos instructions de journal habituelles à l’aide de la fonctionnalité de journalisation standard de Postgres. Dans Azure Database pour PostgreSQL - Serveur flexible, vous pouvez également configurer tous les journaux à envoyer au magasin de journaux Azure Monitor pour analyse ultérieure dans Log Analytics. Si vous activez la journalisation des ressources Azure Monitor, vos journaux sont automatiquement envoyés (au format JSON) au services Stockage Azure et Event Hubs, et/ou aux journaux Azure Monitor, conformément à votre choix.

Pour savoir comment configurer la journalisation dans les services Stockage Azure et Event Hubs ou les journaux Azure Monitor, consultez la section concernant les journaux de ressources dans l’[article sur les journaux de serveur](concepts-logging.md).

## <a name="installing-pgaudit"></a>Installation de pgAudit

Pour installer pgAudit, vous devez l’inclure dans les bibliothèques de préchargement partagées du serveur. Une modification apportée au paramètre `shared_preload_libraries` de Postgres nécessite un redémarrage du serveur pour prendre effet. Vous pouvez modifier les paramètres à l’aide du [portail Azure](howto-configure-server-parameters-using-portal.md), d’[Azure CLI](howto-configure-server-parameters-using-cli.md) ou de l’[API REST](/rest/api/postgresql/singleserver/configurations/createorupdate).

À l’aide du [Portail Azure](https://portal.azure.com) :

   1. Sélectionnez votre instance Azure Database pour PostgreSQL – Serveur flexible.
   2. Dans la barre latérale, sélectionnez **Paramètres du serveur**.
   3. Recherchez le paramètre `shared_preload_libraries`.
   4. Sélectionnez **pgaudit**.
     :::image type="content" source="./media/concepts-audit/shared-preload-libraries.png" alt-text="Capture d’écran montrant Azure Database pour PostgreSQL ; activation de shared_preload_libraries pour pgaudit":::
   5. Vous pouvez vérifier que **pgaudit** est chargé dans shared_preload_libraries en exécutant la requête suivante dans psql :
        ```SQL
      show shared_preload_libraries;
      ```
      Vous devriez voir **pgaudit** dans le résultat de la requête qui renverra shared_preload_libraries.

   6. Connectez-vous à votre serveur à l’aide d’un client (par exemple, psql), puis activez l’extension pgAudit.
      ```SQL
      CREATE EXTENSION pgaudit;
      ```

> [!TIP]
> Si une erreur s’affiche, vérifiez que vous avez redémarré le serveur après l’enregistrement de `shared_preload_libraries`.

## <a name="pgaudit-settings"></a>Paramètres pgAudit

pgAudit vous permet de configurer la journalisation d’audit des sessions et des objets. La [journalisation d’audit des sessions](https://github.com/pgaudit/pgaudit/blob/master/README.md#session-audit-logging) émet des journaux détaillés où se trouvent les instructions exécutées. La [journalisation d’audit des objets](https://github.com/pgaudit/pgaudit/blob/master/README.md#object-audit-logging) est limitée à certaines relations. Vous pouvez choisir de configurer un ou plusieurs types de journalisation. 

> [!NOTE]
> Les paramètres pgAudit sont spécifiés globalement et ne peuvent pas être spécifiés au niveau d’une base de données ou d’un rôle.

Une fois que vous avez [activé pgAudit](#installing-pgaudit), vous pouvez configurer ses paramètres pour démarrer la journalisation. Pour configurer pgAudit, vous pouvez suivre les instructions ci-dessous. À l’aide du [Portail Azure](https://portal.azure.com) :

   1. Sélectionnez votre serveur Azure Database pour PostgreSQL.
   2. Dans la barre latérale, sélectionnez **Paramètres du serveur**.
   3. Recherchez les paramètres `pg_audit`.
   4. Choisissez les paramètres de configuration appropriés à modifier. Par exemple, pour démarrer la journalisation, définissez `pgaudit.log` sur `WRITE`. :::image type="content" source="./media/concepts-audit/pgaudit-config.png" alt-text="Capture d’écran montrant Azure Database pour PostgreSQL ; configuration de la journalisation avec pgaudit":::
   5. Cliquez sur le bouton **Enregistrer** pour enregistrer les modifications.


La [documentation pgAudit](https://github.com/pgaudit/pgaudit/blob/master/README.md#settings) donne la définition de chaque paramètre. Testez d’abord les paramètres pour vérifier que vous obtenez le comportement attendu.

> [!NOTE]
> L’affectation de la valeur ON à `pgaudit.log_client` permet de rediriger les journaux vers un processus client (comme psql) au lieu de les écrire dans un fichier. Ce paramètre doit généralement rester désactivé. <br> <br>
> `pgaudit.log_level` est activé uniquement lorsque `pgaudit.log_client` est activé.

> [!NOTE]
> Dans Azure Database pour PostgreSQL - Serveur flexible, `pgaudit.log` ne peut pas être défini à l’aide d’un raccourci de signe `-` (moins) contrairement à ce qui est dit dans la documentation pgAudit. Toutes les classes d’instruction requises (READ, WRITE, etc.) doivent être spécifiées individuellement.

## <a name="audit-log-format"></a>Format du journal d’audit
Chaque entrée d’audit est indiquée par `AUDIT:` en début de ligne. Le format du reste de l’entrée est abordé en détail dans la [documentation pgAudit](https://github.com/pgaudit/pgaudit/blob/master/README.md#format).

## <a name="getting-started"></a>Prise en main
Pour commencer, définissez `pgaudit.log` sur `WRITE`, puis ouvrez vos journaux de serveur pour consulter la sortie. 

## <a name="viewing-audit-logs"></a>Affichage des journaux d’audit
La façon dont vous accédez aux journaux dépend du point de terminaison que vous choisissez. Pour le stockage Azure, consultez l’article [Compte de stockage des journaux](../../azure-monitor/essentials/resource-logs.md#send-to-azure-storage). Pour Event Hubs, consultez l’article [Diffusion des journaux Azure](../../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs).

Pour les journaux Azure Monitor, les journaux sont envoyés à l’espace de travail que vous avez sélectionné. Les journaux Postgres utilisent le mode de collecte **AzureDiagnostics**, pour qu’ils puissent être interrogés à partir de la table AzureDiagnostics. Les champs de la table sont décrits ci-dessous. En savoir plus sur l’interrogation et la génération d’alertes dans la vue d’ensemble [Interroger les journaux Azure Monitor](../../azure-monitor/logs/log-query-overview.md).

Vous pouvez utiliser cette requête pour commencer. Vous pouvez configurer des alertes basées sur les requêtes.

Rechercher toutes les entrées pgAudit dans les journaux Postgres pour un serveur particulier au cours du dernier jour
```kusto
AzureDiagnostics
| where LogicalServerName_s == "myservername"
| where TimeGenerated > ago(1d) 
| where Message contains "AUDIT:"
```

## <a name="next-steps"></a>Étapes suivantes
- [En savoir plus sur la journalisation dans Azure Database pour PostgreSQL - Serveur flexible](concepts-logging.md)
- [Découvrez comment configurer la journalisation dans Azure Database pour PostgreSQL - Serveur flexible et comment accéder aux journaux](howto-configure-and-access-logs.md)