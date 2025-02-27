---
title: Placement d'un groupe de serveurs PostgreSQL Hyperscale sur les nœuds de cluster Kubernetes
description: Explique comment les instances PostgreSQL qui forment un groupe de serveurs PostgreSQL Hyperscale sont placées sur les nœuds de cluster Kubernetes
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 11/03/2021
ms.topic: how-to
ms.openlocfilehash: 05dd347914d7be942c00232de78cf89484f07555
ms.sourcegitcommit: e41827d894a4aa12cbff62c51393dfc236297e10
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2021
ms.locfileid: "131555321"
---
# <a name="azure-arc-enabled-postgresql-hyperscale-server-group-placement"></a>Placement d’un groupe de serveurs PostgreSQL Hyperscale pour Azure Arc

Cet article repose sur un exemple qui illustre la façon dont les instances PostgreSQL du groupe de serveurs PostgreSQL Hyperscale pour Azure Arc sont placées sur les nœuds physiques du cluster Kubernetes qui les héberge. 

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="configuration"></a>Configuration

Dans cet exemple, nous utilisons un cluster Azure Kubernetes Service (AKS) qui comporte quatre nœuds physiques. 

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/1_cluster_portal.png" alt-text="Cluster AKS à 4 nœuds sur le portail Azure":::

Répertoriez les nœuds physiques du cluster Kubernetes. Exécutez la commande suivante :

```console
kubectl get nodes
```

`kubectl` retourne les quatre nœuds physiques contenus dans le cluster Kubernetes :

```output
NAME                                STATUS   ROLES   AGE   VERSION
aks-agentpool-42715708-vmss000000   Ready    agent   11h   v1.17.9
aks-agentpool-42715708-vmss000001   Ready    agent   11h   v1.17.9
aks-agentpool-42715708-vmss000002   Ready    agent   11h   v1.17.9
aks-agentpool-42715708-vmss000003   Ready    agent   11h   v1.17.9
```

L'architecture peut être représentée comme suit :

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/2_logical_cluster.png" alt-text="Représentation logique de 4 nœuds regroupés dans un cluster Kubernetes":::

Le cluster Kubernetes héberge un contrôleur de données Azure Arc et un groupe de serveurs PostgreSQL Hyperscale pour Azure Arc . Ce groupe de serveurs est constitué de trois instances PostgreSQL : un coordinateur et deux Workers.

Répertoriez les pods à l'aide de la commande suivante :

```console
kubectl get pods -n arc3
```
`kubectl` retourne :

```output
NAME                 READY   STATUS    RESTARTS   AGE
…
postgres01c-0         3/3     Running   0          9h
postgres01w-0         3/3     Running   0          9h
postgres01w-1         3/3     Running   0          9h
```
Chacun de ces pods héberge une instance PostgreSQL. Ensemble, les pods forment le groupe de serveurs PostgreSQL Hyperscale pour Azure Arc :

```output
Pod name        Role in the server group
postgres01c-0 Coordinator
postgres01w-0   Worker
postgres01w-1   Worker
```

## <a name="placement"></a>Sélection élective
Voyons comment Kubernetes place les pods du groupe de serveurs. Décrivez chaque pod et identifiez le nœud physique du cluster Kubernetes sur lequel ils sont placés. Par exemple, pour le Coordinateur, exécutez la commande suivante :

```console
kubectl describe pod postgres01c-0 -n arc3
```

`kubectl` retourne :

```output
Name:         postgres01c-0
Namespace:    arc3
Priority:     0
Node:         aks-agentpool-42715708-vmss000000
Start Time:   Thu, 17 Sep 2020 00:40:33 -0700
…
```

L'exécution de cette commande pour chacun des pods nous permet de résumer le placement actuel comme suit :

| Rôle du groupe de serveurs|Pod du groupe de serveurs|Nœud physique Kubernetes hébergeant le pod |
|--|--|--|
| Worker|postgres01-1|aks-agentpool-42715708-vmss000002 |
| Worker|postgres01-2|aks-agentpool-42715708-vmss000003 |

Et notez également, dans la description des pods, les noms des conteneurs hébergés par chaque pod. Par exemple, pour le deuxième Worker, exécutez la commande suivante :

