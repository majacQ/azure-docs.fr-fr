---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory avec MS Azure SSO Access for Ethidex Compliance Office™ | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et MS Azure SSO Access for Ethidex Compliance Office™.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/06/2021
ms.author: jeedes
ms.openlocfilehash: 882fc86100b5f6852a06d223eb62ffa5b1540308
ms.sourcegitcommit: 677e8acc9a2e8b842e4aef4472599f9264e989e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2021
ms.locfileid: "132326018"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-ms-azure-sso-access-for-ethidex-compliance-office"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory avec MS Azure SSO Access for Ethidex Compliance Office™

Dans ce tutoriel, vous allez découvrir comment intégrer MS Azure SSO Access for Ethidex Compliance Office™ avec Azure Active Directory (Azure AD). Quand vous intégrez MS Azure SSO Access for Ethidex Compliance Office™ avec Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à MS Azure SSO Access for Ethidex Compliance Office™.
* Permettre à vos utilisateurs de se connecter automatiquement à MS Azure SSO Access for Ethidex Compliance Office™ avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.


## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement MS Azure SSO Access for Ethidex Compliance Office™ pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* MS Azure SSO Access for Ethidex Compliance Office™ prend en charge l’authentification unique lancée par le **fournisseur d’identité**.

## <a name="adding-ms-azure-sso-access-for-ethidex-compliance-office-from-the-gallery"></a>Ajout de MS Azure SSO Access for Ethidex Compliance Office™ à partir de la galerie

Pour configurer l’intégration de MS Azure SSO Access for Ethidex Compliance Office™ dans Azure AD, vous devez ajouter MS Azure SSO Access for Ethidex Compliance Office™ à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **MS Azure SSO Access for Ethidex Compliance Office™** dans la zone de recherche.
1. Sélectionnez **MS Azure SSO Access for Ethidex Compliance Office™** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-ms-azure-sso-access-for-ethidex-compliance-office"></a>Configurer et tester l’authentification unique Azure AD pour MS Azure SSO Access for Ethidex Compliance Office™

Configurez et testez l’authentification unique Azure AD avec MS Azure SSO Access for Ethidex Compliance Office™ à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans MS Azure SSO Access for Ethidex Compliance Office™.

Pour configurer et tester l’authentification unique Azure AD avec MS Azure SSO Access for Ethidex Compliance Office™, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique MS Azure SSO Access for Ethidex Compliance Office](#configure-ms-azure-sso-access-for-ethidex-compliance-office-sso)** pour configurer les paramètres d’authentification unique côté application.
    1. **[Créer un utilisateur de test MS Azure SSO Access for Ethidex Compliance Office](#create-ms-azure-sso-access-for-ethidex-compliance-office-test-user)** pour avoir un équivalent de B.Simon dans MS Azure SSO Access for Ethidex Compliance Office™ lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, dans la page d’intégration de l’application **MS Azure SSO Access for Ethidex Compliance Office™**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de crayon de **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `com.ethidex.prod.<CLIENTID>`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://www.ethidex.com/saml2/sp/acs/<CLIENTID>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels. Contactez l’[équipe du support technique de MS Azure SSO Access for Ethidex Compliance Office™](mailto:support@ethidex.com) pour obtenir ces valeurs. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. L’application MS Azure SSO Access for Ethidex Compliance Office™ s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut, où **nameidentifier** est mappé avec **user.userprincipalname**. L’application MS Azure SSO Access for Ethidex Compliance Office™ s’attend à ce que **nameidentifier** soit mappé à **user.mail**. Vous devez donc modifier le mappage d’attribut en cliquant sur l’icône **Modifier**.

    ![image](common/edit-attribute.png)

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer MS Azure SSO Access for Ethidex Compliance Office™**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

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

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à MS Azure SSO Access for Ethidex Compliance Office™.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **MS Azure SSO Access for Ethidex Compliance Office™**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-ms-azure-sso-access-for-ethidex-compliance-office-sso"></a>Configurer MS Azure SSO Access for Ethidex Compliance Office SSO

Pour configurer l’authentification unique du côté **MS Azure SSO Access for Ethidex Compliance Office™**, vous devez envoyer le **Certificat (en base64)** téléchargé et les URL appropriées copiées à partir du portail Azure à l’[équipe de support technique MS Azure SSO Access for Ethidex Compliance Office™](mailto:support@ethidex.com). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-ms-azure-sso-access-for-ethidex-compliance-office-test-user"></a>Créer un utilisateur de test MS Azure SSO Access for Ethidex Compliance Office SSO

Dans cette section, vous allez créer un utilisateur appelé B.Simon dans MS Azure SSO Access for Ethidex Compliance Office™. Rapprochez-vous de l’[équipe de support MS Azure SSO Access for Ethidex Compliance Office™](mailto:support@ethidex.com) pour ajouter les utilisateurs dans la plateforme MS Azure SSO Access for Ethidex Compliance Office™. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes.

* Dans le portail Azure, cliquez sur Tester cette application. Vous êtes alors automatiquement connecté à l’instance d’Ethidex Compliance Office™ pour laquelle vous avez configuré l’authentification unique.

* Vous pouvez utiliser Mes applications de Microsoft. Quand vous cliquez sur la vignette Ethidex Compliance Office™ dans Mes applications, vous êtes automatiquement connecté à l’instance d’Ethidex Compliance Office™ pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur Mes applications, consultez [Présentation de Mes applications](https://support.microsoft.com/account-billing/sign-in-and-start-apps-from-the-my-apps-portal-2f3b1bae-0e5a-4a86-a33e-876fbd2a4510).


## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Ethidex Compliance Office™, vous pouvez appliquer le contrôle de session, qui protège en temps réel contre l’exfiltration et l’infiltration de données sensibles de votre organisation. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Defender for Cloud Apps](/cloud-app-security/proxy-deployment-any-app).
