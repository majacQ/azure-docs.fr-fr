---
title: Guide de résolution des problèmes de performances des partages de fichiers Azure
description: Résoudre les problèmes de niveau de performance connus avec les partages de fichiers Azure. Découvrez les causes potentielles et les solutions de contournement associées lorsque ces problèmes surviennent.
author: jeffpatt24
ms.service: storage
ms.topic: troubleshooting
ms.date: 07/06/2021
ms.author: jeffpatt
ms.subservice: files
ms.openlocfilehash: 44cbae10fc83ddbacf9abc8fd6510667c1f27e00
ms.sourcegitcommit: 2ed2d9d6227cf5e7ba9ecf52bf518dff63457a59
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2021
ms.locfileid: "132524376"
---
# <a name="troubleshoot-azure-file-shares-performance-issues"></a>Résoudre les problèmes de performances des partages de fichiers Azure

Cet article répertorie certains problèmes courants liés à des partages de fichiers Azure. Il décrit des causes potentielles et des solutions de contournement de ces problèmes.

## <a name="applies-to"></a>S’applique à
| Type de partage de fichiers | SMB | NFS |
|-|:-:|:-:|
| Partages de fichiers Standard (GPv2), LRS/ZRS | ![Oui](../media/icons/yes-icon.png) | ![Non](../media/icons/no-icon.png) |
| Partages de fichiers Standard (GPv2), GRS/GZRS | ![Oui](../media/icons/yes-icon.png) | ![Non](../media/icons/no-icon.png) |
| Partages de fichiers Premium (FileStorage), LRS/ZRS | ![Oui](../media/icons/yes-icon.png) | ![Oui](../media/icons/yes-icon.png) |

## <a name="high-latency-low-throughput-and-general-performance-issues"></a>Latence élevée, débit faible et problèmes généraux de niveau de performance

### <a name="cause-1-share-was-throttled"></a>Cause 1 : Le partage a été limité