```console
kubectl describe pod postgres01w-1 -n arc3
```

`kubectl` retourne :

```output
…
Node:         aks-agentpool-42715708-vmss000003/10.240.0.7
..
Containers:
  Fluentbit:
…
  Postgres:
…
  Telegraf:
…
```

Chacun des pods du groupe de serveurs PostgreSQL Hyperscale pour Azure Arc héberge les trois conteneurs suivants :

|Conteneurs|Description
|----|----|
|`Fluentbit` |Collecteur de journaux * de données : https://fluentbit.io/
|`Postgres`|Instance PostgreSQL faisant partie du groupe de serveurs PostgreSQL Hyperscale pour Azure Arc
|`Telegraf` |Collecteur de métriques : https://www.influxdata.com/time-series-platform/telegraf/

L’architecture ressemble à ceci :

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/3_pod_placement.png" alt-text="3 pods, chacun placé sur un nœud distinct":::

Cela signifie qu’à ce stade, chacune des instances PostgreSQL qui constituent le groupe de serveurs PostgreSQL Hyperscale pour Azure Arc est hébergée sur un hôte physique spécifique dans le conteneur Kubernetes. Il s’agit de la configuration la plus performante du groupe de serveurs PostgreSQL Hyperscale pour Azure Arc, car chaque rôle (coordinateur et Workers) utilise les ressources de chaque nœud physique. Ces ressources ne sont pas partagées entre plusieurs rôles PostgreSQL.

## <a name="scale-out-azure-arc-enabled-postgresql-hyperscale"></a>Effectuer un scale-out de PostgreSQL Hyperscale pour Azure Arc

Effectuons maintenant un scale-out pour ajouter un troisième nœud Worker au groupe de serveurs et observons ce qui se passe. Une quatrième instance PostgreSQL est créée, et celle-ci est hébergée dans un quatrième pod.
Pour effectuer un scale-out, exécutez la commande suivante :

```azurecli
az postgres arc-server edit --name postgres01 --workers 3 --k8s-namespace arc3 --use-k8s
```

Ce qui génère la sortie suivante :

```output
Updating postgres01 in namespace `arc3`
postgres01 is Ready
```

Répertoriez les groupes de serveurs déployés dans le contrôleur de données Azure Arc et vérifiez que le groupe de serveurs s'exécute désormais avec trois Workers. Exécutez la commande suivante :

```azurecli
az postgres arc-server list --k8s-namespace arc3 --use-k8s
```

Et notez que nous sommes passés de deux à trois Workers :

```output
[
  {
    "name": "postgres01",
    "replicas": 1,
    "state": "Ready",
    "workers": 3
  }
]
```

Comme précédemment, notez que le groupe de serveurs utilise désormais un total de quatre pods :

```console
kubectl get pods -n arc3
```

```output
NAME                 READY   STATUS    RESTARTS   AGE
…
postgres01c-0         3/3     Running   0          11h
postgres01w-0         3/3     Running   0          11h
postgres01w-1         3/3     Running   0          11h
postgres01w-2         3/3     Running   0          5m2s
```

Et décrivez le nouveau pod pour identifier le nœud physique du cluster Kubernetes sur lequel il est hébergé.
Exécutez la commande suivante :

```console
kubectl describe pod postgres01w-2 -n arc3
```

Pour identifier le nom du nœud d'hébergement :

```output
Name:         postgres01w-2
Namespace:    arc3
Priority:     0
Node:         aks-agentpool-42715708-vmss000000
```

Le placement des instances PostgreSQL sur les nœuds physiques du cluster est maintenant le suivant :

|Rôle du groupe de serveurs|Pod du groupe de serveurs|Nœud physique Kubernetes hébergeant le pod
|-----|-----|-----
|Coordinatrice|postgres01c-0|aks-agentpool-42715708-vmss000000
|Worker|postgres01w-1|aks-agentpool-42715708-vmss000002
|Worker|postgres01w-2|aks-agentpool-42715708-vmss000003
|Worker|postgres01w-3|aks-agentpool-42715708-vmss000000

Et notez que le pod du nouveau Worker (postgres01w-2) a été placé sur le même nœud que le coordinateur. 

