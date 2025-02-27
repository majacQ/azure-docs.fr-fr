---
title: Stratégie de contrôle de version - Azure Database pour PostgreSQL - Serveur unique et Serveur flexible. (préversion)
description: Décrit la stratégie relative aux versions principales et mineures de PostgreSQL prises en charge dans Azure Database pour PostgreSQL - Serveur unique.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: conceptual
ms.date: 08/03/2020
ms.custom: fasttrack-edit
ms.openlocfilehash: 6d66d030514f121fabe8de36783d879afe0b5ea9
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122524147"
---
# <a name="azure-database-for-postgresql-versioning-policy"></a>Stratégie de contrôle de version Azure Database pour PostgreSQL

Cette page décrit la stratégie de contrôle de version d’Azure Database pour PostgreSQL et s’applique aux modes de déploiement Azure Database pour PostgreSQL - Serveur unique et Azure Database pour PostgreSQL - Serveur flexible (préversion).

## <a name="supported--postgresql-versions"></a>Versions de PostgreSQL prises en charge

Azure Database pour PostgreSQL prend en charge les versions de base de données suivantes.

| Version | Serveur unique | Serveur flexible (préversion) | Hyperscale (Citus) |
| ----- | :------: | :----: | :----: |
| PostgreSQL 13 |  | X  | X |
| PostgreSQL 12 |  | X  | X |
| PostgreSQL 11 | X | X | X |
| PostgreSQL 10 | X |  |  |
| PostgreSQL 9.6 | X |  |  |
| *PostgreSQL 9.5 (retiré)* | X |  |  |

