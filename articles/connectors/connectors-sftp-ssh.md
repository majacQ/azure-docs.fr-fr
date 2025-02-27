---
title: Se connecter à un serveur SFTP avec SSH
description: Automatiser les tâches qui supervisent, créent, gèrent, envoient et reçoivent des fichiers pour un serveur SFTP à l’aide de SSH et d’Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.reviewer: estfan, logicappspm, azla
ms.topic: conceptual
ms.date: 08/05/2021
tags: connectors
ms.openlocfilehash: 32638d71f2c700700be5eb1b2ad63f18969d82a0
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122525247"
---
# <a name="create-and-manage-sftp-files-using-ssh-and-azure-logic-apps"></a>Créer et gérer des fichiers SFTP à l’aide de SSH et d’Azure Logic Apps

Pour automatiser les tâches qui créent et gèrent des fichiers sur un serveur [Secure File Transfer Protocol (SFTP)](https://www.ssh.com/ssh/sftp/) à l’aide du protocole [Secure Shell (SSH)](https://www.ssh.com/ssh/protocol/), vous pouvez créer des workflows d’intégration automatisés à l’aide d’Azure Logic Apps et du connecteur SFTP-SSH. SFTP est un protocole réseau qui fournit un accès aux fichiers, le transfert de fichiers et la gestion des fichiers sur n’importe quel flux de données fiable.

Voici quelques exemples de tâches que vous pouvez automatiser :

* Superviser à quel moment des fichiers sont ajoutés ou changés.
* Obtenir, créer, copier, renommer, mettre à jour, lister et supprimer des fichiers.
* Créez des dossiers.
* Obtenir les métadonnées et le contenu des fichiers.
* Extraire des archives dans des dossiers.

Dans votre workflow, vous pouvez utiliser un déclencheur qui supervise les événements sur votre serveur SFTP et met la sortie à la disposition d’autres actions. Vous pouvez ensuite utiliser des actions pour effectuer diverses tâches sur votre serveur SFTP. Vous pouvez également inclure d’autres actions qui utilisent la sortie des actions SFTP-SSH. Par exemple, si vous récupérez régulièrement des fichiers de votre serveur SFTP, vous pouvez envoyer des alertes par e-mail au sujet de ces fichiers et de leur contenu en utilisant le connecteur Office 365 Outlook ou le connecteur Outlook.com. Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md)