L’architecture ressemble à ceci :

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/4_pod_placement_.png" alt-text="Quatrième pod sur le même nœud que le coordinateur":::

Pourquoi le nouveau Worker/pod n'est-il pas placé sur le nœud physique restant du cluster Kubernetes aks-agentpool-42715708-vmss000003 ?

Parce que le dernier nœud physique du cluster Kubernetes héberge en fait plusieurs pods qui hébergent eux-mêmes des composants supplémentaires requis pour exécuter les services de données compatibles Azure Arc. Au moment de la planification, Kubernetes a estimé que le nœud physique aks-agentpool-42715708-vmss000000 était le meilleur candidat pour héberger le Worker supplémentaire. 

Les commandes déjà utilisées précédemment nous permettent de voir ce que chaque nœud physique héberge :

|Noms des autres pods\* |Usage|Nœud physique Kubernetes hébergeant les pods
|----|----|----
|bootstrapper-jh48b|Service qui traite les demandes entrantes pour créer, modifier et supprimer des ressources personnalisées telles que des instances gérées SQL, des groupes de serveurs PostgreSQL Hyperscale et des contrôleurs de données.|aks-agentpool-42715708-vmss000003
|control-gwmbs||aks-agentpool-42715708-vmss000002
|controldb-0|Magasin de données du contrôleur qui est utilisé pour stocker la configuration et l’état du contrôleur de données.|aks-agentpool-42715708-vmss000001
|controlwd-zzjp7|Service « surveillant » du contrôleur qui garde un œil sur la disponibilité du contrôleur de données.|aks-agentpool-42715708-vmss000000
|logsdb-0|Instance de recherche élastique qui est utilisée pour stocker tous les journaux collectés sur l’ensemble des pods des services de données Arc. Elasticsearch, reçoit les données du conteneur `Fluentbit` de chaque Pod|aks-agentpool-42715708-vmss000003
|logsui-5fzv5|Instance Kibana qui se trouve au-dessus de la base de données de recherche élastique pour présenter une interface graphique utilisateur Log Analytics.|aks-agentpool-42715708-vmss000003
|metricsdb-0|Instance InfluxDB qui est utilisée pour stocker toutes les métriques collectées sur l’ensemble des pods des services de données Arc. InfluxDB, reçoit les données du conteneur `Telegraf` de chaque Pod|aks-agentpool-42715708-vmss000000
|metricsdc-47d47|Jeu de démons déployé sur tous les nœuds Kubernetes du cluster pour collecter des métriques au niveau du nœud à leur sujet.|aks-agentpool-42715708-vmss000002
|metricsdc-864kj|Jeu de démons déployé sur tous les nœuds Kubernetes du cluster pour collecter des métriques au niveau du nœud à leur sujet.|aks-agentpool-42715708-vmss000001
|metricsdc-l8jkf|Jeu de démons déployé sur tous les nœuds Kubernetes du cluster pour collecter des métriques au niveau du nœud à leur sujet.|aks-agentpool-42715708-vmss000003
|metricsdc-nxm4l|Jeu de démons déployé sur tous les nœuds Kubernetes du cluster pour collecter des métriques au niveau du nœud à leur sujet.|aks-agentpool-42715708-vmss000000
|metricsui-4fb7l|Instance Grafana qui se trouve au-dessus de la base de données InfluxDB pour présenter une interface graphique utilisateur du tableau de bord d’analyse.|aks-agentpool-42715708-vmss000003
|mgmtproxy-4qppp|Couche de proxy d’application web qui se trouve devant les instances Grafana et Kibana.|aks-agentpool-42715708-vmss000002

> \* Le suffixe des noms de pods varie en fonction des autres déploiements. De plus, nous ne répertorions ici que les pods hébergés dans l'espace de noms Kubernetes du contrôleur de données Azure Arc.

L’architecture ressemble à ceci :

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/5_full_list_of_pods.png" alt-text="Tous les pods de l'espace de noms, sur différents nœuds":::