## <a name="major-version-support"></a>Prise en charge de la version principale
Chaque version principale de PostgreSQL est prise en charge par Azure Database pour PostgreSQL à partir de la date du début de la prise en charge de la version par Azure et jusqu’à ce que la version soit supprimée par la communauté PostgreSQL, comme indiqué dans la [stratégie de contrôle de version de la communauté PostgreSQL](https://www.postgresql.org/support/versioning/).

## <a name="minor-version-support"></a>Prise en charge de la version mineure
Azure Database pour PostgreSQL effectue automatiquement des mises à niveau de versions mineures vers la version PostgreSQL préférée d’Azure dans le cadre d’une maintenance périodique. 

## <a name="major-version-retirement-policy"></a>Stratégie de retrait de la version majeure
Le tableau ci-dessous fournit les détails relatifs au retrait des versions principales de PostgreSQL. Les dates suivent la [stratégie de contrôle de version de la communauté PostgreSQL](https://www.postgresql.org/support/versioning/).

| Version | Nouveautés | Date de début de la prise en charge d’Azure | Date de mise hors service|
| ----- | ----- | ------ | ----- |
| [PostgreSQL 9.5 (retiré)](https://www.postgresql.org/about/news/postgresql-132-126-1111-1016-9621-and-9525-released-2165/)| [Caractéristiques](https://www.postgresql.org/docs/9.5/release-9-5.html)  | 18 avril 2018   | 11 février 2021
| [PostgreSQL 9.6](https://www.postgresql.org/about/news/postgresql-96-released-1703/) | [Caractéristiques](https://wiki.postgresql.org/wiki/NewIn96) | 18 avril 2018  | 11 novembre 2021
| [PostgreSQL 10](https://www.postgresql.org/about/news/postgresql-10-released-1786/) | [Caractéristiques](https://wiki.postgresql.org/wiki/New_in_postgres_10) | 4 juin 2018  | 10 novembre 2022
| [PostgreSQL 11](https://www.postgresql.org/about/news/postgresql-11-released-1894/) | [Caractéristiques](https://www.postgresql.org/docs/11/release-11.html) | 24 juillet 2019  | 9 novembre 2023
| [PostgreSQL 12](https://www.postgresql.org/about/news/postgresql-12-released-1976/) | [Caractéristiques](https://www.postgresql.org/docs/12/release-12.html) | 22 septembre 2020  | 14 novembre 2024
| [PostgreSQL 13](https://www.postgresql.org/about/news/postgresql-13-released-2077/) | [Caractéristiques](https://www.postgresql.org/docs/13/release-13.html) | 25 mai 2021   | 13 novembre 2025

## <a name="retired-postgresql-engine-versions-not-supported-in-azure-database-for-postgresql"></a>Versions de moteur PostgreSQL supprimées non prises en charge dans Azure Database pour PostgreSQL

Vous pouvez continuer à exécuter la version supprimée dans Azure Database pour PostgreSQL. Toutefois, notez les restrictions suivantes après la date de suppression pour chaque version de la base de données PostgreSQL :
- Dans la mesure où la communauté ne publie plus de correctifs de bogues ou de correctifs de sécurité, Azure Database pour PostgreSQL n’effectue pas de mise à jour corrective sur le moteur de base de données supprimé pour la résolution de bogues ou de problèmes de sécurité. Il n’y aura pas non plus de mesures de sécurité concernant le moteur de base de données supprimé. Vous pouvez dès lors être confronté à des failles de sécurité ou à d’autres problèmes. Toutefois, Azure continuera à effectuer des opérations de maintenance et de mise à jour périodiques pour l’hôte, le système d’exploitation, les conteneurs et tout autre composant lié aux services.
- Si un problème de support que vous pouvez rencontrer est lié au moteur PostgreSQL lui-même, dans la mesure où la Communauté ne fournit plus les correctifs, nous ne sommes pas en mesure de vous fournir du support technique. Dans ce cas, vous devrez mettre à niveau votre base de données vers une des versions prises en charge.
- Vous ne serez plus en mesure de créer des serveurs de base de données pour la version supprimée. Toutefois, vous pouvez effectuer des récupérations à un instant dans le passé et créer des réplicas de lecture pour vos serveurs existants.
- Les nouvelles fonctionnalités de service développées par Azure Database pour PostgreSQL peuvent uniquement être disponibles pour les versions de serveur de base de données prises en charge.
- Les contrats de niveau de service de durée de fonctionnement s’appliquent uniquement aux problèmes liés au service Azure Database pour PostgreSQL et non aux temps d’arrêt causés par des bogues liés au moteur de base de données.  
- En cas de menace sérieuse pour le service provoquée par la vulnérabilité du moteur de base de données PostgreSQL identifiée dans la version de la base de données supprimée, Azure peut décider d’arrêter votre serveur de base de données pour sécuriser le service. Dans ce cas, vous serez averti de la mise à niveau du serveur avant sa mise en ligne.

## <a name="postgresql-version-syntax"></a>Syntaxe de version PostgreSQL
Avant PostgreSQL version 10, la [stratégie de gestion de version PostgreSQL](https://www.postgresql.org/support/versioning/) considérait une _mise à niveau principale_ comme une augmentation du premier _ou_ du deuxième nombre. Par exemple, une mise à niveau de la version 9.5 vers la version 9.6 était considérée comme une mise à niveau _principale_. Depuis la version 10, seule une modification du premier numéro est considérée comme une mise à niveau principale. Par exemple, une mise à niveau de la version 10.0 vers la version 10.1 est une mise à niveau _mineure_. Une mise à niveau de la version 10 à 11 est une mise à niveau _principale_.

## <a name="next-steps"></a>Étapes suivantes
- Consultez Azure Database pour PostgreSQL - [Versions prises en charge](./concepts-supported-versions.md) Serveur unique
- Consultez Azure Database pour PostgreSQL - [Versions prises en charge](flexible-server/concepts-supported-versions.md) Serveur flexible (préversion)
- Pour plus d’informations sur la façon d’effectuer des mises à niveau de version majeures, consultez la documentation [Mises à niveau de version majeures](how-to-upgrade-using-dump-and-restore.md).
- Pour plus d’informations sur les extensions PostgrSQL prises en charge, consultez [le document Extensions](concepts-extensions.md).
