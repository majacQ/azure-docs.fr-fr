---
title: Guide pratique pour apporter une image personnalisée Linux à partir de votre environnement lab physique
description: Explique comment apporter une image Linux personnalisée depuis votre environnement de labo physique.
ms.date: 07/27/2021
ms.topic: how-to
ms.openlocfilehash: 6f044c062b770c6653a1e239ca7c1074ed969b9c
ms.sourcegitcommit: 92889674b93087ab7d573622e9587d0937233aa2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2021
ms.locfileid: "130180891"
---
# <a name="bring-a-linux-custom-image-from-your-physical-lab-environment"></a>Apporter une image personnalisée Linux depuis votre environnement de labo physique

Les étapes de cet article montrent comment importer une image Linux personnalisée qui démarre à partir de votre environnement de labo physique. Avec cette approche, vous créez un disque dur virtuel à partir de votre environnement physique, puis vous importez le disque dur virtuel dans une galerie d’images partagées afin qu’il puisse être utilisé dans Azure Lab Services. Avant d’utiliser cette approche pour créer une image personnalisée, lisez les [Approches recommandées pour la création d’images personnalisées](approaches-for-custom-image-creation.md) afin de déterminer celle adaptée à votre scénario.

Azure approuve de nombreuses [distributions et versions](../virtual-machines/linux/endorsed-distros.md#supported-distributions-and-versions). Les étapes permettant d’apporter une image Linux personnalisée depuis un disque dur virtuel varient d’une distribution à l’autre. Chaque distribution est différente, car chacune d’elles a des prérequis uniques qui doivent être configurés pour qu’elle puisse s’exécuter sur Azure.

Dans cet article, nous allons montrer les étapes permettant d’apporter une image Ubuntu 16.04\18.04\20.04 personnalisée depuis un disque dur virtuel. Pour plus d’informations sur l’utilisation d’un disque dur virtuel afin de créer des images personnalisées pour d’autres distributions, consultez la [Procédure générique pour les distributions Linux](../virtual-machines/linux/create-upload-generic.md).

## <a name="prerequisites"></a>Configuration requise

Pour effectuer les étapes indiquées dans cet article, vous devez disposer de l’autorisation de créer un [disque managé Azure](../virtual-machines/managed-disks-overview.md) dans l’abonnement Azure de votre établissement scolaire.

Dans le cadre d’un déplacement d’images d’un environnement lab physique vers Lab Services, restructurez chaque image afin qu’elle inclue uniquement les logiciels nécessaires à un cours de labo. Pour plus d’informations, lisez le billet de blog sur le [Déplacement d’un labo physique vers Azure Lab Services](https://techcommunity.microsoft.com/t5/azure-lab-services/moving-from-a-physical-lab-to-azure-lab-services/ba-p/1654931).

## <a name="prepare-a-custom-image-by-using-hyper-v-manager"></a>Préparer une image personnalisée avec le Gestionnaire Hyper-V

Les étapes suivantes montrent comment créer une image Ubuntu 18.04\20.04 à partir d’une machine virtuelle Hyper-V en utilisant le Gestionnaire Windows Hyper-V.

1. Téléchargez sur votre ordinateur hôte Windows l’image officielle de [Linux Ubuntu Server](https://ubuntu.com/server/docs) que vous utiliserez pour configurer l’image personnalisée sur une machine virtuelle Hyper-V.

   Si vous utilisez Ubuntu 18.04 LTS, nous vous recommandons d’utiliser une image sur laquelle le bureau graphique [GNOME](https://www.gnome.org/) ou [MATE](https://mate-desktop.org/) *n’est pas installé*. GNOME et MATE sont actuellement en conflit réseau avec l’agent Linux Azure, celui-ci étant nécessaire au bon fonctionnement de l’image dans Azure Lab Services. Utilisez à la place une image Ubuntu Server et installez un autre bureau graphique, par exemple [XFCE](https://www.xfce.org/).  Une autre option consiste à installer [GNOME\MATE](https://github.com/Azure/azure-devtestlab/tree/master/samples/ClassroomLabs/Scripts/LinuxGraphicalDesktopSetup/GNOME_MATE/ReadMe.md) avec le modèle de machine virtuelle d’un lab.

   Ubuntu publie également des [disques durs virtuels Azure prédéfinis téléchargeables](https://cloud-images.ubuntu.com/). Ces disques durs virtuels sont destinés à la création d’images personnalisées à partir d’un ordinateur hôte Linux et d’un hyperviseur tel que KVM. Pour ces disques durs virtuels, vous devez d’abord définir le mot de passe utilisateur par défaut, opération qui ne peut être effectuée qu’avec des outils Linux, tels que qemu, qui ne sont pas disponibles pour Windows. Ainsi, quand vous créez une image personnalisée avec Windows Hyper-V, vous ne pouvez pas vous connecter à ces disques durs virtuels pour personnaliser l’image. Pour plus d’informations sur les disques durs virtuels Azure prédéfinis, consultez la [documentation d’Ubuntu](https://help.ubuntu.com/community/UEC/Images?_ga=2.114783623.1858181609.1624392241-1226151842.1623682781#QEMU_invocation).

1. Commencez par une machine virtuelle Hyper-V dans votre environnement lab physique créé à partir de votre image. Pour plus d’informations, lisez l’article sur la [création d’une machine virtuelle dans Hyper-V](/windows-server/virtualization/hyper-v/get-started/create-a-virtual-machine-in-hyper-v). Définissez les paramètres comme indiqué ici :
    - La machine virtuelle doit être créée en tant que machine virtuelle de **Génération 1**.
    - Utilisez l’option de configuration réseau **Commutateur par défaut** pour permettre à la machine virtuelle de se connecter à Internet.
    - Dans les paramètres **Connecter un disque dur virtuel**, la **taille** du disque *ne doit pas* être supérieure à 128 Go, comme indiqué dans l’image suivante.

        :::image type="content" source="./media/upload-custom-image-shared-image-gallery/connect-virtual-hard-disk.png" alt-text="Capture de l’écran Connecter un disque dur virtuel.":::

    - Dans les paramètres **Options d’installation**, sélectionnez le fichier **.iso** que vous avez téléchargé à partir d’Ubuntu.

    Les images dont la taille de disque dépasse 128 Go *ne sont pas* prises en charge par Lab Services.

1. Connectez-vous à la machine virtuelle Hyper-V et préparez-la pour Azure en suivant les [étapes manuelles de création et de chargement d’un disque dur virtuel Ubuntu](../virtual-machines/linux/create-upload-ubuntu.md#manual-steps).

    Les étapes de préparation d’une image Linux pour Azure varient selon la distribution. Pour plus d’informations et pour connaître les étapes propres à chaque distribution, consultez les [distributions et versions](../virtual-machines/linux/endorsed-distros.md#supported-distributions-and-versions).

    Quand vous suivez les étapes précédentes, gardez à l’esprit certains points importants :
    - Les étapes créent une image [généralisée](../virtual-machines/shared-image-galleries.md#generalized-and-specialized-images) quand vous exécutez la commande **deprovision+user**. Toutefois, cela ne garantit pas que l’image est exempte de toute information sensible ou qu’elle convient pour la redistribution.
    - La dernière étape consiste à convertir le fichier **VHDX** en un fichier **VHD**. Voici les étapes équivalentes qui montrent comment effectuer cette opération avec le **Gestionnaire Hyper-V** :

        1. Accédez à **Gestionnaire Hyper-V** > **Action** > **Modifier le disque**.
        1. Localisez le disque VHDX à convertir.
        1. Ensuite, choisissez **Convertir** pour convertir le disque.
        1. Sélectionnez l’option de conversion au **format de disque dur virtuel**.
        1. Pour le **Type de disque**, sélectionnez **Taille fixe**.
            - Si vous choisissez aussi d’augmenter la taille du disque à ce stade, veillez à ne *pas* dépasser 128 Go.
            :::image type="content" source="./media/upload-custom-image-shared-image-gallery/choose-action.png" alt-text="Capture de l’écran Choisir une action.":::

Pour vous aider à redimensionner le disque dur virtuel et à le convertir en un disque VHDX, vous pouvez également utiliser les applets de commande PowerShell suivantes :

- [Resize-VHD](/powershell/module/hyper-v/resize-vhd)
- [Convert-VHD](/powershell/module/hyper-v/convert-vhd)

## <a name="upload-the-custom-image-to-a-shared-image-gallery"></a>Chargez l’image personnalisée sur une galerie d’images partagées

1. Chargez le disque dur virtuel sur Azure pour créer un disque managé.
    1. Vous pouvez utiliser l’Explorateur Stockage Azure ou AzCopy à partir de la ligne de commande, comme démontré dans l’article [Charger un disque dur virtuel sur Azure ou copier un disque managé dans une autre région](../virtual-machines/windows/disks-upload-vhd-to-managed-disk-powershell.md).

        > [!WARNING]
        > Si votre ordinateur se met en veille ou se verrouille, le processus de chargement peut s’interrompre et échouer. Assurez-vous aussi que quand AzCopy se termine, vous révoquez l’accès SAS au disque. Dans le cas contraire, quand vous tentez de créer une image à partir du disque, vous obtenez une erreur : L’opération « Créer l’image » n’est pas prise en charge avec le disque « nom de votre disque » dans l’état « Chargement actif ». Code d’erreur : OperationNotAllowed*. »

    1. Après avoir chargé le disque dur virtuel, vous devriez désormais disposer d’un disque managé visible dans le portail Azure.

        Vous pouvez utiliser l’onglet **Taille+Performances** du portail Azure pour le disque managé afin de modifier la taille de votre disque. Comme mentionné précédemment, la taille *ne doit pas* être supérieure à 128 Go.

1. Dans une galerie d’images partagées, créez une définition et une version d’image :
    1. [Créez une définition d’image](../virtual-machines/image-version.md) :
        - Choisissez **Gen 1** pour la **génération de la machine virtuelle**.
        - Choisissez **Linux** comme **Système d’exploitation**.
        - Choisissez **Généralisé** comme **État du système d’exploitation**.

        Pour plus d’informations sur les valeurs que vous pouvez spécifier pour une définition d’image, consultez [Définitions d’image](../virtual-machines/shared-image-galleries.md#image-definitions).

        Vous pouvez également choisir d’utiliser une définition d’image existante, et créer une nouvelle version de votre image personnalisée.

    1. [Créez une version d’image](../virtual-machines/image-version.md) :
        - La propriété **Numéro de version** utilise le format suivant : *VersionMajeure.VersionMineure.Patch*. Lorsque vous utilisez Lab Services pour créer un labo et choisir une image personnalisée, la version la plus récente de l’image est automatiquement utilisée. La version la plus récente est choisie en fonction de la valeur la plus élevée de MajorVersion, de MinorVersion, puis de Patch.
        - Pour la **Source**, sélectionnez **Disques et/ou captures instantanées** à partir de la liste déroulante.
        - Pour la propriété **Disque de système d’exploitation**, choisissez le disque que vous avez créé lors des étapes précédentes.

        Pour plus d’informations sur les valeurs que vous pouvez spécifier dans une version d’image, consultez [Versions d’image](../virtual-machines/shared-image-galleries.md#image-versions).

## <a name="create-a-lab"></a>Création d’un laboratoire

[Créez le labo](tutorial-setup-classroom-lab.md) dans Lab Services et sélectionnez l’image personnalisée à partir de la galerie d’images partagées.

Si vous avez augmenté la taille du disque *après* l’installation du système d’exploitation sur la machine virtuelle Hyper-V d’origine, vous devrez peut-être également étendre la partition dans le système de fichiers de Linux pour utiliser l’espace de disque non alloué.  Connectez-vous au modèle de machine virtuelle du labo et suivez les étapes similaires à celles présentées dans [Étendre une partition de disque et un système de fichiers](../virtual-machines/linux/expand-disks.md#expand-a-disk-partition-and-filesystem).

Le disque du système d’exploitation se trouve généralement sur la partition **/dev/sad2**. Pour afficher la taille actuelle de la partition du disque du système d’exploitation, utilisez la commande **df -h**.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble de Shared Image Gallery](../virtual-machines/shared-image-galleries.md)
- [Attacher ou détacher une galerie d’images partagées](how-to-attach-detach-shared-image-gallery.md)
- [Utiliser une galerie d’images partagées](how-to-use-shared-image-gallery.md)