Comme décrit ci-dessus, les nœuds coordinateurs (Pod 1) du groupe de serveurs PostgreSQL Hyperscale pour Azure Arc partagent les mêmes ressources physiques que le troisième nœud Worker (Pod 4) du groupe de serveurs. Cette configuration est acceptable, car le nœud coordinateur utilise généralement très peu de ressources par rapport à un nœud Worker. Pour cette raison, choisissez avec soin :
- la taille du cluster Kubernetes et les caractéristiques de chacun de ses nœuds physiques (mémoire, vCore) ;
- le nombre de nœuds physiques contenus dans le cluster Kubernetes ;
- les applications ou charges de travail que vous hébergez sur le cluster Kubernetes.

L’hébergement d’un trop grand nombre de charges de travail sur le cluster Kubernetes peut avoir un effet d’étranglement sur le groupe de serveurs PostgreSQL Hyperscale pour Azure Arc. Dans ce cas, sa capacité de mise à l'échelle horizontale sera réduite. Les performances du système ne dépendent pas seulement du placement ou des caractéristiques physiques des nœuds physiques ou du système de stockage. Elles dépendent également de la façon dont vous configurez chacune des ressources exécutées à l’intérieur du cluster Kubernetes (y compris PostgreSQL Hyperscale pour Azure Arc), comme les requêtes et les limites que vous définissez pour la mémoire et vCore. Le volume de charge de travail que vous pouvez héberger sur un cluster Kubernetes donné dépend des caractéristiques du cluster Kubernetes, de la nature des charges de travail, du nombre d'utilisateurs, de la manière dont les opérations du cluster Kubernetes sont effectuées, etc...

## <a name="scale-out-aks"></a>Effectuer un scale-out d'AKS

Il s’agit ici de démontrer que la mise à l’échelle horizontale du cluster AKS et du serveur PostgreSQL Hyperscale pour Azure Arc permet de tirer le meilleur parti des hautes performances de PostgreSQL Hyperscale pour Azure Arc.
Nous allons ajouter un cinquième nœud au cluster AKS :

:::row:::
    :::column:::
        Avant
    :::column-end:::
    :::column:::
        Après
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        :::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/6_layout_before.png" alt-text="Disposition du portail Azure avant":::
    :::column-end:::
    :::column:::
        :::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/7_layout_after.png" alt-text="Disposition de portail Azure après":::
    :::column-end:::
:::row-end:::

L’architecture ressemble à ceci :

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/8_logical_layout_after.png" alt-text="Disposition logique sur le cluster Kubernetes après la mise à jour":::

Voyons quels pods de l'espace de noms du contrôleur de données Arc sont hébergés sur le nouveau nœud physique AKS en exécutant la commande suivante :

```console
kubectl describe node aks-agentpool-42715708-vmss000004
```

Et mettons à jour la représentation de l'architecture de notre système :

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/9_updated_list_of_pods.png" alt-text="Tous les pods sur le diagramme logique du cluster":::

Nous constatons que le nouveau nœud physique du cluster Kubernetes héberge uniquement le pod de métriques requis pour les services de données Azure Arc. Notez que, dans cet exemple, nous nous concentrons uniquement sur l'espace de noms du contrôleur de données Arc ; nous ne représentons pas les autres pods.

## <a name="scale-out-azure-arc-enabled-postgresql-hyperscale-again"></a>Effectuer un nouveau scale-out de PostgreSQL Hyperscale pour Azure Arc

Le cinquième nœud physique n'héberge pas encore de charge de travail. Au fil du scale-out de PostgreSQL Hyperscale pour Azure Arc, Kubernetes optimisera le placement du nouveau pod PostgreSQL, et il ne devrait pas le colocaliser sur des nœuds physiques qui hébergent déjà plus de charges de travail. Exécutez la commande suivante pour faire passer PostgreSQL Hyperscale pour Azure Arc de 3 à 4 Workers. À la fin de l'opération, le groupe de serveurs sera constitué et réparti sur cinq instances PostgreSQL, un coordinateur et quatre Workers.

```azurecli
az postgres arc-server edit --name postgres01 --workers 4 --k8s-namespace arc3 --use-k8s
```

Ce qui génère la sortie suivante :

```output
Updating postgres01 in namespace `arc3`
postgres01 is Ready
```

Répertoriez les groupes de serveurs déployés dans le contrôleur de données et vérifiez que le groupe de serveurs s'exécute désormais avec quatre Workers :

```azurecli
az postgres arc-server list --k8s-namespace arc3 --use-k8s
```

