---
title: Historique de publication des mises en production de l'agent de protection par mot de passe - Azure Active Directory
description: Historique de publication des versions de documents et des changements de comportement
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: article
ms.date: 06/04/2021
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2aa4c7625637c84404b5ed7074c078717127ee1f
ms.sourcegitcommit: 57b7356981803f933cbf75e2d5285db73383947f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2021
ms.locfileid: "129545356"
---
# <a name="azure-ad-password-protection-agent-version-history"></a>Historique des versions de l’agent de protection par mot de passe Azure AD

## <a name="121760"></a>1.2.176.0

Date de publication : 4 juin 2021

* Correctifs mineurs aux problèmes qui ont empêché le proxy et les agents DC de s’exécuter correctement dans certains environnements.

## <a name="121720"></a>1.2.172.0

Date de publication : 22 février 2021

Près de deux ans se sont écoulés depuis la mise en production des versions en disponibilité générale des agents locaux de protection par mot de passe Azure AD. Une nouvelle mise à jour est désormais disponible. Consultez les descriptions des modifications ci-dessous. Merci à tous ceux qui nous ont fait part de leurs commentaires sur le produit. 

* Les logiciels de l’agent DC et de l’agent proxy requièrent désormais tous deux que .NET 4.7.2 soit installé.
  * Si ce n'est pas le cas, téléchargez et exécutez le programme d’installation disponible sur la page [Programme d’installation hors connexion de .NET Framework 4.7.2 pour Windows](https://support.microsoft.com/topic/microsoft-net-framework-4-7-2-offline-installer-for-windows-05a72734-2127-a15d-50cf-daf56d5faec2).
* Le module PowerShell AzureADPasswordProtection est maintenant également installé par le logiciel de l’agent DC.
* Deux nouvelles cmdlets PowerShell liées à l’intégrité ont été ajoutées : Test-AzureADPasswordProtectionDCAgent et Test-AzureADPasswordProtectionProxy.
* La dll de filtre de mot de passe de l’agent DC AzureADPasswordProtection se charge et s’exécute sur des ordinateurs sur lesquels lsass.exe est configuré pour s’exécuter en mode PPL.
* Résolution d’un bogue dans l’algorithme de mot de passe qui permettait l’acceptation incorrecte de mots de passe interdits de moins de cinq caractères.
  * Ce bogue n’est applicable que si votre politique locale de longueur minimale de mot de passe AD a d’abord été configurée pour autoriser des mots de passe de moins de cinq caractères.
* Autres correctifs de bogues mineurs.

Les nouveaux programmes d’installation mettent automatiquement à niveau des versions antérieures du logiciel. Si vous avez installé les logiciels de l’agent DC et de l’agent proxy sur un seul ordinateur (recommandé uniquement pour des environnements de test), vous devez les mettre à niveau en même temps.

Il est possible d’exécuter des versions plus anciennes et plus récentes des logiciels de l’agent DC et de l’agent proxy au sein d’un domaine ou d’une forêt. Toutefois, la meilleure pratique que nous recommandons est de mettre à niveau tous les agents vers la dernière version. Les mises à niveau d’agent peuvent être effectuées dans n’importe quel ordre. Des nouveaux agents DC peuvent communiquer via des agents proxy plus anciens, et des agents DC plus anciens peuvent communiquer via des agents proxy plus récents.

## <a name="121250"></a>1.2.125.0

Date de sortie : 2 mars 2019

* Correction de fautes de frappe mineures dans les messages du journal des événements
* Mise à jour du CLUF vers la version finale mise à la disposition générale

> [!NOTE]
> La build 1.2.125.0 correspond à la version mise à la disposition générale. Merci encore à tous ceux qui nous ont fait part de leurs commentaires sur le produit !

## <a name="121160"></a>1.2.116.0

Date de sortie : 3 mars 2019

* Les cmdlets Get-AzureADPasswordProtectionProxy et Get-AzureADPasswordProtectionDCAgent indiquent désormais la version du logiciel et le locataire Azure actuel avec les limitations suivantes :
  * La version du logiciel et les données du locataire Azure sont uniquement disponibles pour les proxies et les agents DC exécutant la version 1.2.116.0 ou ultérieure.
  * Les données du locataire Azure peuvent ne peuvent pas être rapportées tant qu'une réinscription (ou un renouvellement) du proxy ou de la forêt n'a pas eu lieu.
* Le service proxy nécessite maintenant l'installation de .NET 4.7.
  * Si ce n'est pas le cas, téléchargez et exécutez le programme d’installation disponible sur la page [Programme d’installation hors connexion de .NET Framework 4.7 pour Windows](https://support.microsoft.com/help/3186497/the-net-framework-4-7-offline-installer-for-windows).
  * Sur les systèmes Server Core, il peut être nécessaire de transmettre l'indicateur /q au programme d'installation de .NET 4.7 pour que l'installation aboutisse.