Les requêtes sont limitées lorsque les limites d’opérations d’E/S par seconde (IOPS) ou les limites d’entrée/de sortie d’un partage de fichiers sont atteintes. Pour comprendre les limites des partages de fichiers standard et Premium, consultez [Cibles de partage de fichiers et de mise à l’échelle des fichiers](./storage-files-scale-targets.md#azure-file-share-scale-targets).

Pour vérifier si votre partage est limité, vous pouvez accéder aux métriques Azure et les utiliser dans le portail.

1. Dans le portail Azure, accédez à votre compte de stockage.

1. Dans le volet de gauche, sous **Supervision**, sélectionnez **Métriques**.

1. Sélectionnez **Fichier** comme espace de noms de métrique pour votre étendue de compte de stockage.

1. Sélectionnez **Transactions** comme métrique.

1. Ajoutez un filtre pour **Type de réponse**, puis vérifiez si des requêtes ont été limitées. 

    Pour les partages de fichiers standard, les types de réponse suivants sont consignés si une requête est limitée :

    - SuccessWithThrottling
    - SuccessWithShareIopsThrottling
    - ClientShareIopsThrottlingError

    Pour les partages de fichiers premium, les types de réponse suivants sont consignés si une requête est limitée :

    - SuccessWithShareEgressThrottling
    - SuccessWithShareIngressThrottling
    - SuccessWithShareIopsThrottling
    - ClientShareEgressThrottlingError
    - ClientShareIngressThrottlingError
    - ClientShareIopsThrottlingError

    Pour en savoir plus sur chaque type de réponse, consultez [Dimensions de mesures](./storage-files-monitoring-reference.md#metrics-dimensions).

    ![Capture d’écran des options de métriques pour les partages de fichiers premium, montrant un filtre de propriété « Type de réponse ».](media/storage-troubleshooting-premium-fileshares/metrics.png)

    > [!NOTE]
    > Pour recevoir une alerte, consultez la section [« Comment créer une alerte si un partage de fichiers est limité »](#how-to-create-an-alert-if-a-file-share-is-throttled), plus loin dans cet article.

#### <a name="solution"></a>Solution

- Si vous utilisez un partage de fichiers standard, [activez les partages de fichiers volumineux](storage-how-to-create-file-share.md#enable-large-files-shares-on-an-existing-account) sur votre compte de stockage et [augmentez la taille du quota de partage de fichiers pour tirer parti de la prise en charge des partages de fichiers volumineux](storage-how-to-create-file-share.md#expand-existing-file-shares). Les partages de fichiers volumineux prennent en charge des limites élevées d’IOPS et de bande passante. Pour plus d’informations, consultez [Objectifs de performance et de scalabilité d’Azure Files](storage-files-scale-targets.md).
- Si vous utilisez un partage de fichiers premium, augmentez la taille du partage de fichiers approvisionné pour rehausser la limite d’IOPS. Pour plus d’informations, consultez [Présentation du provisionnement des partages de fichiers Premium](./understanding-billing.md#provisioned-model).

### <a name="cause-2-metadata-or-namespace-heavy-workload"></a>Cause 2 : Métadonnées ou charge de travail importante de l’espace de noms

Si la majorité de vos demandes sont centrées sur les métadonnées (comme `createfile`, `openfile`, `closefile`, `queryinfo` ou `querydirectory`), la latence sera plus importante que celle des opérations de lecture/d’écriture.

Pour déterminer si la plupart de vos demandes sont centrées sur des métadonnées, commencez par suivre les étapes 1 à 4 décrites précédemment dans Cause 1. Pour l’étape 5, au lieu d’ajouter un filtre pour le **Type de réponse**, ajoutez un filtre de propriété pour **Nom de l’API**.

![Capture d’écran des options de métriques pour les partages de fichiers premium, montrant un filtre de propriété « Nom de l’API ».](media/storage-troubleshooting-premium-fileshares/MetadataMetrics.png)

#### <a name="workaround"></a>Solution de contournement

- Vérifiez si l’application peut être modifiée pour réduire le nombre d’opérations sur les métadonnées.
- Ajoutez un disque dur virtuel (VHD) sur le partage de fichiers, et montez-le à partir du client pour effectuer des opérations de fichiers sur les données. Cette approche fonctionne pour des scénarios à un seul rédacteur/lecteur ou des scénarios avec plusieurs lecteurs et aucun rédacteur. Comme le système de fichiers appartient au client plutôt qu’à Azure Files, les opérations sur les métadonnées peuvent être locales. La configuration offre des performances similaires à celles d’un stockage local directement attaché.
    -   Pour monter un disque dur virtuel (VHD) sur un client Windows, utilisez la cmdlet PowerShell [Mount-DiskImage](/powershell/module/storage/mount-diskimage).
    -   Pour monter un disque dur virtuel (VHD) sur Linux, consultez la documentation de votre distribution Linux.     

### <a name="cause-3-single-threaded-application"></a>Cause 3 : Application à thread unique

Si l’application que vous utilisez est à thread unique, cette configuration peut entraîner un débit d’IOPS sensiblement inférieur au débit maximal possible, selon la taille de votre partage approvisionné.

#### <a name="solution"></a>Solution

- Augmentez le parallélisme de l’application en augmentant le nombre de threads.
- Basculer vers des applications où le parallélisme est possible. Par exemple, pour des opérations de copie, vous pourriez utiliser AzCopy ou RoboCopy à partir de clients Windows, ou la commande **parallèle** à partir de clients Linux.

### <a name="cause-4-number-of-smb-channels-exceeds-four"></a>Cause 4 : Le nombre de canaux SMB est supérieur à quatre

Si vous utilisez SMB Multichannel et que vous avez plus de quatre canaux, cela entraîne une dégradation des performances. Pour déterminer si le nombre de vos connexions est supérieur à quatre, utilisez la cmdlet PowerShell `get-SmbClientConfiguration` pour afficher les paramètres actuels du nombre de connexions.

#### <a name="solution"></a>Solution

Définissez le paramètre Windows par carte réseau pour SMB de sorte que le nombre total de canaux ne dépasse pas quatre. Par exemple, si vous avez deux cartes réseau, vous pouvez définir la valeur maximale par carte réseau sur deux à l’aide de la cmdlet PowerShell suivante : `Set-SmbClientConfiguration -ConnectionCountPerRssNetworkInterface 2`.

## <a name="very-high-latency-for-requests"></a>Latence très élevée pour les requêtes

### <a name="cause"></a>Cause

La machine virtuelle cliente pourrait se trouver dans une région différente de celle où se situe le partage de fichiers. La latence élevée peut également être due à la latence causée par le client ou le réseau.

### <a name="solution"></a>Solution

- Exécutez l’application à partir d’une machine virtuelle située dans la même région que le partage de fichiers.
- Pour votre compte de stockage, passez en revue les métriques de transaction **SuccessE2ELatency** et **SuccessServerLatency** via **Azure Monitor** dans le portail Azure. Une différence importante entre les valeurs des métriques SuccessE2ELatency et SuccessServerLatency est une indication de la latence probablement due au réseau ou au client. Consultez [Métriques de transaction](storage-files-monitoring-reference.md#transaction-metrics) dans les références de données Azure Files.

## <a name="client-unable-to-achieve-maximum-throughput-supported-by-the-network"></a>Impossible pour le client d’atteindre le débit maximal pris en charge par le réseau

### <a name="cause"></a>Cause
L’une des causes possibles est l’absence de prise en charge de SMB multicanal pour les partages de fichiers standard. Actuellement, Azure Files ne prend en charge qu’un seul canal. Par conséquent, il n’y a qu’une seule connexion de la machine virtuelle cliente au serveur. Cette connexion unique étant fixée à un seul processeur sur l’ordinateur virtuel client, le débit maximal réalisable à partir d’une machine virtuelle est lié par un seul cœur.

### <a name="workaround"></a>Solution de contournement

- Pour les partages de fichiers premium, [activez SMB Multichannel](files-smb-protocol.md#smb-multichannel).
- L’obtention d’une machine virtuelle avec un cœur plus grand pourrait contribuer à améliorer le débit.
- L’exécution de l’application cliente à partir de plusieurs machines virtuelles augmente le débit.
- Utilisez les API REST si c’est possible.
- Pour les partages de fichiers NFS, nconnect est disponible en préversion. Non recommandé pour les charges de travail de production.

## <a name="throughput-on-linux-clients-is-significantly-lower-than-that-of-windows-clients"></a>Le débit sur les clients Linux est sensiblement plus faible que sur des clients Windows

### <a name="cause"></a>Cause

Il s’agit d’un problème connu d’implémentation du client SMB sur Linux.

### <a name="workaround"></a>Solution de contournement

- Répartissez la charge sur plusieurs machines virtuelles.
- Sur la même machine virtuelle, utilisez plusieurs points de montage avec une option `nosharesock`, et répartissez la charge entre ces points de montage.
- Sur Linux, essayez d’effectuer le montage avec une option `nostrictsync` pour éviter de forcer un vidage SMB à chaque appel de `fsync`. Pour Azure Files, cette option n’interfère pas avec la cohérence des données, mais elle pourrait entraîner la présence de métadonnées de fichier obsolètes dans les listes de répertoires (commande `ls -l`). L’interrogation directe des métadonnées du fichier à l’aide de la commande `stat` retournera les métadonnées de fichier les plus récentes.

## <a name="high-latencies-for-metadata-heavy-workloads-involving-extensive-openclose-operations"></a>Latences élevées pour les charges de travail lourdes de métadonnées impliquant des opérations d’ouverture/de fermeture étendues

### <a name="cause"></a>Cause

Absence de prise en charge pour les baux de répertoire.

### <a name="workaround"></a>Solution de contournement

- Si possible, évitez d’ouvrir/de fermer le descripteur un nombre de fois excessif sur le même répertoire dans un laps de temps bref.
- Pour les machines virtuelles Linux, augmentez le délai d’expiration du cache du répertoire d’entrée en spécifiant `actimeo=<sec>` comme option de montage. Par défaut, le délai d’expiration est de 1 seconde. Ainsi, une valeur plus élevée, par exemple de 3 ou 5 secondes, peut être utile.
- Pour des machines virtuelles CentOS Linux ou Red Hat Enterprise Linux (RHEL), mettez à niveau le système vers CentOS Linux 8.2 ou RHEL 8.2. Pour les autres machines virtuelles Linux, mettez à niveau le noyau vers la version 5.0 ou une version ultérieure.

## <a name="low-iops-on-centos-linux-or-rhel"></a>Faible nombre d’IOPS sur CentOS Linux ou RHEL

### <a name="cause"></a>Cause

Une profondeur d’E/S supérieure à 1 n’est pas prise en charge sur CentOS Linux ou RHEL.

### <a name="workaround"></a>Solution de contournement

- Effectuez une mise à niveau vers CentOS Linux 8 ou RHEL 8.
- Passez à Ubuntu.

## <a name="slow-file-copying-to-and-from-azure-file-shares-in-linux"></a>Lenteur de copie des fichiers vers et depuis des partages de fichiers Azure dans Linux

En cas de lenteur de copie des fichiers, consultez la section « Lenteur de copie des fichiers vers et depuis des partages de fichiers Azure dans Linux » du [Guide de résolution des problèmes de Linux](storage-troubleshoot-linux-file-connection-problems.md#slow-file-copying-to-and-from-azure-files-in-linux).

## <a name="jittery-or-sawtooth-pattern-for-iops"></a>IOPS instables ou en dents de scie

### <a name="cause"></a>Cause

L’application cliente dépasse constamment les IOPS de base. Actuellement, il n’y a pas de lissage de la charge de demande côté service. Si le client dépasse les IOPS de base, il est limité par le service. En raison de la limitation, le client peut être confronté à des IOPS instables ou en dents de scie. Dans ce cas, les IOPS moyennes que le client obtient peuvent être inférieures aux IOPS de base.

### <a name="workaround"></a>Solution de contournement
- Réduisez la charge de demande de l’application cliente afin que le partage ne soit pas limité.
- Augmentez le quota du partage afin que celui-ci ne soit pas limité.

## <a name="excessive-directoryopendirectoryclose-calls"></a>Appels DirectoryOpen/DirectoryClose excessifs

### <a name="cause"></a>Cause

Si les appels **DirectoryOpen/DirectoryClose** comptent parmi les principaux appels d’API et que vous ne vous attendez pas à ce que le client fasse autant d’appels, le problème pourrait être dû au logiciel antivirus installé sur la machine virtuelle cliente Azure.

### <a name="workaround"></a>Solution de contournement

- Un correctif pour ce problème est disponible dans la [Mise à jour d’avril de la plateforme pour Windows](https://support.microsoft.com/help/4052623/update-for-windows-defender-antimalware-platform).

## <a name="file-creation-is-slower-than-expected"></a>La création de fichiers est plus lente que prévu

### <a name="cause"></a>Cause

Les performances des charges de travail qui s’appuient sur la création d’un grand nombre de fichiers ne sont pas très différentes entre des partages de fichiers premium et des partages de fichiers standard.

### <a name="workaround"></a>Solution de contournement

- Aucun.

## <a name="slow-performance-from-windows-81-or-server-2012-r2"></a>Niveau de performance ralenti à partir de Windows 8.1 ou de Server 2012 R2

### <a name="cause"></a>Cause

Latence supérieure à celle attendue de l’accès aux partages de fichiers Azure pour des charges de travail nécessitant beaucoup d’E/S.

### <a name="workaround"></a>Solution de contournement

- Installez le [correctif logiciel](https://support.microsoft.com/help/3114025/slow-performance-when-you-access-azure-files-storage-from-windows-8-1) disponible.

## <a name="smb-multichannel-is-not-being-triggered"></a>La fonctionnalité SMB Multichannel n’est pas déclenchée.

### <a name="cause"></a>Cause

Modifications récentes apportées aux paramètres de configuration de la fonctionnalité SMB Multichannel sans remontage.

### <a name="solution"></a>Solution
 
-   Après toute modification apportée aux paramètres de configuration de la fonctionnalité SMB Multichannel de compte ou de client de SMB Windows, vous devez démonter le partage, attendre 60 secondes, puis remonter le partage pour déclencher la fonctionnalitél.
-   Pour le système d’exploitation du client Windows, générez une charge d’E/S avec une profondeur de file d’attente élevée, par exemple de 8, en copiant un fichier pour déclencher la fonctionnalité SMB Multichannel.  Pour le système d’exploitation du serveur, la fonctionnalité SMB Multichannel est déclenchée avec une profondeur de file d’attente de 1, c’est-à-dire dès que vous démarrez une E/S sur le partage.

## <a name="high-latency-on-web-sites-hosted-on-file-shares"></a>Latence élevée des sites web hébergés sur des partages de fichiers 

### <a name="cause"></a>Cause  

Un nombre élevé de notifications de modification de fichier sur des partages de fichiers peut entraîner des latences importantes. Cela se produit généralement avec des sites web hébergés sur des partages de fichiers avec une structure de répertoires imbriqués profonde. Un scénario classique est l’application web hébergée par IIS où la notification de modification de fichier est configurée pour chaque répertoire dans la configuration par défaut. Chaque modification ([ReadDirectoryChangesW](/windows/win32/api/winbase/nf-winbase-readdirectorychangesw)) sur le partage pour lequel le client est inscrit envoie une notification de modification du service de fichiers au client, ce qui mobilise des ressources du système, et le problème empire avec le nombre de modifications. Cela peut entraîner une limitation du partage et, par conséquent, une latence plus élevée côté client. 

Pour vérifier cela, vous pouvez utiliser les métriques Azure dans le portail : 

1. Dans le portail Azure, accédez à votre compte de stockage. 
1. Dans le menu de gauche, sous Supervision, sélectionnez Métriques. 
1. Sélectionnez Fichier comme espace de noms de métrique pour votre étendue de compte de stockage. 
1. Sélectionnez Transactions comme métrique. 
1. Ajoutez un filtre pour ResponseType et vérifiez si toutes les demandes ont un code de réponse SuccessWithThrottling (pour SMB ou NFS) ou ClientThrottlingError (pour REST).

### <a name="solution"></a>Solution 

- Si la notification de modification de fichier n’est pas utilisée, désactivez-la (par défaut).
    - [Désactivez la notification de modification de fichier](https://support.microsoft.com/help/911272/fix-asp-net-2-0-connected-applications-on-a-web-site-may-appear-to-sto) en mettant à jour FCNMode. 
    - Mettez à jour l’intervalle d’interrogation du processus Worker IIS (W3WP) à 0 en définissant `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\W3SVC\Parameters\ConfigPollMilliSeconds ` dans votre registre, puis redémarrez le processus W3WP. Pour en savoir plus sur ce paramètre, consultez [Clés de Registre qui s’appliquent au processus de travail IIS (W3WP)](/troubleshoot/iis/use-registry-keys#registry-keys-that-apply-to-iis-worker-process-w3wp).
- Augmentez la fréquence de l’intervalle d’interrogation de notification de modification de fichier pour réduire le volume.
    - Mettez à jour l’intervalle d’interrogation du processus Worker W3WP en le définissant sur une valeur plus élevée (par exemple, 10 ou 30 minutes) en fonction de vos besoins. Définissez `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\W3SVC\Parameters\ConfigPollMilliSeconds ` [dans votre registre](/troubleshoot/iis/use-registry-keys#registry-keys-that-apply-to-iis-worker-process-w3wp), puis redémarrez le processus W3WP.
- Si le répertoire physique mappé de votre site web comporte une structure de répertoires imbriqués, vous pouvez essayer de limiter l’étendue de la notification de modification de fichier pour réduire le volume des notifications. Par défaut, IIS utilise la configuration des fichiers Web.config dans le répertoire physique auquel le répertoire virtuel est mappé, ainsi que dans tous les répertoires enfants de ce répertoire physique. Si vous ne souhaitez pas utiliser de fichiers Web.config dans les répertoires enfants, spécifiez la valeur false pour l’attribut allowSubDirConfig sur le répertoire virtuel. Pour plus d’informations, cliquez [ici](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#virtual-directories). 
    - Définissez le paramètre « allowSubDirConfig » du répertoire virtuel IIS dans Web.config sur la valeur *false* afin d'exclure de l'étendue les répertoires enfants physiques mappés.  

## <a name="how-to-create-an-alert-if-a-file-share-is-throttled"></a>Comment créer une alerte en cas de limitation d’un partage de fichiers

1. Accédez à votre **compte de stockage** dans le **portail Azure**.
2. Dans la section **Supervision**, cliquez sur **Alertes**, puis sur **+ Nouvelle règle d’alerte**.
3. Cliquez sur **Modifier une ressource**, sélectionnez le **type de ressource fichier** pour le compte de stockage, puis cliquez sur **Terminé**. Par exemple, si le nom du compte de stockage est `contoso`, sélectionnez la ressource `contoso/file`.
4. Cliquez sur **Ajouter une condition** pour ajouter une condition.
5. Vous verrez une liste de signaux pris en charge pour le compte de stockage ; sélectionnez la métrique **Transactions**.
6. Dans le panneau **Configurer la logique de signal**, cliquez sur la liste déroulante **Nom de la dimension**, puis sélectionnez **Type de réponse**.
7. Cliquez sur la liste déroulante **Valeurs de dimension** et sélectionnez les types de réponses appropriés pour votre partage de fichiers.

    Pour les partages de fichiers standard, sélectionnez les types de réponse suivants :

    - SuccessWithThrottling
    - SuccessWithShareIopsThrottling
    - ClientShareIopsThrottlingError

    Pour les partages de fichiers premium, sélectionnez les types de réponse suivants :

    - SuccessWithShareEgressThrottling
    - SuccessWithShareIngressThrottling
    - SuccessWithShareIopsThrottling
    - ClientShareEgressThrottlingError
    - ClientShareIngressThrottlingError
    - ClientShareIopsThrottlingError

   > [!NOTE]
   > Si les types de réponse ne sont pas répertoriés dans la liste déroulante **Valeurs de dimension**, cela signifie que la ressource n’a pas été limitée. Pour ajouter les valeurs de dimension, à côté de la liste déroulante **Valeurs de dimension**, sélectionnez **Ajouter une valeur personnalisée**, entrez le type de réponse (par exemple, **SuccessWithThrottling**), sélectionnez **OK**, puis répétez ces étapes pour ajouter tous les types de réponses pertinents à votre partage de fichiers.

8. Pour les **partages de fichiers Premium**, cliquez sur la liste déroulante **Nom de la dimension** et sélectionnez **Partage de fichiers**. Pour les **partages de fichiers standard**, passez à l'**étape 10**.

   > [!NOTE]
   > Si le partage de fichiers est un partage standard, la dimension **Partage de fichiers** ne répertorie pas le(s) partage(s) de fichiers, car les métriques par partage ne sont pas disponibles pour les partages de fichiers standard. Les alertes de limitation pour les partages de fichiers standard sont déclenchées si un partage de fichiers au sein du compte de stockage est limité et l’alerte n’identifiera pas quel partage de fichiers a été limité. Étant donné que les métriques par partage ne sont pas disponibles pour les partages de fichiers standard, il est recommandé de disposer d’un partage de fichiers par compte de stockage.

9. Cliquez sur la liste déroulante **Valeurs de dimension**, puis sélectionnez le ou les partages de fichiers pour lesquels vous souhaitez recevoir une alerte.
10. Définissez les **paramètres d’alerte** (valeur de seuil, opérateur, granularité d’agrégation et fréquence), puis cliquez sur **Terminé**.

    > [!TIP]
    > Si vous utilisez un seuil statique, le graphique des métriques peut vous aider à déterminer une valeur seuil raisonnable si le partage de fichiers est actuellement limité. Si vous utilisez un seuil dynamique, le graphique des métriques affiche les seuils calculés en fonction des données récentes.

11. Cliquez sur **Ajouter un groupe d’actions** pour ajouter un **groupe d’actions** (e-mail, SMS, etc.) à l’alerte, soit en sélectionnant un groupe d’actions existant, soit en créant un nouveau groupe d’actions.
12. Renseignez les **Détails de l’alerte**, par exemple le **Nom de la règle d’alerte**, la **Description** et la **Gravité**.
13. Cliquez sur **Créer une règle d’alerte** pour créer l’alerte.

Pour en savoir plus sur la configuration des alertes dans Azure Monitor, consultez [Vue d’ensemble des alertes dans Microsoft Azure](../../azure-monitor/alerts/alerts-overview.md).

## <a name="how-to-create-alerts-if-a-premium-file-share-is-trending-toward-being-throttled"></a>Comment créer des alertes si un partage de fichiers premium tend à être limité

1. Dans le portail Azure, accédez à votre compte de stockage.
2. Dans la section **Supervision**, sélectionnez **Alertes**, puis **Nouvelle règle d’alerte**.
3. Sélectionnez **Modifier une ressource**, le **Type de ressource fichier** pour le compte de stockage, puis **Terminé**. Par exemple, si le nom du compte de stockage est *contoso*, sélectionnez la ressource contoso/file.
4. Choisissez **Sélectionner une condition** pour ajouter une condition.
5. Dans la liste de signaux pris en charge pour le compte de stockage, sélectionnez la métrique **Sortie**.

   > [!NOTE]
   > Vous devez créer trois alertes distinctes pour être alerté lorsque les valeurs d’entrée, de sortie ou de transaction dépassent les seuils que vous définissez. Cela est dû au fait qu’une alerte n’est déclenchée que si toutes les conditions sont réunies. Par exemple, si vous placez toutes les conditions dans une seule alerte, vous ne recevrez d’alerte que si les valeurs d’entrée, de sortie et transactions dépassent toutes leur seuil.

6. Faites défiler vers le bas. Dans la liste déroulante **Nom de la dimension**, sélectionnez **Partage de fichiers**.
7. Dans la liste déroulante **Valeurs de la dimension**, sélectionnez le ou les partages de fichiers pour lesquels vous souhaitez recevoir une alerte.
8. Définissez les paramètres d’alerte en sélectionnant des valeurs dans les listes déroulantes **Opérateur**, **Valeur de seuil**, **Granularité d’agrégation** et **Fréquence d’évaluation**, puis sélectionnez **Terminé**.

   Les métriques de sortie, d’entrée et de transactions sont exprimées par minute, même si vous avez configuré la sortie, l’entrée et les E/S par seconde. Par exemple, si votre sortie configurée est de 90&nbsp;Mio/s et que vous souhaitez que votre seuil soit à 80&nbsp;% de la sortie configurée, sélectionnez les paramètres d’alerte suivants : 
   - Pour **Valeur de seuil** : *75497472* 
   - Pour **Opérateur** : *supérieur ou égal à*
   - Pour **Type d’agrégation** : *moyenne*
   
   Selon le bruit souhaité pour votre alerte, vous pouvez également sélectionner des valeurs pour la **Granularité d’agrégation** et la **Fréquence d’évaluation**. Par exemple, si vous souhaitez que votre alerte examine l’entrée moyenne sur une période de 1 heure, et que votre règle d’alerte soit exécutée toutes les heures, sélectionnez ce qui suit :
   - Pour **Granularité d’agrégation** : *1 heure*
   - Pour **Fréquence d’évaluation** : *1 heure*

9. Choisissez **Ajouter un groupe d’actions**, puis ajoutez un groupe d’actions (par exemple, E-mail ou SMS) à l’alerte en sélectionnant un groupe d’actions existant ou en en créant un.
10. Entrez les détails de l’alerte, tels que le **Nom de la règle d’alerte**, la **Description** et la **Gravité**.
11. Sélectionnez **Créer une règle d’alerte** pour créer l’alerte.

    > [!NOTE]
    > - Pour recevoir une notification quand votre partage de fichiers premium est sur le point d’être limité *en raison de l’entrée configurée*, suivez les instructions précédentes, mais avec la modification suivante :
    >    - À l’étape 5, sélectionnez la métrique **Entrée** au lieu de la métrique **Sortie**.
    >
    > - Pour recevoir une notification quand votre partage de fichiers premium est sur le point d’être limité *en raison des IOPS configurées*, suivez les instructions précédentes, mais avec les modifications suivantes :
    >    - À l’étape 5, sélectionnez la métrique **Transactions** au lieu de la métrique **Sortie**.
    >    - À l’étape 10, la seule option pour le **Type d’agrégation** est *Totale*. Par conséquent, la valeur de seuil dépend de la granularité d’agrégation sélectionnée. Par exemple, si vous souhaitez que votre seuil soit de 80&nbsp;pour cent des IOPS de base configurées et que vous sélectionnez *1 heure* comme **Granularité d’agrégation**, votre **Valeur de seuil** correspond à vos IOPS de base (en octets) &times;&nbsp;0,8 &times;&nbsp;3600. 

Pour en savoir plus sur la configuration des alertes dans Azure Monitor, consultez [Vue d’ensemble des alertes dans Microsoft Azure](../../azure-monitor/alerts/alerts-overview.md).

## <a name="see-also"></a>Voir aussi
- [Résoudre les problèmes d’Azure Files dans Windows](storage-troubleshoot-windows-file-connection-problems.md)  
- [Résoudre les problèmes d’Azure Files dans Linux](storage-troubleshoot-linux-file-connection-problems.md)  
- [FAQ sur les fichiers Azure](storage-files-faq.md)