Et notez que nous sommes passés de trois à quatre Workers. 

```console
[
  {
    "name": "postgres01",
    "replicas": 1,
    "state": "Ready",
    "workers": 4
  }
]
```

Comme précédemment, notez que le groupe de serveurs utilise maintenant quatre pods :

```output
kubectl get pods -n arc3

NAME                 READY   STATUS    RESTARTS   AGE
…
postgres01c-0         3/3     Running   0          13h
postgres01w-0         3/3     Running   0          13h
postgres01w-1         3/3     Running   0          13h
postgres01w-2         3/3     Running   0          179m
postgres01w-3         3/3     Running   0          3m13s
```

La forme du groupe de serveurs est désormais la suivante :

|Rôle du groupe de serveurs|Pod du groupe de serveurs
|----|-----
|Coordinatrice|postgres01c-0
|Worker|postgres01w-0
|Worker|postgres01w-1
|Worker|postgres01w-2
|Worker|postgres01w-3

Nous allons décrire le pod postgres01w-3 pour identifier le nœud physique sur lequel il est hébergé :

```console
kubectl describe pod postgres01w-3 -n arc3
```

Et notez sur quels pods il s'exécute :

|Rôle du groupe de serveurs|Pod du groupe de serveurs| Pod
|----|-----|------
|Coordinatrice|postgres01c-0|aks-agentpool-42715708-vmss000000
|Worker|postgres01w-0|aks-agentpool-42715708-vmss000002
|Worker|postgres01w-1|aks-agentpool-42715708-vmss000003
|Worker|postgres01w-2|aks-agentpool-42715708-vmss000000
|Worker|postgres01w-3|aks-agentpool-42715708-vmss000004

Et l'architecture se présente comme suit :

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/10_kubernetes_schedules_newest_pod.png" alt-text="Kubernetes planifie le nouveau pod dans le nœud le moins utilisé":::

Kubernetes a planifié le nouveau pod PostgreSQL dans le nœud physique le moins chargé du cluster Kubernetes.

## <a name="summary"></a>Récapitulatif

Pour tirer le meilleur parti de la scalabilité et des performances de la mise à l’échelle horizontale du groupe de serveurs compatibles Azure Arc, vous devez éviter les conflits entre les ressources au sein du cluster Kubernetes :
- entre le groupe de serveurs PostgreSQL Hyperscale pour Azure Arc et d’autres charges de travail hébergées sur le même cluster Kubernetes ;
- entre toutes les instances PostgreSQL qui constituent le groupe de serveurs PostgreSQL Hyperscale pour Azure Arc.

Pour ce faire, vous disposez de plusieurs méthodes :
- Effectuez un scale-out de Kubernetes et de PostgreSQL Hyperscale pour Azure Arc : envisagez d’effectuer une mise à l’échelle horizontale du cluster Kubernetes de la même manière que vous mettez à l’échelle le groupe de serveurs PostgreSQL Hyperscale pour Azure Arc. Ajoutez un nœud physique au cluster pour chaque Worker que vous ajoutez au groupe de serveurs.
- Effectuez un scale-out de Postgres Hyperscale pour Azure Arc sans mise à l’échelle de Kubernetes : en définissant les contraintes de ressources appropriées (requête et limites sur la mémoire et vCore) sur les charges de travail hébergées dans Kubernetes (PostgreSQL Hyperscale pour Azure Arc inclus), vous permettrez la colocation des charges de travail sur Kubernetes et réduirez les risques de conflits entre les ressources. Vous devez vous assurer que les caractéristiques physiques des nœuds physiques du cluster Kubernetes permettent de respecter les contraintes de ressources que vous définissez. Vous devez également veiller à ce que l'équilibre soit maintenu à mesure que les charges de travail évoluent ou que d'autres charges de travail sont ajoutées dans le cluster Kubernetes.
- Utilisez les mécanismes Kubernetes (sélecteur de pod, affinité, anti-affinité) pour influencer le placement des pods.

## <a name="next-steps"></a>Étapes suivantes

[Effectuer un scale-out de votre groupe de serveurs PostgreSQL Hyperscale pour Azure Arc en ajoutant des nœuds Worker](scale-out-in-postgresql-hyperscale-server-group.md)