* Le service proxy prend désormais en charge la mise à niveau automatique. La mise à niveau automatique utilise le service du programme de mise à jour de l’agent Microsoft Azure AD Connect, installé à côté du service proxy. La mise à niveau automatique est activée par défaut.
* La mise à niveau automatique peut être activée ou désactivée à l'aide de la cmdlet Set-AzureADPasswordProtectionProxyConfiguration. Le paramètre actuel peut être interrogé à l'aide de la cmdlet Get-AzureADPasswordProtectionProxyConfiguration.
* Le binaire du service d'agent DC a été renommé AzureADPasswordProtectionDCAgent.exe.
* Le binaire du service proxy a été renommé AzureADPasswordProtectionProxy.exe. Il peut être nécessaire de modifier les règles de pare-feu en conséquence si un pare-feu tiers est utilisé.
  * REMARQUE : si un fichier de configuration de proxy http était utilisé dans une installation proxy précédente, celui-ci doit être renommé (en remplaçant *proxyservice.exe.config* par *AzureADPasswordProtectionProxy.exe.config*) après cette mise à niveau.
* Tous les contrôles de fonctionnalité limités dans le temps ont été supprimés de l'agent DC.
* Correction de bogues mineurs et améliorations de la journalisation.

## <a name="12650"></a>1.2.65.0

Date de publication : 1er février 2019

Modifications :

* L’agent du contrôleur de domaine et le service proxy sont désormais pris en charge sur noyau du serveur. La configuration requise du système d’exploitation est identique à ce qu’elle était auparavant : Windows Server 2012 pour les agents du contrôleur de domaine et Windows Server 2012 R2 pour les proxys.
* Les cmdlets Register-AzureADPasswordProtectionProxy et Register-AzureADPasswordProtectionForest prennent désormais en charge les modes d’authentification Azure basés sur le code d’appareil.
* La cmdlet Get-AzureADPasswordProtectionDCAgent ignore les points de connexion de service altérés ou non valides. Cette modification résout le bogue qui avait pour effet que les contrôleurs de domaine apparaissaient parfois plusieurs fois dans la sortie.
* La cmdlet Get-AzureADPasswordProtectionSummaryReport ignore les points de connexion de service altérés ou non valides. Cette modification résout le bogue qui avait pour effet que les contrôleurs de domaine apparaissaient parfois plusieurs fois dans la sortie.
* Le module PowerShell du proxy est désormais inscrit à partir de %ProgramFiles%\WindowsPowerShell\Modules. La variable d’environnement PSModulePath de la machine n’est plus modifiée.
* Une nouvelle cmdlet Get-AzureADPasswordProtectionProxy a été ajoutée pour faciliter la découverte de proxys inscrits dans une forêt ou un domaine.
* L’agent du contrôleur de domaine utilise un nouveau dossier dans le partage sysvol pour la réplication des stratégies de mot de passe et d’autres fichiers.

   Ancien emplacement du dossier :

   `\\<domain>\sysvol\<domain fqdn>\Policies\{4A9AB66B-4365-4C2A-996C-58ED9927332D}`

   Nouvel emplacement du dossier :

   `\\<domain>\sysvol\<domain fqdn>\AzureADPasswordProtection`

   (Cette modification a été apportée pour éviter les avertissements « GPO orphelin » faux positifs.)

   > [!NOTE]
   > Aucun partage de données ou migration n’est effectué entre l’ancien dossier et le nouveau. Les versions antérieures de l’agent du contrôleur de domaine continueront à utiliser l’ancien emplacement jusqu’à leur mise à niveau vers cette version ou une version ultérieure. Une fois que tous les agents du contrôleur de domaine exécutent la version 1.2.65.0 ou version ultérieure, l’ancien dossier sysvol peut être supprimé manuellement.

* Désormais, le service proxy et l’agent du contrôleur de domaine détectent et suppriment les copies altérées de leurs points de connexion de service respectifs.
* Chaque agent du contrôleur de domaine supprime régulièrement les points de connexion de service altérés et périmés dans son domaine, pour les points de connexion tant de l’agent du contrôleur de domaine que du service proxy. Les points de connexion tant de l’agent du contrôleur de domaine que du service proxy sont considérés comme périmés si leur indicateur de pulsations date de plus de sept jours.
* L’agent du contrôleur de domaine renouvelle désormais le certificat de la forêt en fonction des besoins.
* Le service proxy renouvelle désormais le certificat du proxy en fonction des besoins.
* Mises à jour de l’algorithme de validation de mot de passe : la liste globale de mots de passe interdits et la liste spécifique du client de mots de passe interdits (si configurée) sont combinées avant les validations de mot de passe. Un mot de passe donné peut désormais être rejeté (échec ou audit uniquement) s’il contient des jetons provenant tant de la liste globale que de la liste spécifique du client. La documentation sur le journal des événements a été mise à jour pour refléter cela. Consultez [Surveiller la protection par mot de passe d’Azure AD](howto-password-ban-bad-on-premises-monitor.md).
* Correctifs de performances et de robustesse
* Amélioration de la journalisation

