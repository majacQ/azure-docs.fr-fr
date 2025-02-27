---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à CylancePROTECT | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et CylancePROTECT.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/24/2021
ms.author: jeedes
ms.openlocfilehash: bd04fd631269fbf86943cf976f8ca00653ae595e
ms.sourcegitcommit: 677e8acc9a2e8b842e4aef4472599f9264e989e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2021
ms.locfileid: "132314817"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cylanceprotect"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à CylancePROTECT

Dans ce tutoriel, vous allez découvrir comment intégrer CylancePROTECT à Azure Active Directory (Azure AD). Quand vous intégrez CylancePROTECT à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à CylancePROTECT.
* Permettre à vos utilisateurs de se connecter automatiquement à CylancePROTECT avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement CylancePROTECT pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* CylancePROTECT prend en charge l’authentification unique initiée par le **fournisseur d’identité**.

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="add-cylanceprotect-from-the-gallery"></a>Ajouter CylancePROTECT à partir de la galerie

Pour configurer l’intégration de CylancePROTECT à Azure AD, vous devez ajouter CylancePROTECT, disponible dans la galerie, à votre liste d’applications SaaS managées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **CylancePROTECT** dans la zone de recherche.
1. Sélectionnez **CylancePROTECT** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-cylanceprotect"></a>Configurer et tester l’authentification unique Azure AD pour CylancePROTECT

Configurez et testez l’authentification unique Azure AD avec CylancePROTECT pour un utilisateur de test nommé **B. Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur CylancePROTECT associé.

Pour configurer et tester l’authentification unique Azure AD avec CylancePROTECT, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique CylancePROTECT](#configure-cylanceprotect-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test CylancePROTECT](#create-cylanceprotect-test-user)** pour avoir un équivalent de B.Simon dans CylancePROTECT lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **CylancePROTECT**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de crayon de **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la page **Configurer l’authentification unique avec SAML**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur**, tapez l’une des URL suivantes :
    
    | Région | Valeur de l’URL |
    |----------|---------|
    | Nord-est de l’Asie-Pacifique (APNE1)| `https://login-apne1.cylance.com/EnterpriseLogin/ConsumeSaml`|
    | Sud-est de l’Asie-Pacifique (AU) | `https://login-au.cylance.com/EnterpriseLogin/ConsumeSaml` |
    | Europe centrale (EUC1)|`https://login-euc1.cylance.com/EnterpriseLogin/ConsumeSaml`|
    | Amérique du Nord|`https://login.cylance.com/EnterpriseLogin/ConsumeSaml`|
    | Amérique du Sud (SAE1)|`https://login-sae1.cylance.com/EnterpriseLogin/ConsumeSaml`|

    b. Dans la zone de texte **URL de réponse**, tapez l’une des URL suivantes :
    
    | Région | Valeur de l’URL |
    |----------|---------|
    | Nord-est de l’Asie-Pacifique (APNE1)|`https://login-apne1.cylance.com/EnterpriseLogin/ConsumeSaml`|
    | Sud-est de l’Asie-Pacifique (AU)|`https://login-au.cylance.com/EnterpriseLogin/ConsumeSaml`|
    | Europe centrale (EUC1)|`https://login-euc1.cylance.com/EnterpriseLogin/ConsumeSaml`|
    | Amérique du Nord|`https://login.cylance.com/EnterpriseLogin/ConsumeSaml`|
    | Amérique du Sud (SAE1)|`https://login-sae1.cylance.com/EnterpriseLogin/ConsumeSaml`|

1. L’application CylancePROTECT attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jetons SAML. La capture d’écran suivante montre la liste des attributs par défaut, où **nameidentifier** est mappé avec **user.userprincipalname**. L’application CylancePROTECT s’attend à ce que **nameidentifier** soit mappé sur **user.mail** et à ce que toutes les autres revendications soient supprimées. Vous devez donc modifier le mappage d’attribut en cliquant sur l’icône **Modifier**.

    ![image](common/edit-attribute.png)

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer CylancePROTECT**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

> [!NOTE]
> Ouvrez le certificat encodé en base64 téléchargé dans un éditeur de texte et copiez uniquement le texte qui se trouve entre les balises **START** et **END** pour le coller dans le portail d’administration Cylance.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory**, **Utilisateurs**, puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans le champ **Nom**, entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur**, entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à CylancePROTECT.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **CylancePROTECT**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-cylanceprotect-sso"></a>Configurer l’authentification unique CylancePROTECT

Pour configurer l’authentification unique côté **CylancePROTECT**, vous devez envoyer le **Certificat (Base64)** téléchargé et les URL correspondantes copiées à partir du portail Azure à l’[équipe du support technique de CylancePROTECT](https://www.cylance.com/en-us/resources/support/support-overview.html). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés. Pour plus d’informations, consultez la documentation Cylance : [https://support.cylance.com/s/](https://support.cylance.com/s/).

### <a name="create-cylanceprotect-test-user"></a>Créer un utilisateur de test CylancePROTECT

Dans cette section, créez un utilisateur appelé Britta Simon dans CylancePROTECT. Collaborez avec l’administrateur de la console pour ajouter les utilisateurs à la plateforme CylancePROTECT. Le titulaire du compte Azure Active Directory reçoit un message électronique contenant un lien à suivre pour confirmer son compte et l’activer.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes.

* Cliquez sur Tester cette application dans le portail Azure ; vous devez être connecté automatiquement à l’instance de CylancePROTECT pour laquelle vous avez configuré l’authentification unique.

* Vous pouvez utiliser Mes applications de Microsoft. Lorsque vous cliquez sur la vignette CylancePROTECT dans Mes applications, vous devez être connecté automatiquement à l’instance de CylancePROTECT pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur Mes applications, consultez [Présentation de Mes applications](https://support.microsoft.com/account-billing/sign-in-and-start-apps-from-the-my-apps-portal-2f3b1bae-0e5a-4a86-a33e-876fbd2a4510).


## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez configuré CylancePROTECT, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Defender for Cloud Apps](/cloud-app-security/proxy-deployment-any-app).
