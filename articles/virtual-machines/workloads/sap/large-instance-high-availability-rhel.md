---
title: Haute disponibilité d’Azure (grandes instances) pour SAP sur RHEL
description: Découvrez comment automatiser un basculement de base de données SAP HANA à l’aide d’un cluster Pacemaker dans Red Hat Enterprise Linux.
author: jaawasth
ms.author: jaawasth
ms.service: virtual-machines-sap
ms.topic: how-to
ms.date: 04/19/2021
ms.openlocfilehash: 199d38a4c6ddca96c745342bcef0b07dc78b48bd
ms.sourcegitcommit: 03e84c3112b03bf7a2bc14525ddbc4f5adc99b85
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/03/2021
ms.locfileid: "129401291"
---
# <a name="azure-large-instances-high-availability-for-sap-on-rhel"></a>Haute disponibilité d’Azure (grandes instances) pour SAP sur RHEL

> [!NOTE]
> Cet article contient des références aux termes *liste noire* et *esclave*, termes que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.

Dans cet article, vous allez apprendre à configurer le cluster Pacemaker dans RHEL 7 pour automatiser un basculement de base de données SAP HANA. Vous devez avoir une bonne compréhension de Linux, de SAP HANA et de Pacemaker pour suivre les étapes de ce guide.

Le tableau suivant répertorie les noms d’hôte utilisés dans cet article. Les blocs de code dans l’article montrent les commandes qui doivent être exécutées, ainsi que la sortie de ces commandes. Portez une attention particulière au nœud référencé dans chaque commande.

| Type | Nom de l'hôte | Nœud|
|-------|-------------|------|
|Hôte principal|`sollabdsm35`|nœud 1|
|Hôte secondaire|`sollabdsm36`|nœud 2|

## <a name="configure-your-pacemaker-cluster"></a>Configurer votre cluster Pacemaker


Avant de commencer la configuration du cluster, configurez l’échange de clés SSH pour établir une relation d’approbation entre les nœuds.

1. Utilisez les commandes suivantes pour créer un élément `/etc/hosts` identique sur les deux nœuds.

    ```
    root@sollabdsm35 ~]# cat /etc/hosts
    27.0.0.1 localhost localhost.azlinux.com
    10.60.0.35 sollabdsm35.azlinux.com sollabdsm35 node1
    10.60.0.36 sollabdsm36.azlinux.com sollabdsm36 node2
    10.20.251.150 sollabdsm36-st
    10.20.251.151 sollabdsm35-st
    10.20.252.151 sollabdsm36-back
    10.20.252.150 sollabdsm35-back
    10.20.253.151 sollabdsm36-node
    10.20.253.150 sollabdsm35-node
    ```

2.  Créez et échangez les clés SSH.
    1. Générez les clés SSH.

    ```
       [root@sollabdsm35 ~]# ssh-keygen -t rsa -b 1024
       [root@sollabdsm36 ~]# ssh-keygen -t rsa -b 1024
    ```
    2. Copiez les clés sur les autres hôtes pour le protocole SSH sans mot de passe.
    
       ```
       [root@sollabdsm35 ~]# ssh-copy-id -i /root/.ssh/id_rsa.pub sollabdsm35
       [root@sollabdsm35 ~]# ssh-copy-id -i /root/.ssh/id_rsa.pub sollabdsm36
       [root@sollabdsm36 ~]# ssh-copy-id -i /root/.ssh/id_rsa.pub sollabdsm35
       [root@sollabdsm36 ~]# ssh-copy-id -i /root/.ssh/id_rsa.pub sollabdsm36
       ```

3.  Désactivez selinux sur les deux nœuds.
    ```
    [root@sollabdsm35 ~]# vi /etc/selinux/config

    ...

    SELINUX=disabled

    [root@sollabdsm36 ~]# vi /etc/selinux/config

    ...

    SELINUX=disabled

    ```  

4. Redémarrez les serveurs, puis utilisez la commande suivante pour vérifier l’état de selinux.
    ```
    [root@sollabdsm35 ~]# sestatus

    SELinux status: disabled

    [root@sollabdsm36 ~]# sestatus

    SELinux status: disabled
    ```

5. Configurez le protocole NTP (Network Time Protocol). L’heure et les fuseaux horaires des deux nœuds de cluster doivent correspondre. Utilisez la commande suivante pour ouvrir `chrony.conf` et vérifier le contenu du fichier.
    1. Le contenu suivant doit être ajouté au fichier de configuration. Modifiez les valeurs actuelles en fonction de votre environnement.
        ```
        vi /etc/chrony.conf
    
         Use public servers from the pool.ntp.org project.
    
         Please consider joining the pool (http://www.pool.ntp.org/join.html).
    
        server 0.rhel.pool.ntp.org iburst
       ```
    
    2. Activez le service chrony. 
      
        ```
        systemctl enable chronyd
    
        systemctl start chronyd
    
        
    
        chronyc tracking
    
        Reference ID : CC0BC90A (voipmonitor.wci.com)
    
        Stratum : 3
    
        Ref time (UTC) : Thu Jan 28 18:46:10 2021
    
        chronyc sources
    
        210 Number of sources = 8
    
        MS Name/IP address Stratum Poll Reach LastRx Last sample
    
        ===============================================================================
    
        ^+ time.nullroutenetworks.c> 2 10 377 1007 -2241us[-2238us] +/- 33ms
    
        ^* voipmonitor.wci.com 2 10 377 47 +956us[ +958us] +/- 15ms
    
        ^- tick.srs1.ntfo.org 3 10 177 801 -3429us[-3427us] +/- 100ms
        ```

