---
title: 'Tutoriel : Intégration d’Azure Active Directory à Predictix Price Reporting | Microsoft Docs'
description: Dans ce tutoriel, vous allez découvrir comment configurer l’authentification unique entre Azure Active Directory et Predictix Price Reporting.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/26/2019
ms.author: jeedes
ms.openlocfilehash: 48fe4b7b30baef036aa3961f189c5354a264b813
ms.sourcegitcommit: 0770a7d91278043a83ccc597af25934854605e8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/13/2021
ms.locfileid: "124733947"
---
# <a name="tutorial-azure-active-directory-integration-with-predictix-price-reporting"></a>Tutoriel : Intégration d’Azure Active Directory à Predictix Price Reporting

Dans ce didacticiel, vous allez apprendre à intégrer Predictix Price Reporting à Azure AD (Azure Active Directory).

Cette intégration offre les avantages suivants :

* Vous pouvez utiliser Azure AD pour contrôler qui a accès à Predictix Price Reporting.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Predictix Price Reporting (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Predictix Price Reporting, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [abonnement d’essai gratuit d’un mois](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Predictix Price Reporting pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Predictix Price Reporting prend en charge l’authentification unique lancée par le fournisseur de services.

## <a name="adding-predictix-price-reporting-from-the-gallery"></a>Ajout de Predictix Price Reporting à partir de la galerie

Pour configurer l’intégration de Predictix Price Reporting avec Azure AD, vous devez ajouter Predictix Price Reporting à partir de la galerie à votre liste d’applications SaaS managées.

1. Dans le [portail Azure](https://portal.azure.com), dans le volet de gauche, sélectionnez **Azure Active Directory** :

    ![Sélectionnez Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** > **Toutes les applications** :

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la fenêtre :

    ![Sélectionner la nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Predictix Price Reporting**. Sélectionnez **Predictix Price Reporting** dans les résultats de la recherche, puis **Ajouter**.

     ![Résultats de la recherche](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Predictix Price Reporting au moyen d’un utilisateur de test appelé Britta Simon.
Pour activer l’authentification unique, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur correspondant dans Predictix Price Reporting.

Pour configurer et tester l’authentification unique Azure AD avec Predictix Price Reporting, procédez comme suit :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser la fonctionnalité.
2. **[Configurer l’authentification unique Predictix Price Reporting](#configure-predictix-price-reporting-single-sign-on)** côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour lui permettre d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Predictix Price Reporting](#create-a-predictix-price-reporting-test-user)** qui est lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** pour vérifier que la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous allez activer l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Predictix Price Reporting, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Predictix Price Reporting**, sélectionnez **Authentification unique** :

    ![Sélectionner l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique :

    ![Sélectionner une méthode d’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** :

    ![Icône Modifier](common/edit-urls.png)

4. Dans la boîte de dialogue **Configuration SAML de base**, procédez comme suit.

    ![Boîte de dialogue Configuration SAML de base](common/sp-identifier.png)

    1. Dans la zone **URL de connexion**, entrez une URL au format suivant :

       `https://<companyname-pricing>.predictix.com/sso/request`

    1. Dans la zone **Identificateur (ID d’entité)** , entrez une URL au format suivant :

        ```https
        https://<companyname-pricing>.predictix.com
        https://<companyname-pricing>.dev.predictix.com
        ```

    > [!NOTE]
    > Ces valeurs sont des espaces réservés. Vous devez utiliser l’URL de connexion et l’identificateur réels. Contactez l’[équipe du support technique Predictix Price Reporting](https://www.infor.com/company/customer-center/) pour obtenir les valeurs. Vous pouvez aussi vous référer aux modèles figurant dans la boîte de dialogue **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, sélectionnez le lien **Télécharger** en regard du **Certificat (Base64)** correspondant à vos besoins, puis enregistrez le certificat sur votre ordinateur :

    ![Lien de téléchargement du certificat](common/certificatebase64.png)

6. Dans la section **Configurer Predictix Price Reporting**, copiez la ou les URL appropriées, selon vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    1. **URL de connexion**.

    1. **Identificateur Azure AD**.

    1. **URL de déconnexion**.

### <a name="configure-predictix-price-reporting-single-sign-on"></a>Configurer l’authentification unique Predictix Price Reporting

Pour configurer l’authentification unique côté Predictix Price Reporting, vous devez envoyer le certificat que vous avez téléchargé et les URL que vous avez copiées à partir du portail Azure à l’[équipe du support technique Predictix Price Reporting](https://www.infor.com/company/customer-center/). Celle-ci vérifiera que l’authentification unique SAML est configurée correctement des deux côtés.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet de gauche du portail Azure, sélectionnez **Azure Active Directory**, **Utilisateurs**, puis **Tous les utilisateurs** :

    ![Sélectionner Tous les utilisateurs](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran :

    ![Sélectionner Nouvel utilisateur](common/new-user.png)

3. Dans la boîte de dialogue **Utilisateur**, procédez comme suit.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    1. Dans la zone **Nom**, entrez **BrittaSimon**.
  
    1. Dans la zone **Nom d’utilisateur**, entrez **BrittaSimon@\<yourcompanydomain>.\<extension>** . (Par exemple, BrittaSimon@contoso.com.)

    1. Sélectionnez **Afficher le mot de passe**, puis notez la valeur affichée dans la zone **Mot de passe**.

    1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure AD en lui accordant l’accès à Predictix Price Reporting.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **Predictix Price Reporting**.

    ![Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Predictix Price Reporting**.

    ![Liste des applications](common/all-applications.png)

3. Dans le volet gauche, sélectionnez **Utilisateurs et groupes** :

    ![Sélectionner Utilisateurs et groupes](common/users-groups-blade.png)

4. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Sélectionnez Ajouter un utilisateur](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste des utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste. Cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Affecter**.

### <a name="create-a-predictix-price-reporting-test-user"></a>Créer un utilisateur de test Predictix Price Reporting

Ensuite, vous devez créer un utilisateur nommé Britta Simon dans Predictix Price Reporting. Collaborez avec l’[équipe du support technique Predictix Price Reporting](https://www.infor.com/company/customer-center/) pour ajouter des utilisateurs. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Vous devez maintenant tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous sélectionnez la vignette Predictix Price Reporting dans le volet d’accès, vous devez être connecté automatiquement à l’instance Predictix Price Reporting pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations, consultez [Accéder aux applications du portail Mes applications et les utiliser](https://support.microsoft.com/account-billing/sign-in-and-start-apps-from-the-my-apps-portal-2f3b1bae-0e5a-4a86-a33e-876fbd2a4510).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)