> [!WARNING]
> Fonctionnalité limitée dans le temps : le service de l’agent du contrôleur de domaine dans cette version (1.2.65.0) cessera de traiter les demandes de validation de mot de passe le 1e septembre 2019.  Les services de l’agent du contrôleur de domaine dans les versions antérieures (voir la liste ci-dessous) cesseront d’opérer le 1e juillet 2019. Le service de l’agent du contrôleur de domaine dans toutes les versions consignera des événements 10021 dans le journal des événements d’administration dans les deux mois précédant ces délais. Toutes les restrictions de limite de temps seront supprimées dans la version en disponibilité générale à venir. Le service de l’agent proxy n’est limité dans le temps dans aucune version, mais il doit toujours être mis à niveau vers la dernière version afin de tirer parti de tous les corrections de bogues et d’autres améliorations subséquentes.

## <a name="12250"></a>1.2.25.0

Date de publication : 1er novembre 2018

Correctifs :

* L'agent DC et le service proxy ne devraient plus échouer pour cause d'échecs des certificats de confiance.
* L'agent DC et le service proxy disposent de correctifs pour les ordinateurs conformes aux normes FIPS.
* Le service proxy fonctionnera désormais correctement dans un environnement réseau utilisant uniquement le protocole TLS 1.2.
* Correctifs mineurs de performances et de robustesse
* Amélioration de la journalisation

Modifications :

* Windows Server 2012 R2 est désormais le système d'exploitation minimum requis pour le service proxy. Windows Server 2012 reste le système d'exploitation minimum requis pour le service de l'agent DC.
* Le service proxy requiert désormais .NET version 4.6.2.
* L’algorithme de validation de mot de passe utilise une table de normalisation des caractères étendus. Cette modification peut entraîner le rejet de mots de passe qui étaient acceptés dans des versions précédentes.

## <a name="12100"></a>1.2.10.0

Date de publication : 17 août 2018

Correctifs :

* Register-AzureADPasswordProtectionProxy et Register-AzureADPasswordProtectionForest prennent désormais en charge l’authentification multifacteur
* Register-AzureADPasswordProtectionProxy nécessite un contrôleur de domaine WS2012 ou ultérieur dans le domaine afin d’éviter les erreurs de chiffrement.
* Le service de l’agent de contrôleur de domaine est plus fiable en ce qui concerne les demandes de nouvelle stratégie de mot de passe à Azure au démarrage.
* Le service de l’agent de contrôleur de domaine demande une nouvelle stratégie de mot de passe à Azure toutes les heures si nécessaire, mais il le fait désormais à compter d’une heure de début sélectionnée de façon aléatoire.
* Le service de l’agent de contrôleur de domaine ne provoque plus un délai illimité lors d’une nouvelle publication de contrôleur de domaine en cas d’installation sur un serveur avant sa promotion en tant que réplica.
* Le service de l’agent de contrôleur de domaine respecte désormais le paramètre de configuration « Activer la protection par mot de passe sur Windows Server Active Directory ».
* Les programmes d’installation du proxy et de l’agent de contrôleur de domaine prennent désormais en charge la mise à niveau sur place en cas de mise à niveau vers des versions ultérieures.

> [!WARNING]
> La mise à niveau sur place à partir de la version 1.1.10.3 n’est pas prise en charge et provoque une erreur d’installation. Pour mettre à niveau vers la version 1.2.10 ou ultérieure, vous devez d’abord désinstaller complètement le logiciel du service de proxy et d’agent de contrôleur de domaine, puis installer la nouvelle version à partir de zéro. La réinscription du service de proxy de protection par mot de passe Azure AD est nécessaire.  La réinscription de la forêt n’est pas nécessaire.

> [!NOTE]
> Les mises à niveau sur place du logiciel d’agent de contrôleur de domaine nécessitent un redémarrage.

* Le service de proxy et l’agent de contrôleur de domaine prennent désormais en charge l’exécution sur un serveur configuré pour utiliser uniquement des algorithmes compatibles FIPS.
* Correctifs mineurs de performances et de robustesse
* Amélioration de la journalisation

## <a name="11103"></a>1.1.10.3

Date de publication : 15 juin 2018

Préversion publique initiale

## <a name="next-steps"></a>Étapes suivantes

[Déployer la protection par mot de passe Azure AD](howto-password-ban-bad-on-premises-deploy.md)