6. Mettre à jour le système
    1. Tout d’abord, installez les dernières mises à jour sur le système avant de commencer l’installation de l’appareil SBD.
    1. Les clients doivent s’assurer qu’ils disposent au moins de la version 4.1.1-12.el7_6.26 du package resource-agents-sap-hana, comme indiqué dans [Stratégies de support pour les clusters à haute disponibilité RHEL - Gestion de SAP HANA dans un cluster](https://access.redhat.com/articles/3397471)
    1. Si vous ne souhaitez pas une mise à jour complète du système, même si c’est recommandé, mettez au moins à jour les packages suivants.
        1. `resource-agents-sap-hana`
        1. `selinux-policy`
        1. `iscsi-initiator-utils`

  
        ```
        node1:~ # yum update
        ```

7. Installez les référentiels SAP HANA et RHEL-HA.

    ```
    subscription-manager repos –list

    subscription-manager repos
    --enable=rhel-sap-hana-for-rhel-7-server-rpms

    subscription-manager repos --enable=rhel-ha-for-rhel-7-server-rpms
    ```
      

8. Installez les outils Pacemaker, SBD, OpenIPMI, ipmitool et fencing_sbd sur tous les nœuds.

    ``` 
    yum install pcs sbd fence-agent-sbd.x86_64 OpenIPMI
    ipmitool
    ```

  ## <a name="configure-watchdog"></a>Configurer la surveillance (watchdog)

Dans cette section, vous découvrez comment configurer la surveillance (watchdog). Cette section utilise les deux mêmes hôtes, `sollabdsm35` et `sollabdsm36` , référencés au début de cet article.

1. Assurez-vous que le démon de la surveillance n’est pas en cours d’exécution sur les systèmes.
    ```
    [root@sollabdsm35 ~]# systemctl disable watchdog
    [root@sollabdsm36 ~]# systemctl disable watchdog
    [root@sollabdsm35 ~]# systemctl stop watchdog
    [root@sollabdsm36 ~]# systemctl stop watchdog
    [root@sollabdsm35 ~]# systemctl status watchdog

    ● watchdog.service - watchdog daemon

    Loaded: loaded (/usr/lib/systemd/system/watchdog.service; disabled;
    vendor preset: disabled)

    Active: inactive (dead)

    Nov 28 23:02:40 sollabdsm35 systemd[1]: Collecting watchdog.service

    ```

2. La surveillance Linux par défaut, qui sera installée pendant l’installation, est la surveillance iTCO qui n’est pas prise en charge par les systèmes UCS et HPE SDFlex. Par conséquent, cette surveillance doit être désactivée.
    1. La mauvaise surveillance est installée et chargée sur le système :
       ```
       sollabdsm35:~ # lsmod |grep iTCO
   
       iTCO_wdt 13480 0
   
       iTCO_vendor_support 13718 1 iTCO_wdt
       ```

    2. Déchargez le pilote incorrect de l’environnement :
       ```  
       sollabdsm35:~ # modprobe -r iTCO_wdt iTCO_vendor_support
   
       sollabdsm36:~ # modprobe -r iTCO_wdt iTCO_vendor_support
       ```  
        
    3. Pour vous assurer que le pilote n’est pas chargé lors du prochain démarrage du système, le pilote doit être placé dans la liste de refus. Pour placer les modules iTCO dans la liste de refus, ajoutez le code suivant à la fin du fichier `50-blacklist.conf` :
       ```
       sollabdsm35:~ # vi /etc/modprobe.d/50-blacklist.conf
   
        unload the iTCO watchdog modules
   
       blacklist iTCO_wdt
   
       blacklist iTCO_vendor_support
       ```
    4. Copiez le fichier sur l’hôte secondaire.
       ```
       sollabdsm35:~ # scp /etc/modprobe.d/50-blacklist.conf sollabdsm35:
       /etc/modprobe.d/50-blacklist.conf
       ```  

    5. Testez pour voir si le service ipmi est démarré. Il est important que le minuteur IPMI ne soit pas en cours d’exécution. La gestion du minuteur sera effectuée à partir du service SBD pacemaker.
       ```
       sollabdsm35:~ # ipmitool mc watchdog get
   
       Watchdog Timer Use: BIOS FRB2 (0x01)
   
       Watchdog Timer Is: Stopped
   
       Watchdog Timer Actions: No action (0x00)
   
       Pre-timeout interval: 0 seconds
   
       Timer Expiration Flags: 0x00
   
       Initial Countdown: 0 sec
   
       Present Countdown: 0 sec
   
       ``` 

3. Par défaut, l’appareil requis /dev/watchdog ne sera pas créé.

    ```
    sollabdsm35:~ # ls -l /dev/watchdog

    ls: cannot access /dev/watchdog: No such file or directory
    ```

4. Configurez la surveillance (watchdog) IPMI.

    ``` 
    sollabdsm35:~ # mv /etc/sysconfig/ipmi /etc/sysconfig/ipmi.org

    sollabdsm35:~ # vi /etc/sysconfig/ipmi

    IPMI_SI=yes
    DEV_IPMI=yes
    IPMI_WATCHDOG=yes
    IPMI_WATCHDOG_OPTIONS="timeout=20 action=reset nowayout=0
    panic_wdt_timeout=15"
    IPMI_POWEROFF=no
    IPMI_POWERCYCLE=no
    IPMI_IMB=no
    ```
5. Copiez le fichier de configuration de la surveillance (watchdog) sur l’hôte secondaire.
    ```
    sollabdsm35:~ # scp /etc/sysconfig/ipmi
    sollabdsm36:/etc/sysconfig/ipmi
    ```
6.  Activez et démarrez le service ipmi.
    ```
    [root@sollabdsm35 ~]# systemctl enable ipmi

    Created symlink from
    /etc/systemd/system/multi-user.target.wants/ipmi.service to
    /usr/lib/systemd/system/ipmi.service.

    [root@sollabdsm35 ~]# systemctl start ipmi

    [root@sollabdsm36 ~]# systemctl enable ipmi

    Created symlink from
    /etc/systemd/system/multi-user.target.wants/ipmi.service to
    /usr/lib/systemd/system/ipmi.service.

    [root@sollabdsm36 ~]# systemctl start ipmi
    ```
     Le service IPMI est maintenant démarré et l’appareil /dev/watchdog est créé, mais le minuteur est toujours arrêté. Plus tard, SBD va gérer la réinitialisation de la surveillance (watchdog) et activer le minuteur IPMI.
7.  Vérifiez que /dev/watchdog existe, mais qu’il n’est pas en cours d’utilisation.
    ```
    [root@sollabdsm35 ~]# ipmitool mc watchdog get
    Watchdog Timer Use: SMS/OS (0x04)
    Watchdog Timer Is: Stopped
    Watchdog Timer Actions: No action (0x00)
    Pre-timeout interval: 0 seconds
    Timer Expiration Flags: 0x10
    Initial Countdown: 20 sec
    Present Countdown: 20 sec

    [root@sollabdsm35 ~]# ls -l /dev/watchdog
    crw------- 1 root root 10, 130 Nov 28 23:12 /dev/watchdog
    [root@sollabdsm35 ~]# lsof /dev/watchdog
    ```

## <a name="sbd-configuration"></a>Configuration de SBD
Dans cette section, vous découvrez comment configurer SBD. Cette section utilise les deux mêmes hôtes, `sollabdsm35` et `sollabdsm36` , référencés au début de cet article.

1.  Assurez-vous que le disque iSCSI ou FC est visible sur les deux nœuds. Cet exemple utilise un appareil SBD basé sur FC. Pour plus d’informations sur l’isolation SBD, consultez [Conseils de conception pour les clusters à haute disponibilité RHEL - Considérations sur SBD](https://access.redhat.com/articles/2941601) et [Stratégies de support pour les clusters à haute disponibilité RHEL- sbd et fence_sbd](https://access.redhat.com/articles/2800691).
2.  L’ID LUN doit être identique sur tous les nœuds.
  
3.  Vérifiez l’état multichemin pour l’appareil SBD.
    ```
    multipath -ll
    3600a098038304179392b4d6c6e2f4b62 dm-5 NETAPP ,LUN C-Mode
    size=1.0G features='4 queue_if_no_path pg_init_retries 50
    retain_attached_hw_handle' hwhandler='1 alua' wp=rw
    |-+- policy='service-time 0' prio=50 status=active
    | |- 8:0:1:2 sdi 8:128 active ready running
    | `- 10:0:1:2 sdk 8:160 active ready running
    `-+- policy='service-time 0' prio=10 status=enabled
    |- 8:0:3:2 sdj 8:144 active ready running
    `- 10:0:3:2 sdl 8:176 active ready running
    ```

4.  Création des disques SBD et configuration de l’isolation primitive du cluster. Cette étape doit être exécutée sur le premier nœud.
    ```
    sbd -d /dev/mapper/3600a098038304179392b4d6c6e2f4b62 -4 20 -1 10 create 

    Initializing device /dev/mapper/3600a098038304179392b4d6c6e2f4b62
    Creating version 2.1 header on device 4 (uuid:
    ae17bd40-2bf9-495c-b59e-4cb5ecbf61ce)

    Initializing 255 slots on device 4

    Device /dev/mapper/3600a098038304179392b4d6c6e2f4b62 is initialized.
    ```

5.  Copiez la configuration de SBD sur le nœud 2.
    ```
    vi /etc/sysconfig/sbd

    SBD_DEVICE="/dev/mapper/3600a09803830417934d6c6e2f4b62" 
    SBD_PACEMAKER=yes
    SBD_STARTMODE=always
    SBD_DELAY_START=no
    SBD_WATCHDOG_DEV=/dev/watchdog
    SBD_WATCHDOG_TIMEOUT=15
    SBD_TIMEOUT_ACTION=flush,reboot
    SBD_MOVE_TO_ROOT_CGROUP=auto
    SBD_OPTS=

    scp /etc/sysconfig/sbd node2:/etc/sysconfig/sbd
    ```

6.  Vérifiez que le disque SBD est visible à partir des deux nœuds.
    ```
    sbd -d /dev/mapper/3600a098038304179392b4d6c6e2f4b62 dump

    ==Dumping header on disk /dev/mapper/3600a098038304179392b4d6c6e2f4b62

    Header version : 2.1

    UUID : ae17bd40-2bf9-495c-b59e-4cb5ecbf61ce

    Number of slots : 255
    Sector size : 512
    Timeout (watchdog) : 5
    Timeout (allocate) : 2
    Timeout (loop) : 1
    Timeout (msgwait) : 10

    ==Header on disk /dev/mapper/3600a098038304179392b4d6c6e2f4b62 is dumped
    ```

7.  Ajoutez l’appareil SBD dans le fichier de configuration de SBD.

    ```
    # SBD_DEVICE specifies the devices to use for exchanging sbd messages
    # and to monitor. If specifying more than one path, use ";" as
    # separator.
    #

    SBD_DEVICE="/dev/mapper/3600a098038304179392b4d6c6e2f4b62"
    ## Type: yesno
     Default: yes
     # Whether to enable the pacemaker integration.
    SBD_PACEMAKER=yes
    ```

## <a name="cluster-initialization"></a>Initialisation du cluster
Dans cette section, vous initialisez le cluster. Cette section utilise les deux mêmes hôtes, `sollabdsm35` et `sollabdsm36` , référencés au début de cet article.

1.  Configurez le mot de passe utilisateur du cluster (tous les nœuds).
    ```
    passwd hacluster
    ```
2.  Démarrez PCS sur tous les systèmes.
    ```
    systemctl enable pcsd
    ```
  

3.  Arrêtez le pare-feu et désactivez-le (tous les nœuds).
    ```
    systemctl disable firewalld 

     systemctl mask firewalld

    systemctl stop firewalld
    ```

4.  Démarrez le service pcsd.
    ```
    systemctl start pcsd
    ```

5.  Exécutez l’authentification du cluster uniquement à partir du nœud 1.

    ```
    pcs cluster auth sollabdsm35 sollabdsm36

        Username: hacluster

            Password:
            sollabdsm35.localdomain: Authorized
            sollabdsm36.localdomain: Authorized

     ``` 

6.  Créez le cluster.
    ```
    pcs cluster setup --start --name hana sollabdsm35 sollabdsm36
    ```
  

7.  Vérifiez l’état du cluster.

    ```
    pcs cluster status

    Cluster name: hana

    WARNINGS:

    No stonith devices and stonith-enabled is not false

    Stack: corosync

    Current DC: sollabdsm35 (version 1.1.20-5.el7_7.2-3c4c782f70) -
    partition with quorum

    Last updated: Sat Nov 28 20:56:57 2020

    Last change: Sat Nov 28 20:54:58 2020 by hacluster via crmd on
    sollabdsm35

    2 nodes configured

    0 resources configured

    Online: [ sollabdsm35 sollabdsm36 ]

    No resources

    Daemon Status:

    corosync: active/disabled

    pacemaker: active/disabled

    pcsd: active/disabled
    ```

8. Si un nœud ne rejoint pas le cluster, vérifiez si le pare-feu est toujours en cours d’exécution.

9. Créer et activer l’appareil SBD
    ```
    pcs stonith create SBD fence_sbd devices=/dev/mapper/3600a098038303f4c467446447a
    ```
  
10. Arrêtez le cluster, redémarrez les services du cluster (sur tous les nœuds).

    ```
    pcs cluster stop --all
    ```

11. Redémarrez les services du cluster (sur tous les nœuds).

    ```
    systemctl stop pcsd
    systemctl stop pacemaker
    systemctl stop corocync
    systemctl enable sbd
    systemctl start corosync
    systemctl start pacemaker
    systemctl start pcsd
    ```

12. Corosync doit démarrer le service SBD.

    ```
    systemctl status sbd

    ● sbd.service - Shared-storage based fencing daemon

    Loaded: loaded (/usr/lib/systemd/system/sbd.service; enabled; vendor
    preset: disabled)

    Active: active (running) since Wed 2021-01-20 01:43:41 EST; 9min ago
    ```

13. Redémarrez le cluster (s’il n’est pas démarré automatiquement à partir de pcsd).

    ```
    pcs cluster start –-all

    sollabdsm35: Starting Cluster (corosync)...

    sollabdsm36: Starting Cluster (corosync)...

    sollabdsm35: Starting Cluster (pacemaker)...

    sollabdsm36: Starting Cluster (pacemaker)...
    ```
  

14. Activez les paramètres Stonith.
    ```
    pcs stonith enable SBD --device=/dev/mapper/3600a098038304179392b4d6c6e2f4d65
    pcs property set stonith-watchdog-timeout=20
    pcs property set stonith-action=reboot
    ```
  

15. Vérifiez le nouvel état du cluster avec une ressource.
    ```
    pcs status

    Cluster name: hana

    Stack: corosync

    Current DC: sollabdsm35 (version 1.1.16-12.el7-94ff4df) - partition
    with quorum

    Last updated: Tue Oct 16 01:50:45 2018

    Last change: Tue Oct 16 01:48:19 2018 by root via cibadmin on
    sollabdsm35

    2 nodes configured

    1 resource configured

    Online: [ sollabdsm35 sollabdsm36 ]

    Full list of resources:

    SBD (stonith:fence_sbd): Started sollabdsm35

    Daemon Status:

    corosync: active/disabled

    pacemaker: active/disabled

    pcsd: active/enabled

    sbd: active/enabled

    [root@node1 ~]#
    ```
  

16. Le minuteur IPMI doit maintenant être exécuté et l’appareil /dev/watchdog doit être ouvert par SBD.

    ```
    ipmitool mc watchdog get

    Watchdog Timer Use: SMS/OS (0x44)

    Watchdog Timer Is: Started/Running

    Watchdog Timer Actions: Hard Reset (0x01)

    Pre-timeout interval: 0 seconds

    Timer Expiration Flags: 0x10

    Initial Countdown: 20 sec

    Present Countdown: 19 sec

    [root@sollabdsm35 ~] lsof /dev/watchdog

    COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME

    sbd 117569 root 5w CHR 10,130 0t0 323812 /dev/watchdog
    ```

17. Vérifiez l’état de SBD.

    ```
    sbd -d /dev/mapper/3600a098038304445693f4c467446447a list

    0 sollabdsm35 clear

    1 sollabdsm36 clear
    ```
  

18. Testez l’isolation SBD en bloquant le noyau.

    * Déclenchez le blocage du noyau.

      ```
      echo c > /proc/sysrq-trigger

      System must reboot after 5 Minutes (BMC timeout) or the value which is
      set as panic_wdt_timeout in the /etc/sysconfig/ipmi config file.
      ```
  
    * Le deuxième test à exécuter consiste à isoler un nœud à l’aide de commandes PCS.

      ```
      pcs stonith fence sollabdsm36
      ```
  

19. Pour le reste du clustering SAP HANA, vous pouvez désactiver STONITH en définissant :

   * pcs property set `stonith-enabled=false`
   * Il est parfois plus facile que STONITH reste désactivé pendant l’installation du cluster. Vous éviterez ainsi les redémarrages inattendus du système.
   * Ce paramètre doit avoir la valeur true pour une utilisation en production. Si ce paramètre n’est pas défini sur true, le cluster n’est pas pris en charge.
   * pcs property set `stonith-enabled=true`

## <a name="hana-integration-into-the-cluster"></a>Intégration de HANA dans le cluster

Dans cette section, vous intégrez HANA dans le cluster. Cette section utilise les deux mêmes hôtes, `sollabdsm35` et `sollabdsm36` , référencés au début de cet article.

La méthode par défaut et prise en charge consiste à créer un scénario à performances optimisées dans lequel la base de données peut être basculée directement. Seul ce scénario est décrit dans ce document. Dans ce cas, nous vous recommandons d’installer un cluster pour le système QAS et un cluster distinct pour le système PRD. Dans ce cas uniquement, il est possible de tester tous les composants avant le passage en production.


* Ce processus est issu de la description RHEL sur la page :

  * https://access.redhat.com/articles/3004101

 ### <a name="steps-to-follow-to-configure-hsr"></a>Étapes à suivre pour configurer HSR

 | **Mode de réplication de journal**            | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **synchrone en mémoire (par défaut)** | Synchrone en mémoire (mode=syncmem) signifie que l’écriture dans le journal est considérée comme réussie, quand l’entrée du journal a été écrite dans le volume du fichier journal de l’instance principale et que l’envoi du journal a été accepté par l’instance secondaire après la copie en mémoire. Quand la connexion au système secondaire est perdue, le système principal continue le traitement des transactions et écrit les modifications uniquement sur le disque local. Une perte de données peut se produire lorsque les systèmes principal et secondaire tombent en panne en même temps tant que le système secondaire est connecté ou lorsqu’une prise de contrôle est exécutée et que le système secondaire est déconnecté. Cette option offre de meilleures performances parce qu’il n’est pas nécessaire d’attendre les E/S disque sur l’instance secondaire, mais elle est plus vulnérable à la perte de données.                                                                                                                                                                                                                                                                                                                     |
| **Synchrone**                     | Synchrone (mode=sync) signifie que l’écriture dans le journal est considérée comme réussie quand l’entrée du journal a été écrite dans le volume du fichier journal de l’instance principale et de l’instance secondaire. Quand la connexion au système secondaire est perdue, le système principal continue le traitement des transactions et écrit les modifications uniquement sur le disque local. Aucune perte de données ne se produit dans ce scénario tant que le système secondaire est connecté. Une perte de données peut se produire lorsqu’une prise de contrôle est exécutée alors que le système secondaire est déconnecté. En outre, ce mode de réplication peut fonctionner avec une option de synchronisation complète. Cela signifie que l’écriture dans le journal est réussie quand le tampon du journal a été écrit dans le fichier journal de l’instance principale et de l’instance secondaire. De plus, quand le système secondaire est déconnecté (par exemple, en raison d’une défaillance du réseau), le système principal interrompt le traitement transactionnel jusqu’à ce que la connexion au système secondaire soit rétablie. Aucune perte de données ne se produit dans ce scénario. Vous pouvez définir l’option de synchronisation complète pour la réplication du système uniquement avec le paramètre \[system\_replication\]/enable\_full\_sync. Pour plus d’informations sur l’activation de l’option de synchronisation complète, consultez Activer l’option de synchronisation complète pour la réplication du système.                                                                                                                                                                                                                                                                                                              |
| **Asynchrone**                    | Asynchrone (mode=async) signifie que le système principal envoie des tampons de journal de restauration au système secondaire de manière asynchrone. Le système principal valide une transaction quand elle a été écrite dans le fichier journal du système principal et envoyée au système secondaire par le biais du réseau. Il n’attend pas la confirmation du système secondaire. Cette option offre de meilleures performances, car il n’est pas nécessaire d’attendre les E/S du journal sur le système secondaire. La cohérence de la base de données pour tous les services sur le système secondaire est garantie. Toutefois, elle est plus vulnérable à la perte de données. Des modifications de données peuvent être perdues lors d’une prise de contrôle.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |

1.  Voici les actions à exécuter sur le nœud 1 (principal).
    1. Assurez-vous que le mode de journalisation de la base de données est défini sur normal.

       ```  
   
       * su - hr2adm
   
       * hdbsql -u system -p $YourPass -i 00 "select value from
       "SYS"."M_INIFILE_CONTENTS" where key='log_mode'"
   
       
   
       VALUE
   
       "normal"
       ```
    2. La réplication du système SAP HANA ne fonctionne qu’une fois la sauvegarde initiale effectuée. La commande suivante crée une sauvegarde initiale dans le répertoire `/tmp/`. Sélectionnez un système de fichiers de sauvegarde approprié pour la base de données. 
       ```
       * hdbsql -i 00 -u system -p $YourPass "BACKUP DATA USING FILE
       ('/tmp/backup')"
   
   
   
       Backup files were created
   
       ls -l /tmp
   
       total 2031784
   
       -rw-r----- 1 hr2adm sapsys 155648 Oct 26 23:31 backup_databackup_0_1
   
       -rw-r----- 1 hr2adm sapsys 83894272 Oct 26 23:31 backup_databackup_2_1
   
       -rw-r----- 1 hr2adm sapsys 1996496896 Oct 26 23:31 backup_databackup_3_1
   
       ```  

    3. Sauvegardez tous les conteneurs de cette base de données.
       ``` 
       * hdbsql -i 00 -u system -p $YourPass -d SYSTEMDB "BACKUP DATA USING
       FILE ('/tmp/sydb')"     
   
       * hdbsql -i 00 -u system -p $YourPass -d SYSTEMDB "BACKUP DATA FOR HR2
       USING FILE ('/tmp/rh2')"
   
       ```

    4. Activez le processus HSR sur le système source.
       ```
       hdbnsutil -sr_enable --name=DC1

       nameserver is active, proceeding ...

       successfully enabled system as system replication source site

       done.
       ```

    5. Vérifiez l’état du système principal.
       ```
       hdbnsutil -sr_state
    
       System Replication State
   
   
       online: true
   
       mode: primary
   
       operation mode: primary
   
       site id: 1
   
       site name: DC1
   
       
   
       is source system: true
   
       is secondary/consumer system: false
   
       has secondaries/consumers attached: false
   
       is a takeover active: false
   
       
   
       Host Mappings:
   
       ~~~~~~~~~~~~~~
   
       Site Mappings:
   
       ~~~~~~~~~~~~~~
   
       DC1 (primary/)
   
       Tier of DC1: 1
   
       Replication mode of DC1: primary
   
       Operation mode of DC1:
   
       done.
       ```

 2. Voici les actions à exécuter sur le nœud 2 (secondaire).
     1. Arrêtez la base de données.
       ```
       su – hr2adm
    
       sapcontrol -nr 00 -function StopSystem
       ```
    

     2. Pour SAP HANA 2.0 uniquement, copiez les fichiers `PKI SSFS_HR2.KEY` et `SSFS_HR2.DAT` du système SAP HANA du nœud principal vers le nœud secondaire.
       ```
       scp
       root@node1:/usr/sap/HR2/SYS/global/security/rsecssfs/key/SSFS_HR2.KEY
       /usr/sap/HR2/SYS/global/security/rsecssfs/key/SSFS_HR2.KEY
   
       
   
       scp
       root@node1:/usr/sap/HR2/SYS/global/security/rsecssfs/data/SSFS_HR2.DAT
       /usr/sap/HR2/SYS/global/security/rsecssfs/data/SSFS_HR2.DAT
       ```

     3. Activez le nœud secondaire en tant que site de réplication.
       ``` 
       su - hr2adm
   
       hdbnsutil -sr_register --remoteHost=node1 --remoteInstance=00
       --replicationMode=syncmem --name=DC2
   
       
   
       adding site ...
   
       --operationMode not set; using default from global.ini/[system_replication]/operation_mode: logreplay
   
       nameserver node2:30001 not responding.
   
       collecting information ...
   
       updating local ini files ...
   
       done.
   
       ```

     4. Démarrez la base de données.
       ```
       sapcontrol -nr 00 -function StartSystem 
       ```
    
     5. Vérifiez l’état de la base de données.
       ```
       hdbnsutil -sr_state
   
       ~~~~~~~~~
       System Replication State
   
       online: true
   
       mode: syncmem
   
       operation mode: logreplay
   
       site id: 2
   
       site name: DC2   
   
       is source system: false
   
       is secondary/consumer system: true
   
       has secondaries/consumers attached: false
   
       is a takeover active: false
   
       active primary site: 1
   
       
   
       primary primarys: node1
   
       Host Mappings:
   
   
   
       node2 -> [DC2] node2
   
       node2 -> [DC1] node1
   
       
   
       Site Mappings:
   
   
   
       DC1 (primary/primary)
   
       |---DC2 (syncmem/logreplay)
   
       
   
       Tier of DC1: 1
   
       Tier of DC2: 2
   
       
   
       Replication mode of DC1: primary
   
       Replication mode of DC2: syncmem
   
       Operation mode of DC1: primary
   
       Operation mode of DC2: logreplay
   
       
   
       Mapping: DC1 -> DC2
   
       done.
       ~~~~~~~~~~~~~~
       ```

3. Vous pouvez également obtenir plus d’informations sur l’état de la réplication :
    ```
    ~~~~~
    hr2adm@node1:/usr/sap/HR2/HDB00> python
    /usr/sap/HR2/HDB00/exe/python_support/systemReplicationStatus.py

    | Database | Host | Port | Service Name | Volume ID | Site ID | Site
    Name | Secondary | Secondary | Secondary | Secondary | Secondary |
    Replication | Replication | Replication |

    | | | | | | | | Host | Port | Site ID | Site Name | Active Status |
    Mode | Status | Status Details |

    | SYSTEMDB | node1 | 30001 | nameserver | 1 | 1 | DC1 | node2 | 30001
    | 2 | DC2 | YES | SYNCMEM | ACTIVE | |

    | HR2 | node1 | 30007 | xsengine | 2 | 1 | DC1 | node2 | 30007 | 2 |
    DC2 | YES | SYNCMEM | ACTIVE | |

    | HR2 | node1 | 30003 | indexserver | 3 | 1 | DC1 | node2 | 30003 | 2
    | DC2 | YES | SYNCMEM | ACTIVE | |


    status system replication site "2": ACTIVE

    overall system replication status: ACTIVE

    Local System Replication State
    

    mode: PRIMARY

    site id: 1

    site name: DC1
    ```
  

#### <a name="log-replication-mode-description"></a>Description du mode de réplication du journal

Pour plus d’informations sur le mode de réplication du journal, consultez la [documentation SAP officielle](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/627bd11e86c84ec2b9fcdf585d24011c.html).
  

#### <a name="network-setup-for-hana-system-replication"></a>Configuration du réseau pour la réplication du système HANA


Pour vous assurer que le trafic de réplication utilise le VLAN approprié pour la réplication, vous devez le configurer correctement dans `global.ini`. Si vous ignorez cette étape, HANA utilise le VLAN d’accès pour la réplication, ce qui peut ne pas convenir.


Les exemples suivants illustrent la configuration de la résolution du nom d’hôte pour la réplication du système sur un site secondaire. Trois réseaux distincts peuvent être identifiés :

* Réseau public avec des adresses comprises dans la plage de 10.0.1.*

* Réseau pour la communication SAP HANA interne entre les hôtes sur chaque site : 192.168.1.*

* Réseau dédié pour la réplication du système : 10.5.1.*

Dans le premier exemple, le paramètre `[system_replication_communication]listeninterface` a été défini sur `.global` et seuls les hôtes du site de réplication voisin sont spécifiés.

Dans l’exemple suivant, le paramètre `[system_replication_communication]listeninterface` a été défini sur `.internal` et tous les hôtes des deux sites sont spécifiés.

  

Pour plus d’informations, consultez [Configuration réseau pour la réplication de système SAP Hana](https://www.sap.com/documents/2016/06/18079a1c-767c-0010-82c7-eda71af511fa.html).

  

Pour la réplication du système, il n’est pas nécessaire de modifier le fichier `/etc/hosts`, les noms d’hôte internes (« virtuels ») doivent être mappés à des adresses IP dans le fichier `global.ini` afin de créer un réseau dédié pour la réplication du système. La syntaxe est la suivante :

global.ini

[system_replication_hostname_resolution]

<ip-address_site>=<internal-host-name_site>


## <a name="configure-sap-hana-in-a-pacemaker-cluster"></a>Configurer SAP HANA dans un cluster Pacemaker
Dans cette section, vous découvrez comment configurer SAP HANA dans un cluster Pacemaker. Cette section utilise les deux mêmes hôtes, `sollabdsm35` et `sollabdsm36` , référencés au début de cet article.

Vérifiez que les prérequis suivants sont remplis :  

* Le cluster Pacemaker est configuré conformément à la documentation et dispose d’une isolation correcte qui fonctionne

* Le démarrage de SAP HANA à l’amorçage est désactivé sur tous les nœuds du cluster, car le démarrage et l’arrêt seront gérés par le cluster

* La réplication et la prise de contrôle du système SAP HANA à l’aide d’outils SAP fonctionnent correctement entre les nœuds du cluster

* SAP HANA contient un compte de supervision qui peut être utilisé par le cluster à partir des deux nœuds du cluster

* Les deux nœuds sont abonnés aux canaux « Haute disponibilité » et « RHEL for SAP HANA » (RHEL 6, RHEL 7)

  

* En général, vous devez exécuter toutes les commandes pcs à partir d’un seul nœud, car CIB sera automatiquement mis à jour à partir de l’interpréteur de commandes pcs.

* [Plus d’informations sur la stratégie de quorum](https://access.redhat.com/solutions/645843)

### <a name="steps-to-configure"></a>Étapes de configuration 
1. Configurez pcs.
    ```
    [root@node1 ~]# pcs property unset no-quorum-policy (optional – only if it was set before)
    [root@node1 ~]# pcs resource defaults resource-stickiness=1000
    [root@node1 ~]# pcs resource defaults migration-threshold=5000
    ```
2.  Configurez corosync.
    Pour plus d’informations, consultez [Comment configurer mon cluster haute disponibilité RHEL 7 avec Pacemaker et Corosync](https://access.redhat.com/solutions/1293523).
    ```
    cat /etc/corosync/corosync.conf

    totem {

    version: 2

    secauth: off

    cluster_name: hana

    transport: udpu

    }

    

    nodelist {

    node {

    ring0_addr: node1.localdomain

    nodeid: 1

    }

    

    node {

    ring0_addr: node2.localdomain

    nodeid: 2

    }

    }

    

    quorum {

    provider: corosync_votequorum

    two_node: 1

    }

    

    logging {

    to_logfile: yes

    logfile: /var/log/cluster/corosync.log

    to_syslog: yes

    }

    ```
  

3.  Créez une ressource SAPHanaTopology clonée.
    La ressource SAPHanaTopology recueille l’état et la configuration de la réplication du système SAP HANA sur chaque nœud. SAPHanaTopology requiert la configuration des attributs suivants.
       ```
       pcs resource create SAPHanaTopology_HR2_00 SAPHanaTopology SID=HR2 op start timeout=600 \
       op stop timeout=300 \
       op monitor interval=10 timeout=600 \
       clone clone-max=2 clone-node-max=1 interleave=true

       ```

    | Nom d’attribut | Description  |
    |---|---|
    | SID | Identificateur du système SAP (SID) de l’installation de SAP HANA. Doit être identique pour tous les nœuds. |
    | NuméroInstance | Identificateur d’instance SAP à 2 chiffres.|

    * État de la ressource
       ```
       pcs resource show SAPHanaTopology_HR2_00
   
       Clone: SAPHanaTopology_HR2_00-clone
        Meta Attrs: clone-max=2 clone-node-max=1 interleave=true
        Resource: SAPHanaTopology_HR2_00 (class=ocf provider=heartbeat type=SAPHanaTopology)
         Attributes: InstanceNumber=00 SID=HR2
         Operations: monitor interval=60 timeout=60 (SAPHanaTopology_HR2_00-monitor-interval-60)
                     start interval=0s timeout=180 (SAPHanaTopology_HR2_00-start-interval-0s)
                     stop interval=0s timeout=60 (SAPHanaTopology_HR2_00-stop-interval-0s)
       
         
       ```

4.  Créez une ressource SAPHana principale/secondaire.
    * La ressource SAPHana est responsable du démarrage, de l’arrêt et le déplacement de la base de données SAP HANA. Cette ressource doit être exécutée en tant que ressource de cluster primaire/secondaire. La ressource dispose des attributs suivants.

| Nom de l'attribut            | Requis ? | Valeur par défaut | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|---------------------------|-----------|---------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SID                       | Oui       | None          | Identificateur du système SAP (SID) de l’installation de SAP HANA. Doit être identique pour tous les nœuds.                                                                                                                                                                                                                                                                                                                                                                                       |
| NuméroInstance            | Oui       | Aucun          | Identifiant d’instance SAP à 2 chiffres.                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| PREFER_SITE_TAKEOVER      | non        | Oui           | Le cluster doit-il être préférablement basculé vers l’instance secondaire au lieu de redémarrer localement le serveur principal ? (« non » : De préférence redémarrage local ; « oui » : De préférence prise de contrôle par un site sitant)                                                                                                                                                                                                                                                                                            |
|                           |           |               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| AUTOMATED_REGISTER        | non        | FALSE         | L’ancien serveur principal SAP HANA doit-il être enregistré comme secondaire après prise de contrôle et DUPLICATE_PRIMARY_TIMEOUT ? (« faux » : non, une intervention manuelle est nécessaire. « vrai » : oui, l’ancien serveur principal sera enregistré par l’agent de ressources comme secondaire)                                                                                                                                                                                                                        |
| DUPLICATE_PRIMARY_TIMEOUT | non        | 7200          | Différence de temps (en secondes) nécessaire entre les horodatages principaux, si une situation double principal se produit. Si la différence de temps est inférieure à l’intervalle de temps, le cluster conserve une instance ou les deux instances à l’état « En attente ». Cela permet à un administrateur de réagir en cas de basculement. Un ancien serveur principal ayant échoué sera enregistré une fois la différence de temps écoulée. Après l’inscription au nouveau serveur principal, toutes les données seront remplacées par la réplication du système. |

5.  Créez les ressources HANA.
    ```
    pcs resource create SAPHana_HR2_00 SAPHana SID=HR2 InstanceNumber=00 PREFER_SITE_TAKEOVER=true DUPLICATE_PRIMARY_TIMEOUT=7200 AUTOMATED_REGISTER=true op start timeout=3600 \
    op stop timeout=3600 \
    op monitor interval=61 role="Slave" timeout=700 \
    op monitor interval=59 role="Master" timeout=700 \
    op promote timeout=3600 \
    op demote timeout=3600 \
    master meta notify=true clone-max=2 clone-node-max=1 interleave=true


    pcs resource show SAPHana_HR2_00-primary


    Primary: SAPHana_HR2_00-primary
     Meta Attrs: clone-max=2 clone-node-max=1 interleave=true notify=true
     Resource: SAPHana_HR2_00 (class=ocf provider=heartbeat type=SAPHana)
      Attributes: AUTOMATED_REGISTER=false DUPLICATE_PRIMARY_TIMEOUT=7200 InstanceNumber=00 PREFER_SITE_TAKEOVER=true SID=HR2
      Operations: demote interval=0s timeout=320 (SAPHana_HR2_00-demote-interval-0s)
                  monitor interval=120 timeout=60 (SAPHana_HR2_00-monitor-interval-120)
                  monitor interval=121 role=Secondary timeout=60 (SAPHana_HR2_00-monitor-
                  interval-121)
                  monitor interval=119 role=Primary timeout=60 (SAPHana_HR2_00-monitor-
                  interval-119)
                  promote interval=0s timeout=320 (SAPHana_HR2_00-promote-interval-0s)
                  start interval=0s timeout=180 (SAPHana_HR2_00-start-interval-0s)
                  stop interval=0s timeout=240 (SAPHana_HR2_00-stop-interval-0s)
   

    
    
    crm_mon -A1

    ....

    2 nodes configured

    5 resources configured

    Online: [ node1.localdomain node2.localdomain ]

    Active resources:

    .....

    Node Attributes:

    * Node node1.localdomain:

    + hana_hr2_clone_state : PROMOTED

    + hana_hr2_remoteHost : node2

    + hana_hr2_roles : 4:P:primary1:primary:worker:primary

    + hana_hr2_site : DC1

    + hana_hr2_srmode : syncmem

    + hana_hr2_sync_state : PRIM

    + hana_hr2_version : 2.00.033.00.1535711040

    + hana_hr2_vhost : node1

    + lpa_hr2_lpt : 1540866498

    + primary-SAPHana_HR2_00 : 150

    * Node node2.localdomain:

    + hana_hr2_clone_state : DEMOTED

    + hana_hr2_op_mode : logreplay

    + hana_hr2_remoteHost : node1

    + hana_hr2_roles : 4:S:primary1:primary:worker:primary

    + hana_hr2_site : DC2

    + hana_hr2_srmode : syncmem

    + hana_hr2_sync_state : SOK

    + hana_hr2_version : 2.00.033.00.1535711040

    + hana_hr2_vhost : node2

    + lpa_hr2_lpt : 30

    + primary-SAPHana_HR2_00 : 100
    ```

6.  Créez une ressource d’adresse IP virtuelle.
    Le cluster contiendra une adresse IP virtuelle afin d’atteindre l’instance principale de SAP HANA. Voici un exemple de commande pour créer une ressource IPaddr2 avec l’adresse IP 10.7.0.84/24.
    ```
    pcs resource create vip_HR2_00 IPaddr2 ip="10.7.0.84"
    pcs resource show vip_HR2_00

    Resource: vip_HR2_00 (class=ocf provider=heartbeat type=IPaddr2)

        Attributes: ip=10.7.0.84

        Operations: monitor interval=10s timeout=20s
    (vip_HR2_00-monitor-interval-10s)

        start interval=0s timeout=20s (vip_HR2_00-start-interval-0s)

        stop interval=0s timeout=20s (vip_HR2_00-stop-interval-0s)
    ```

7.  Créez des contraintes.
    * Pour un fonctionnement approprié, nous devons nous assurer que les ressources SAPHanaTopology sont démarrées avant de démarrer les ressources SAPHana, et que l’adresse IP virtuelle est présente sur le nœud où la ressource principale de SAPHana est en cours d’exécution. Pour ce faire, les deux contraintes suivantes doivent être créées.
       ```
       pcs constraint order SAPHanaTopology_HR2_00-clone then SAPHana_HR2_00-primary symmetrical=false
       pcs constraint colocation add vip_HR2_00 with primary SAPHana_HR2_00-primary 2000
       ```

###  <a name="testing-the-manual-move-of-saphana-resource-to-another-node"></a>Test du déplacement manuel de la ressource SAPHana vers un autre nœud

#### <a name="sap-hana-takeover-by-cluster"></a>(prise de contrôle de SAP Hana par le cluster)


Pour tester le déplacement de la ressource SAPHana d’un nœud à un autre, utilisez la commande ci-dessous. Notez que l’option `--primary` ne doit pas être utilisée lors de l’exécution de la commande suivante en raison du mode de fonctionnement de la ressource SAPHana en interne.

`pcs resource move SAPHana_HR2_00-primary`

Après chaque appel de commande de déplacement de ressource pcs, le cluster crée des contraintes d’emplacement pour effectuer le déplacement de la ressource. Ces contraintes doivent être supprimées pour permettre le basculement automatique à l’avenir.
Pour les supprimer, vous pouvez utiliser la commande suivante.

```
pcs resource clear SAPHana_HR2_00-primary
crm_mon -A1
Node Attributes:
    * Node node1.localdomain:
    + hana_hr2_clone_state : DEMOTED
    + hana_hr2_remoteHost : node2
    + hana_hr2_roles : 2:P:primary1::worker:
    + hana_hr2_site : DC1
    + hana_hr2_srmode : syncmem
    + hana_hr2_sync_state : PRIM
    + hana_hr2_version : 2.00.033.00.1535711040
    + hana_hr2_vhost : node1
    + lpa_hr2_lpt : 1540867236
    + primary-SAPHana_HR2_00 : 150
    * Node node2.localdomain:
    + hana_hr2_clone_state : PROMOTED
    + hana_hr2_op_mode : logreplay
    + hana_hr2_remoteHost : node1
    + hana_hr2_roles : 4:S:primary1:primary:worker:primary
    + hana_hr2_site : DC2
    + hana_hr2_srmode : syncmem
    + hana_hr2_sync_state : SOK
    + hana_hr2_version : 2.00.033.00.1535711040
    + hana_hr2_vhost : node2
    + lpa_hr2_lpt : 1540867311
    + primary-SAPHana_HR2_00 : 100
```
  

* Connectez-vous à HANA pour vérifier.

  * Hôte rétrogradé :

    ```
    hdbsql -i 00 -u system -p $YourPass -n 10.7.0.82

    result:

            * -10709: Connection failed (RTE:[89006] System call 'connect'
    failed, rc=111:Connection refused (10.7.0.82:30015))
    ```
  
  * Hôte promu :

    ```
    hdbsql -i 00 -u system -p $YourPass -n 10.7.0.84
    
    Welcome to the SAP HANA Database interactive terminal.
    
    
    
    Type: \h for help with commands
    
    \q to quit
    
    
    
    hdbsql HR2=>
    
    
    
    
    DB is online
    ```
  

Avec l’option `AUTOMATED_REGISTER=false`, vous ne pouvez pas basculer d’avant en arrière.

Si cette option a la valeur false, vous devez réinscrire le nœud :
```
hdbnsutil -sr_register --remoteHost=node2 --remoteInstance=00 --replicationMode=syncmem --name=DC1
```

À présent, le nœud 2, qui était le nœud principal, agit comme hôte secondaire.

Envisagez d’affecter la valeur true à cette option pour automatiser l’inscription de l’hôte rétrogradé.
  
```
pcs resource update SAPHana_HR2_00-primary AUTOMATED_REGISTER=true
pcs cluster node clear node1
```

La préférence pour l’enregistrement automatique dépend du scénario du client. Le réenregistrement automatique du nœud après une prise de contrôle est plus facile pour l’équipe d’exploitation. Toutefois, vous souhaiterez peut-être enregistrer le nœud manuellement pour exécuter d’abord des tests supplémentaires afin de vérifier que tout fonctionne comme prévu.

##  <a name="references"></a>Références

1. [Réplication de système SAP HANA automatisée dans un scale-up au sein d’un cluster Pacemaker](https://access.redhat.com/articles/3397471)
2. [Stratégies de support pour les clusters à haute disponibilité RHEL - Gestion de SAP HANA dans un cluster](https://access.redhat.com/articles/3397471)
3. [Configuration de Pacemaker sur RHEL dans Azure - Machines Virtuelles Azure](high-availability-guide-rhel-pacemaker.md)
4. [Contrôle des grandes instances Azure HANA à l’aide du portail Azure - Machines Virtuelles Azure](hana-li-portal.md)
