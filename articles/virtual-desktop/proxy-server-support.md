---
title: Instructions sur les serveurs proxy pour Azure Virtual Desktop – Azure
description: Quelques instructions et recommandations pour l’utilisation de serveurs proxy dans les déploiements d’Azure Virtual Desktop.
author: Heidilohr
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: helohr
ms.reviewer: denisgun
manager: femila
ms.openlocfilehash: d90f6106d94777c58418a209097d84fe69458d06
ms.sourcegitcommit: 92889674b93087ab7d573622e9587d0937233aa2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2021
ms.locfileid: "130177571"
---
# <a name="proxy-server-guidelines-for-azure-virtual-desktop"></a>Instructions sur les serveurs proxy pour Azure Virtual Desktop

Cet article vous montre comment utiliser un serveur proxy avec Azure Virtual Desktop. Les recommandations de cet article s’appliquent uniquement aux connexions entre l’infrastructure, le client et les agents hôtes de session d’Azure Virtual Desktop. Cet article ne traite pas de la connectivité réseau pour Office, Windows 10, FSLogix ou d’autres applications Microsoft.

## <a name="what-are-proxy-servers"></a>Que sont les serveurs proxy ?

Nous vous recommandons de contourner les proxys pour le trafic d’Azure Virtual Desktop. Les proxys ne rendent pas Azure Virtual Desktop plus sûr, car le trafic est déjà chiffré. Pour en savoir plus sur la sécurité des connexions, consultez [Sécurité des connexions](network-connectivity.md#connection-security). 

La plupart des serveurs proxy ne sont pas conçus pour prendre en charge les connexions WebSocket durables et peuvent nuire à la stabilité des connexions. La scalabilité des serveurs proxy pose également problème, car Azure Virtual Desktop utilise plusieurs connexions à long terme. Si vous utilisez des serveurs proxy, ils doivent être d’une taille adéquate pour gérer ces connexions.

Si la zone géographique du serveur proxy est éloignée de l’hôte, cette distance augmentera la latence des connexions utilisateur. Une latence plus élevée signifie un temps de connexion plus long et une expérience utilisateur moins bonne, en particulier dans les scénarios qui nécessitent des graphiques, de l’audio ou des interactions à faible latence avec les périphériques d’entrée. Si vous devez utiliser un serveur proxy, gardez à l’esprit que vous devez placer le serveur dans la même zone géographique que l’agent Azure Virtual Desktop et le client.

Si vous configurez votre serveur proxy comme étant le seul chemin à emprunter pour le trafic d’Azure Virtual Desktop, les données du protocole RDP sont forcées sur le protocole TCP au lieu du protocole UDP. Cette manœuvre réduit la qualité visuelle et la réactivité de la connexion à distance.

En résumé, nous vous déconseillons d’utiliser des serveurs proxy sur Azure Virtual Desktop, car ils entraînent des problèmes de performance liés à la dégradation de la latence et à la perte de paquets. 

## <a name="bypassing-a-proxy-server"></a>Contournement d’un serveur proxy

Si les stratégies de sécurité et de réseau de votre organisation nécessitent l’utilisation de serveurs proxy pour le trafic web, vous pouvez configurer votre environnement de manière à contourner les connexions d’Azure Virtual Desktop tout en acheminant le trafic via le serveur proxy. Toutefois, les stratégies de chaque organisation étant uniques, certaines méthodes peuvent être plus efficaces que d’autres pour votre déploiement. Voici quelques méthodes de configuration que vous pouvez essayer pour éviter toute perte de performance et de fiabilité dans votre environnement :

- Étiquettes de service Azure sur le pare-feu Azure
- Contournement du serveur proxy à l’aide de fichiers Proxy Auto Configuration (.PAC)
- Liste de contournement dans la configuration du proxy local
- Utilisation de serveurs proxy pour la configuration par utilisateur
- Utilisation de Shortpath RDP pour la connexion RDP tout en conservant le trafic du service sur le proxy

## <a name="recommendations-for-using-proxy-servers"></a>Recommandations relatives à l’utilisation de serveurs proxy

Certaines organisations requièrent que tout le trafic utilisateur passe par un serveur proxy pour le suivi ou l’inspection des paquets. Cette section décrit la configuration recommandée de votre environnement dans ces cas.

### <a name="use-proxy-servers-in-the-same-azure-geography"></a>Utiliser des serveurs proxy dans la même zone géographique Azure

Lorsque vous utilisez un serveur proxy, celui-ci gère toutes les communications avec l’infrastructure d’Azure Virtual Desktop et effectue la résolution DNS et le routage Anycast dans l’instance Azure Front Door la plus proche. Si vos serveurs proxy sont distants ou distribués dans une zone géographique Azure, votre résolution géographique sera moins précise. Une résolution géographique moins précise signifie que les connexions sont acheminées vers un cluster Azure Virtual Desktop plus distant. Pour éviter ce problème, utilisez uniquement des serveurs proxy qui sont géographiquement proches de votre cluster Azure Virtual Desktop.

### <a name="use-rdp-shortpath-for-managed-networks-for-desktop-connectivity"></a>Utiliser Shortpath RDP pour les réseaux gérés pour la connectivité de bureau

Lorsque vous activez Shortpath RDP pour des réseaux managés, les données RDP contournent le serveur proxy, si possible. Le contournement du serveur proxy garantit un routage optimal lors de l’utilisation du transport UDP. D’autres types de trafic d’Azure Virtual Desktop, tels que la répartition, l’orchestration et les diagnostics, passent toujours par le serveur proxy.

### <a name="dont-use-ssl-termination-on-the-proxy-server"></a>Ne pas utilise de terminaison SSL sur le serveur proxy

La terminaison SSL (Secure Sockets Layer) remplace les certificats de sécurité des composants Azure Virtual Desktop par des certificats générés par le serveur proxy. Cette fonctionnalité du serveur proxy permet l’inspection des paquets pour le trafic HTTPS sur le serveur proxy. Toutefois, l’inspection des paquets augmente également le temps de réponse du service, ce qui fait que les utilisateurs mettent plus de temps à se connecter. Pour les scénarios de connexion inverse, l’inspection des paquets du trafic RDP n’est pas nécessaire, car le trafic RDP de la connexion inverse est binaire et utilise des niveaux de chiffrement supplémentaires.

Si vous configurez votre serveur proxy pour qu’il utilise l’inspection SSL, n’oubliez pas que vous ne pouvez pas restaurer votre serveur à son état d’origine après que l’inspection SSL a apporté des modifications. Si un élément de votre environnement Azure Virtual Desktop cesse de fonctionner alors que l’inspection SSL est activée, vous devez désactiver l’inspection SSL et réessayer avant d’ouvrir une demande de support. L’inspection SSL peut également provoquer l’arrêt de l’agent Azure Virtual Desktop parce qu’elle interfère avec les connexions approuvées entre l’agent et le service.

### <a name="dont-use-proxy-servers-that-need-authentication"></a>Ne pas utiliser de serveurs proxy nécessitant une authentification

Les composants Azure Virtual Desktop sur l’hôte de la session s’exécutent dans le contexte de leur système d’exploitation. Ils ne prennent donc pas en charge les serveurs proxy nécessitant une authentification. Si le serveur proxy nécessite une authentification, la connexion échouera.

### <a name="plan-for-the-proxy-server-network-capacity"></a>Planifier la capacité réseau du serveur proxy

Les serveurs proxy ont des limites de capacité. Contrairement au trafic HTTP ordinaire, le trafic RDP comporte des connexions durables et bavardes qui sont bidirectionnelles et consomment beaucoup de bande passante. Avant de configurer un serveur proxy, demandez à votre fournisseur de serveur proxy quel est le débit de votre serveur. Veillez également à lui demander combien de sessions proxy vous pouvez exécuter en même temps. Après avoir déployé le serveur proxy, surveillez attentivement son utilisation des ressources pour détecter les goulots d’étranglement dans le trafic d’Azure Virtual Desktop.

### <a name="proxy-servers-for-windows-7-session-hosts"></a>Serveurs proxy pour les hôtes de session Windows 7

Les hôtes de session fonctionnant sous Windows 7 ne prennent pas en charge les connexions par serveur proxy pour les données RDP à connexion inverse. Si l’hôte de la session ne peut pas se connecter directement aux passerelles Azure Virtual Desktop, la connexion ne fonctionnera pas.

### <a name="proxy-servers-and--teams-optimization"></a>Serveurs proxy et optimisation de Teams

Azure Virtual Desktop ne prend pas en charge les serveurs proxy pour l’optimisation de Teams.

## <a name="session-host-configuration-recommendations"></a>Recommandations de configuration de l’hôte de la session

Pour configurer un serveur proxy au niveau de l’hôte de la session, vous devez activer un proxy à l’échelle du système. N’oubliez pas que la configuration à l’échelle du système concerne tous les composants du système d’exploitation et toutes les applications exécutées sur l’hôte de la session. Les sections suivantes sont des recommandations pour la configuration des proxys à l’échelle du système.
 
### <a name="use-the-web-proxy-auto-discovery-wpad-protocol"></a>Utiliser le protocole WPAD

L’agent Azure Virtual Desktop tente automatiquement de localiser un serveur proxy sur le réseau à l’aide du protocole WPAD (Web Proxy Auto-Discovery). Lors d’une tentative de localisation, l’agent recherche un fichier nommé **wpad.domainsuffix** dans le serveur de noms de domaine (DNS). Si l’agent trouve le fichier dans le DNS, il effectue une requête HTTP pour un fichier nommé **wpad.dat**. La réponse devient le script de configuration du proxy qui choisit le serveur proxy sortant.

Pour configurer votre réseau afin d’utiliser la résolution DNS pour WPAD, suivez les instructions de l’article [Paramètres de détection automatique pour Internet Explorer 11](/internet-explorer/ie11-deploy-guide/auto-detect-settings-for-ie11). Assurez-vous que la liste de refus des requêtes globales du serveur DNS autorise la résolution WPAD en suivant les instructions fournies dans [Set-DnsServerGlobalQueryBlockList](/powershell/module/dnsserver/set-dnsserverglobalqueryblocklist?view=windowsserver2019-ps&preserve-view=true).

### <a name="manually-set-a-device-wide-internet-explorer-proxy"></a>Définir manuellement un proxy Internet Explorer à l’échelle de l’appareil

Vous pouvez définir un proxy à l’échelle de l’appareil ou un fichier Proxy Auto Configuration (.PAC) qui s’applique à tous les utilisateurs interactifs, LocalSystem et NetworkService à l’aide du [fournisseur de services de configuration NetworkProxy](/windows/client-management/mdm/networkproxy-csp). 

Vous pouvez également configurer le serveur proxy pour le compte système local en exécutant la commande **bitsadmin** suivante, comme indiqué dans l’exemple suivant : 

```console
bitsadmin /util /setieproxy LOCALSYSTEM AUTOSCRIPT http://server/proxy.pac 
```

## <a name="client-side-proxy-support"></a>Prise en charge du proxy côté client

Le client Azure Virtual Desktop prend en charge les serveurs proxy configurés avec les paramètres système ou un [fournisseur CSP NetworkProxy](/windows/client-management/mdm/networkproxy-csp).

### <a name="support-for-clients-running-on-windows-7"></a>Prise en charge des clients fonctionnant sous Windows 7

Les clients fonctionnant sous Windows 7 ne prennent pas en charge les connexions par serveur proxy pour les données RDP à connexion inverse. Si le client ne peut pas se connecter directement aux passerelles Azure Virtual Desktop, la connexion ne fonctionnera pas.

### <a name="azure-virtual-desktop-client-support"></a>Prise en charge des clients Azure Virtual Desktop

Le tableau suivant liste les clients Azure Virtual Desktop qui prennent en charge les serveurs proxy :

| Nom du client | Prise en charge du serveur proxy |
|---|---|
| Windows Desktop | Oui |
| Client web | Oui |
| Android | Non |
| iOS | Oui |
| macOS | Oui |
| Windows Store | Oui |

Pour plus d’informations sur la prise en charge des proxys sur les clients dynamiques basés sur Linux, consultez [Prise en charge des clients dynamiques](./user-documentation/linux-overview.md).

## <a name="support-limitations"></a>Limites de la prise en charge

De nombreux services et applications tiers jouent le rôle de serveur proxy. Ces services tiers comprennent des pare-feu distribués de nouvelle génération, des systèmes de sécurité web et des serveurs proxy de base. Nous ne pouvons pas garantir que chaque configuration est compatible avec Azure Virtual Desktop. Microsoft fournit uniquement une prise en charge limitée des connexions établies via un serveur proxy. Si vous rencontrez des problèmes de connectivité lors de l’utilisation d’un serveur proxy, le support de Microsoft vous recommande de configurer un contournement du proxy, puis d’essayer de reproduire le problème.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la sécurisation de vos déploiements d’Azure Virtual Desktop, consultez notre [guide de sécurité](security-guide.md).