Pour connaître les différences entre le connecteur SFTP-SSH et le connecteur SFTP, consultez la section [Comparer SFTP-SSH et SFTP](#comparison) plus loin dans cette rubrique.

## <a name="limits"></a>limites

* Actuellement, le connecteur SFTP-SSH ne prend pas en charge les serveurs SFTP suivants :

  * IBM DataPower
  * MessageWay
  * OpenText Secure MFT
  * OpenText GXS

* Les actions SFTP-SSH prenant en charge la [segmentation](../logic-apps/logic-apps-handle-large-messages.md) peuvent gérer des fichiers jusqu’à 1 Go, tandis que les actions SFTP-SSH ne prenant pas en charge la segmentation peuvent gérer des fichiers jusqu’à 50 Mo. La taille de bloc par défaut est 15 Mo. Toutefois, cette taille peut changer de façon dynamique, en commençant à 5 Mo et augmentant progressivement jusqu’à la valeur maximale de 50 Mo. Le dimensionnement dynamique est basé sur des facteurs tels que la latence du réseau, le temps de réponse du serveur, etc.

  > [!NOTE]
  > Pour les applications logiques utilisées dans un [environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la version de ce connecteur avec l’étiquette ISE nécessite des blocs pour utiliser les [limites de messages de l’ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits) à la place.

  Vous pouvez passer outre ce comportement adaptatif lorsque vous [spécifiez une taille de bloc constante](#change-chunk-size) à utiliser à la place. Cette taille peut être comprise entre 5 Mo et 50 Mo. Par exemple, supposons que vous disposez d’un fichier de 45 Mo et d’un réseau pouvant prendre en charge cette taille de fichier sans latence. La segmentation adaptative génère plusieurs appels plutôt qu’un seul. Pour réduire le nombre d’appels, vous pouvez essayer de définir une taille de bloc de 50 Mo. Dans un autre scénario, si votre application logique a un délai d’expiration, par exemple lorsque vous utilisez des blocs de 15 Mo, vous pouvez essayer de réduire la taille à 5 Mo.

  La taille de bloc est associée à une connexion. Cet attribut signifie que vous pouvez utiliser la même connexion pour les actions qui prennent en charge la segmentation et les actions qui ne la prennent pas en charge. Dans ce cas, la taille de segment pour les actions ne prenant pas en charge les plages de segmentation est comprise entre 5 Mo et 50 Mo. Ce tableau indique les actions SFTP-SSH prenant en charge la segmentation :

  | Action | Prise en charge de la segmentation | Remplacement de la prise en charge de la taille de bloc |
  |--------|------------------|-----------------------------|
  | **Copier un fichier** | Non | Non applicable |
  | **Créer un fichier** | Oui | Oui |
  | **Créer un dossier** | Non applicable | Non applicable |
  | **Supprimer un fichier** | Non applicable | Non applicable |
  | **Extraire une archive dans un dossier** | Non applicable | Non applicable |
  | **Obtenir le contenu d’un fichier** | Oui | Oui |
  | **Obtenir le contenu d’un fichier à l’aide du chemin** | Oui | Oui |
  | **Obtenir les métadonnées d’un fichier** | Non applicable | Non applicable |
  | **Obtenir les métadonnées d’un fichier à l’aide du chemin** | Non applicable | Non applicable |
  | **Répertorier les fichiers dans un dossier** | Non applicable | Non applicable |
  | **Renommer le fichier** | Non applicable | Non applicable |
  | **Mettre à jour un fichier** | Non | Non applicable |
  ||||

* Les déclencheurs SFTP-SSH ne prennent pas en charge la segmentation des messages. Quand ils demandent le contenu des fichiers, les déclencheurs sélectionnent uniquement les fichiers dont la taille est inférieure ou égale à 15 Mo. Pour obtenir des fichiers supérieurs à 15 Mo, suivez plutôt ce modèle :

  1. Utilisez un déclencheur SFTP-SSH qui ne retourne que les propriétés du fichier. Ces déclencheurs ont des noms qui incluent la description **(propriétés uniquement)** .

  1. Suivez le déclencheur avec l’action SFTP-SSH **Obtenir le contenu du fichier**. Cette action lit le fichier complet et utilise implicitement la segmentation des messages.

<a name="comparison"></a>

## <a name="compare-sftp-ssh-versus-sftp"></a>Comparer SFTP/SSH par rapport à SFTP

La liste suivante décrit les principales fonctionnalités SFTP-SSH qui diffèrent du connecteur SFTP :

* Il utilise la [bibliothèque SSH.NET](https://github.com/sshnet/SSH.NET), qui est une bibliothèque Secure Shell (SSH) open source prenant en charge .NET.

* Fournit l’action **Créer un dossier**, qui crée un dossier au niveau du chemin spécifié sur le serveur SFTP.

* Fournit l’action **Renommer le fichier**, qui renomme un fichier sur le serveur SFTP.

* Met en cache la connexion au serveur SFTP *pendant jusqu’à 1 heure*. Cette fonctionnalité permet d’améliorer les performances et de réduire la fréquence à laquelle le connecteur tente de se connecter au serveur. Pour définir la durée de ce comportement de mise en cache, modifiez la propriété [**ClientAliveInterval**](https://man.openbsd.org/sshd_config#ClientAliveInterval) dans la configuration SSH sur votre serveur SFTP.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Les informations d’identification de compte et l’adresse de votre serveur SFTP, afin que votre workflow puisse accéder à votre compte SFTP. Vous devez également accéder à une clé privée SSH et au mot clé privé SSH. Pour charger des fichiers volumineux à l’aide de la segmentation, vous devez disposer d’un accès en lecture et en écriture au dossier racine de votre serveur SFTP. Sinon, vous obtiendrez une erreur « 401 non autorisé ».

  Le connecteur SFTP-SSH prend en charge l’authentification par clé privée et l’authentification par mot de passe. Cependant, le connecteur SFTP-SSH prend en charge *uniquement* ces formats de clés privées, algorithmes de chiffrement, empreintes digitales et algorithmes d’échange de clés :

  * **Formats de clé privée** : les clés RSA (Rivest Shamir Adleman) et DSA (Digital Signature Algorithm) aux formats OpenSSH et ssh.com. Si votre clé privée est au format de fichier PuTTY (.ppk), commencez par [convertir la clé au format de fichier OpenSSH (.pem)](#convert-to-openssh).
  * **Algorithmes de chiffrement** : DES-EDE3-CBC, DES-EDE3-CFB, DES-CBC, AES-128-CBC, AES-192-CBC, and AES-256-CBC
  * **Empreinte digitale** : MD5
  * **Algorithmes d’échange de clé** : curve25519-sha256, curve25519-sha256@libssh.org, ecdh-sha2-nistp256, ecdh-sha2-nistp384, ecdh-sha2-nistp521, diffie-hellman-group-exchange-sha256, diffie-hellman-group-exchange-sha1, diffie-hellman-group16-sha512, diffie-hellman-group14-sha256, diffie-hellman-group14-sha1, and diffie-hellman-group1-sha1

  Après avoir ajouté une action ou un déclencheur SFTP-SSH à votre workflow, vous devez fournir les informations de connexion pour votre serveur SFTP. Quand vous fournissez votre clé privée SSH pour cette connexion, ***vous ne devez pas entrer ou modifier manuellement la clé** _, car ceci pourrait entraîner l’échec de la connexion. Veillez plutôt à _*_copier la clé_*_ à partir de votre fichier de clé privée SSH, puis à la _ *_coller_** dans les informations de connexion. Pour plus d’informations, consultez la section [Se connecter à SFTP avec SSH](#connect) plus loin dans cet article.

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* Le workflow d’application logique où vous souhaitez accéder à votre compte SFTP. Pour démarrer avec un déclencheur SFTP-SSH, [créez un workflow d’application logique vide](../logic-apps/quickstart-create-first-logic-app-workflow.md). Pour utiliser une action SFTP-SSH, démarrez votre workflow avec un autre déclencheur, par exemple, le déclencheur **Périodicité**.

## <a name="how-sftp-ssh-triggers-work"></a>Fonctionnement des déclencheurs SFTP-SSH

<a name="polling-behavior"></a>

### <a name="polling-behavior"></a>Comportement d’interrogation

Les déclencheurs SFTP/SSH interrogent le système de fichiers SFTP et recherchent tout fichier modifié depuis la dernière interrogation. Certains outils permettent de conserver l’horodatage lorsque les fichiers sont modifiés. Dans ce cas, vous devez désactiver cette fonctionnalité pour permettre l’exécution du déclencheur. Voici quelques paramètres communs :

| Client SFTP | Action |
|-------------|--------|
| Winscp | Accédez à **Options** > **Préférences** > **Transférer** > **Modifier** > **Conserver l’horodatage** > **Désactiver** |
| FileZilla | Accédez à **Transférer** > **Conserver les horodatages des fichiers transférés** > **Désactiver** |
|||

Quand un déclencheur détecte un nouveau fichier, il vérifie que le nouveau fichier est complet et non partiellement écrit. Par exemple, un fichier peut être en cours de modification lorsque le déclencheur vérifie le serveur de fichiers. Pour éviter de retourner un fichier partiellement écrit, le déclencheur note l’horodatage du fichier qui comporte des modifications récentes, mais ne retourne pas immédiatement ce fichier. Le déclencheur retourne le fichier uniquement lors d’une nouvelle interrogation du serveur. Parfois, ce comportement peut entraîner un retard correspondant à jusqu’à deux fois l’intervalle d’interrogation du déclencheur.

<a name="trigger-recurrence-shift-drift"></a>

### <a name="trigger-recurrence-shift-and-drift"></a>Décalage et dérive de la périodicité du déclencheur

Les déclencheurs basés sur la connexion, où vous devez d’abord créer une connexion, par exemple le déclencheur SFTP-SSH, diffèrent des déclencheurs intégrés qui s’exécutent en mode natif dans Azure Logic Apps, tels que le [déclencheur de récurrence](../connectors/connectors-native-recurrence.md). Dans les déclencheurs récurrents basés sur la connexion, la planification de la périodicité n'est pas le seul pilote à contrôler l'exécution, et le fuseau horaire détermine uniquement l'heure de début initiale. Les exécutions suivantes dépendent de la planification de la périodicité, de la dernière exécution du déclencheur *et* d’autres facteurs qui peuvent décaler les heures d’exécution ou produire un comportement inattendu. Par exemple, un comportement inattendu peut inclure de ne pas respecter la planification spécifiée lors des passages à l’heure d’été et à l’heure d’hiver. Pour que l’heure fixée pour la périodicité ne change pas au moment du passage à l’heure d’été, ajustez la périodicité manuellement. De cette façon, votre workflow continue à s’exécuter à l’heure prévue. Sinon, l'heure de début est avancée d'une heure lors du passage à l'heure d'été et reculée d'une heure lors du passage à l'heure d'hiver. Pour plus d’informations, consultez [Périodicité des déclencheurs basés sur la connexion](../connectors/apis-list.md#recurrence-for-connection-based-triggers).

<a name="convert-to-openssh"></a>

## <a name="convert-putty-based-key-to-openssh"></a>Convertir une clé PuTTY au format OpenSSH

Le format PuTTy et le format OpenSSH utilisent des extensions de nom de fichier différentes. Le format PuTTy utilise l’extension de nom de fichier .ppk (ou PuTTY Private Key). Le format OpenSSH utilise l’extension de nom de fichier .pem (ou Privacy Enhanced Mail). Si votre clé privée est au format PuTTy et que vous devez utiliser le format OpenSSH, commencez par convertir la clé au format OpenSSH en procédant comme suit :

### <a name="unix-based-os"></a>Système d’exploitation UNIX

1. Si vous n’avez pas déjà installé les outils PuTTY sur votre système, faites-le maintenant, par exemple :

   `sudo apt-get install -y putty`

1. Exécutez cette commande, qui crée un fichier que vous pouvez utiliser avec le connecteur SFTP-SSH :

   `puttygen <path-to-private-key-file-in-PuTTY-format> -O private-openssh -o <path-to-private-key-file-in-OpenSSH-format>`

   Par exemple :

   `puttygen /tmp/sftp/my-private-key-putty.ppk -O private-openssh -o /tmp/sftp/my-private-key-openssh.pem`

### <a name="windows-os"></a>Système d’exploitation Windows

1. Si ce n’est déjà fait, [téléchargez la dernière version du générateur de clé PuTTY (puttygen.exe)](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html), puis lancez le générateur.

1. Sur l’écran suivant, sélectionnez **Load**.

   ![Sélectionner « Load »](./media/connectors-sftp-ssh/puttygen-load.png)

1. Accédez à votre fichier de clé privée au format PuTTY, puis sélectionnez **Open**.

1. Dans le menu **Conversions**, sélectionnez **Export OpenSSH key**.

   ![Sélectionner « Export OpenSSH key »](./media/connectors-sftp-ssh/export-openssh-key.png)

1. Enregistrez le fichier de clé privée avec l’extension `.pem`.

## <a name="considerations"></a>Considérations

Cette section décrit les considérations à prendre en compte pour utiliser les déclencheurs et les actions de ce connecteur.

<a name="different-folders-trigger-processing-file-storage"></a>

### <a name="use-different-sftp-folders-for-file-upload-and-processing"></a>Utilisation de différents dossiers SFTP pour le chargement et le traitement des fichiers

Sur votre serveur SFTP, utilisez des dossiers distincts pour stocker les fichiers chargés et pour que le déclencheur supervise ces fichiers en vue de leur traitement. Dans le cas contraire, le déclencheur ne se déclenche pas et se comporte de manière imprévisible, par exemple en ignorant un nombre aléatoire de fichiers traités. Toutefois, cette exigence signifie que vous avez besoin d’un moyen de déplacer des fichiers entre ces dossiers. 

Si ce problème de déclencheur se produit, supprimez les fichiers du dossier supervisé par le déclencheur et utilisez un autre dossier pour stocker les fichiers chargés.

<a name="create-file"></a>

### <a name="create-file"></a>Créer un fichier

Pour créer un fichier sur votre serveur SFTP, vous pouvez utiliser l’action SFTP-SSH **Créer un fichier**. Lorsque cette action crée le fichier, le service Logic Apps appelle également automatiquement votre serveur SFTP pour récupérer les métadonnées associées. Toutefois, si vous déplacez le fichier qui vient d’être créé avant que le service Logic Apps ne puisse effectuer l’appel pour obtenir les métadonnées, vous recevez un message d’erreur `404`, `'A reference was made to a file or folder which does not exist'`. Pour ignorer la lecture des métadonnées du fichier après la création de ce dernier, suivez la procédure pour [ajouter et définir la propriété **Récupérer toutes les métadonnées de fichier** sur **Non**](#file-does-not-exist).

<a name="connect"></a>

## <a name="connect-to-sftp-with-ssh"></a>Se connecter à SFTP avec SSH

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Connectez-vous au [portail Azure](https://portal.azure.com) et ouvrez votre application logique dans le concepteur d’application logique, si elle n’est pas déjà ouverte.

1. Pour les applications logiques vides, dans la zone de recherche, entrez `sftp ssh` comme filtre. Dans la liste des déclencheurs, sélectionnez le déclencheur souhaité.

   -ou-

   Pour les applications logiques existantes, sous la dernière étape où vous souhaitez ajouter une action, sélectionnez **Nouvelle étape**. Dans la zone de recherche, entrez `sftp ssh` en guise de filtre. Dans la liste des actions, sélectionnez l’action souhaitée.

   Pour ajouter une action entre des étapes, placez votre pointeur au-dessus de la flèche qui les sépare. Cliquez sur le signe ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Fournissez les informations nécessaires pour votre connexion.

   > [!IMPORTANT]
   >
   > Lorsque vous entrez votre clé privée SSH dans la propriété **Clé privée SSH**, suivez ces étapes supplémentaires pour vous assurer que vous fournissez la valeur complète et correcte pour cette propriété. Une clé non valide entraîne l’échec de la connexion.

   Bien qu’il est possible d’utiliser n’importe quel éditeur de texte, vous trouverez ici des exemples d’étapes montrant comment copier et coller correctement votre clé à l’aide de Notepad.exe.

   1. Ouvrez votre fichier de clé privée SSH dans un éditeur de texte. Dans cet exemple, les étapes utilisent le Bloc-notes.

   1. Dans le menu **Edition** du Bloc-notes, choisissez **Sélectionner tout**.

   1. Sélectionnez **Edition** > **Copier**.

   1. Dans l’action ou le déclencheur SFTP-SSH que vous avez ajouté(e), *collez l’intégralité* de la clé copiée dans la propriété **Clé privée SSH**, qui prend en charge plusieurs lignes. **_N’entrez pas ni ne modifiez la clé manuellement_**.

1. Après avoir entré les informations de connexion, sélectionnez **Créer**.

1. Fournissez maintenant les informations nécessaires pour le déclencheur ou l’action sélectionnés et continuez à générer le flux de travail de votre application logique.

<a name="change-chunk-size"></a>

## <a name="override-chunk-size"></a>Remplacer la taille de bloc

Pour remplacer le comportement adaptatif par défaut utilisé par la segmentation, vous pouvez spécifier une taille de bloc constante comprise entre 5 Mo et 50 Mo.

1. En haut à droite de l’action, sélectionnez le bouton représentant des points de suspension ( **…** ), puis **Paramètres**.

   ![Ouvrir les paramètres SFTP-SSH](./media/connectors-sftp-ssh/sftp-ssh-connector-setttings.png)

1. Sous **Transfert de contenu**, dans la propriété **Taille de segment**, entrez une valeur entière comprise entre `5` et `50`, par exemple : 

   ![Spécifier la taille de bloc à utiliser à la place](./media/connectors-sftp-ssh/specify-chunk-size-override-default.png)

1. Lorsque vous avez fini, sélectionnez **Terminé**.

## <a name="examples"></a>Exemples

<a name="file-added-modified"></a>

### <a name="sftp---ssh-trigger-when-a-file-is-added-or-modified"></a>SFTP - déclencheur SSH : Lors de l’ajout ou de la modification d’un fichier

Ce déclencheur démarre un workflow quand un fichier est ajouté ou modifié sur un serveur SFTP. Comme exemples d’actions de suivi, le workflow peut utiliser une condition pour vérifier si le contenu du fichier répond aux critères spécifiés. Si le contenu respecte la condition, l’action SFTP-SSH **Obtenir le contenu du fichier** peut obtenir le contenu, puis une autre action SFTP-SSH peut placer ce fichier dans un autre dossier sur le serveur SFTP.

**Exemple en entreprise** : vous pouvez utiliser ce déclencheur pour superviser l’apparition dans un dossier SFTP de nouveaux fichiers représentant les commandes des clients. Vous pouvez ensuite utiliser une action SFTP-SSH comme **Obtenir le contenu du fichier** afin d’obtenir le contenu de la commande à des fins de traitement et stocker cette commande dans une base de données de commandes.

<a name="get-content"></a>

### <a name="sftp---ssh-action-get-file-content-using-path"></a>SFTP - action SSH : Obtenir le contenu d’un fichier à l’aide du chemin

Cette action récupère le contenu d’un fichier sur un serveur SFTP en spécifiant le chemin du fichier. Par exemple, vous pouvez ajouter le déclencheur de l’exemple précédent et une condition à laquelle le contenu du fichier doit satisfaire. Si la condition est vérifiée, l’action qui obtient le contenu peut s’exécuter.

<a name="troubleshooting-errors"></a>

## <a name="troubleshoot-problems"></a>Résoudre les problèmes

Cette section décrit les solutions possibles aux erreurs ou problèmes courants.

<a name="connection-attempt-failed"></a>

### <a name="504-error-a-connection-attempt-failed-because-the-connected-party-did-not-properly-respond-after-a-period-of-time-or-established-connection-failed-because-connected-host-has-failed-to-respond-or-request-to-the-sftp-server-has-taken-more-than-000030-seconds"></a>Erreur 504 : « Une tentative de connexion a échoué car le participant connecté n’a pas répondu convenablement au-delà d’une certaine durée, ou une connexion établie a échoué car l’hôte de connexion n’a pas répondu » ou « La demande au serveur SFTP a pris plus de « 00:00:30 » secondes »

Cette erreur peut se produire lorsque votre application logique ne parvient pas à établir correctement une connexion avec le serveur SFTP. Il peut y avoir différentes raisons à ce problème, ainsi essayez les options de dépannage suivantes :

* Le délai de connexion est de 20 secondes. Vérifiez que votre serveur SFTP bénéficie de bonnes performances et que les appareils intermédiaires, tels que les pare-feu, n’ajoutent pas de surcharge.

* Si vous avez configuré un pare-feu, assurez-vous d’ajouter les adresses **IP du connecteur géré** pour votre région à la liste approuvée. Pour trouver les adresses IP de la région de votre application logique, consultez [IP sortantes du connecteur managé - Azure Logic Apps](/connectors/common/outbound-ip-addresses).

* Si cette erreur se produit par intermittence, remplacez le paramètre **Stratégie de nouvelles tentatives** de l’action SFTP-SSH par un nombre de tentatives supérieur à quatre tentatives par défaut.

* Vérifiez si le serveur SFTP limite le nombre de connexions à partir de chaque adresse IP. Si une limite existe, vous devrez peut-être resteindre le nombre d’instances d’applications logiques simultanées.

* Pour réduire le coût d’établissement de la connexion, dans la configuration SSH de votre serveur SFTP, affectez à la propriété [**ClientAliveInterval**](https://man.openbsd.org/sshd_config#ClientAliveInterval) la valeur d’environ une heure.

* Examinez le journal du serveur SFTP pour vérifier si la demande provenant de l’application logique a joint le serveur SFTP. Pour obtenir plus d’informations sur le problème de connectivité, vous pouvez également exécuter une trace réseau sur votre pare-feu et votre serveur SFTP.

<a name="file-does-not-exist"></a>

### <a name="404-error-a-reference-was-made-to-a-file-or-folder-which-does-not-exist"></a>Erreur 404 : « Une référence a été faite à un fichier ou dossier qui n’existe pas »

Cette erreur peut se produire lorsque votre workflow crée un fichier sur votre serveur SFTP au moyen de l’action SFTP-SSH **Créer un fichier**, mais déplace immédiatement ce fichier avant que le service Logic Apps ne puisse obtenir les métadonnées du fichier. Quand votre workflow exécute l’action **Créer un fichier**, le service Logic Apps appelle automatiquement votre serveur SFTP pour récupérer les métadonnées associées. Toutefois, si votre application logique déplace le fichier, le service Logic Apps ne le trouve plus. C’est pourquoi vous recevez le message d’erreur `404`.

Si vous ne pouvez pas éviter ou retarder le déplacement du fichier, vous pouvez ignorer la lecture des métadonnées du fichier après sa création en procédant de la façon suivante :

1. Dans l’action **Créer un fichier**, ouvrez la liste **Ajouter un nouveau paramètre**, sélectionnez la propriété **Récupérer toutes les métadonnées de fichier** et définissez la valeur sur **Non**.

1. Si vous avez besoin de ces métadonnées de fichier ultérieurement, vous pourrez utiliser l’action **Récupérer les métadonnées de fichier**.

## <a name="connector-reference"></a>Référence de connecteur

Pour plus d’informations techniques sur ce connecteur, notamment au sujet des déclencheurs, des actions et des limites décrits dans le fichier Swagger du connecteur, consultez la [page de référence du connecteur](/connectors/sftpwithssh/).

> [!NOTE]
> Pour les applications logiques utilisées dans un [environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la version de ce connecteur avec l’étiquette ISE nécessite des blocs pour utiliser les [limites de messages de l’ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits) à la place.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)
