---
title: Vérification en deux étapes - Azure AD MFA et ADFS - Azure Active Directory
description: Page Azure AD Multi-Factor Authentication décrivant la prise en main d'Azure AD MFA et d'AD FS.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 08/27/2021
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: e9abc9e2e2ca2b9305cccf5cec554994c3703ef1
ms.sourcegitcommit: f53f0b98031cd936b2cd509e2322b9ee1acba5d6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/30/2021
ms.locfileid: "123214494"
---
# <a name="getting-started-with-azure-ad-multi-factor-authentication-and-active-directory-federation-services"></a>Prise en main d'Azure AD Multi-Factor Authentication et des services de fédération Active Directory (AD FS)

<center>

![Prise en main d'Azure AD MFA et d'AD FS](./media/multi-factor-authentication-get-started-adfs/adfs.png)</center>

Si votre organisation a fédéré votre instance locale d'Active Directory avec Azure Active Directory à l'aide d'AD FS, vous disposez de deux options pour l'utilisation d'Azure AD Multi-Factor Authentication.

* Sécuriser les ressources cloud à l'aide d'Azure AD Multi-Factor Authentication ou des services de fédération Active Directory (AD FS)
* Sécuriser les ressources de cloud et locales à l'aide du serveur Azure Multi-Factor Authentication

Le tableau suivant récapitule le processus de vérification avec la sécurisation des ressources à l'aide d'Azure AD Multi-Factor Authentication et d'AD FS

| Processus de vérification : applications basées sur le navigateur | Processus de vérification : applications hors navigateur |
|:--- |:--- |
| Sécurisation des ressources Azure AD à l'aide d'Azure AD Multi-Factor Authentication |<li>La première étape de vérification est effectuée en local à l’aide d’AD FS.</li> <li>La seconde étape est une méthode par téléphone effectuée à l’aide de l’authentification cloud.</li> |
| Sécurisation des ressources Azure AD à l'aide des services ADFS |<li>La première étape de vérification est effectuée en local à l’aide d’AD FS.</li><li>La seconde étape est effectuée localement en répondant à la demande.</li> |

Mises en garde relatives aux mots de passe d'application pour les utilisateurs fédérés :

* Les mots de passe d’applications sont vérifiés à l’aide de l’authentification cloud et contournent donc la fédération. La fédération n’est utilisée activement que lorsque vous configurez un mot de passe d’application.
* Les paramètres de contrôle d’accès client locaux ne sont pas honorés par les mots de passe d’application.
* Vous perdez la fonctionnalité de journalisation-authentification locale pour les mots de passe d’application.
* La désactivation/suppression de compte peut mettre jusqu’à 3 heures pour synchroniser les répertoires, ce qui peut retarder la désactivation/suppression des mots de passe d’application dans l’identité de cloud.

Pour plus d'informations sur la configuration d'Azure AD Multi-Factor Authentication ou du serveur Azure Multi-Factor Authentication avec AD FS, consultez les articles suivants :

* [Sécuriser les ressources cloud à l'aide d'Azure AD Multi-Factor Authentication et d'AD FS](howto-mfa-adfs.md)
* [Sécuriser les ressources de cloud et locales à l'aide du serveur Azure Multi-Factor Authentication avec Windows Server](howto-mfaserver-adfs-windows-server.md)
* [Sécuriser les ressources de cloud et locales à l'aide du serveur Azure Multi-Factor Authentication avec AD FS 2.0](howto-mfaserver-adfs-2.md)
