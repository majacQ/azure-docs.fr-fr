---
title: Se connecter à Office 365 Outlook
description: Automatiser les tâches et les workflows qui gèrent la messagerie, les contacts et les calendriers dans Office 365 avec Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, azla
ms.topic: article
ms.date: 08/11/2021
tags: connectors
ms.openlocfilehash: b60565cec180242535402daee27ed5018505727a
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122562984"
---
# <a name="connect-to-office-365-outlook-using-azure-logic-apps"></a>Se connecter à Office 365 Outlook à l’aide d’Azure Logic Apps

Avec [Azure Logic Apps](../logic-apps/logic-apps-overview.md) et le [connecteur Office 365 Outlook](/connectors/office365connector/), vous pouvez créer des tâches et des workflows automatisés qui gèrent votre compte professionnel ou scolaire en générant des applications logiques. Par exemple, vous pouvez automatiser les tâches suivantes :

* Recevoir, envoyer et répondre à des e-mails
* Planifier des réunions sur votre calendrier
* Ajouter et modifier des contacts

Vous pouvez utiliser n’importe quel déclencheur pour démarrer votre workflow, par exemple, à l’arrivée d’un nouvel e-mail, lorsqu’un élément de calendrier est mis à jour ou quand un événement se produit dans un autre service, comme Salesforce. Vous pouvez utiliser des actions qui répondent à l’événement déclencheur, par exemple, envoyer un e-mail ou créer un événement de calendrier.

## <a name="prerequisites"></a>Prérequis

* Les informations d’identification de compte Microsoft Office 365 utilisées avec Outlook quand vous vous connectez avec un [compte professionnel ou scolaire](https://support.microsoft.com/office/what-account-to-use-with-office-and-you-need-one-914e6610-2763-47ac-ab36-602a81068235#bkmk_msavsworkschool).

  Vous avez besoin de ces informations d’identification pour autoriser votre workflow à accéder à votre compte Outlook.

  > [!NOTE]
  > Si vous disposez d’un compte @outlook.com ou @hotmail.com, utilisez le [connecteur Outlook.com](../connectors/connectors-create-api-outlook.md). Pour vous connecter à Outlook avec un autre compte d’utilisateur, tel qu’un compte de service, consultez [Se connecter avec d’autres comptes](#connect-using-other-accounts).
  >
  > Si vous utilisez [Microsoft Azure géré par 21Vianet](https://portal.azure.cn), l’authentification Azure AD (Active Directory) fonctionne uniquement avec un compte pour Microsoft Office 365 géré par 21Vianet (.cn) et non les comptes .com.

* Un compte et un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* L’application logique à partir de laquelle vous souhaitez accéder à votre compte Outlook. Pour démarrer votre workflow avec un déclencheur Office 365 Outlook, vous devez disposer d’une [application logique vide](../logic-apps/quickstart-create-first-logic-app-workflow.md). Pour ajouter une action Office 365 Outlook à votre workflow, votre workflow d’application logique doit déjà disposer d’un déclencheur.

## <a name="connector-reference"></a>Référence de connecteur

Pour plus d’informations techniques sur ce connecteur, notamment au sujet des déclencheurs, des actions et des limites décrits dans son fichier Swagger, consultez la [page de référence du connecteur](/connectors/office365/).

## <a name="add-a-trigger"></a>Ajouter un déclencheur

Un [déclencheur](../logic-apps/logic-apps-overview.md#logic-app-concepts) désigne un événement qui démarre le workflow dans votre application logique. Cet exemple d’application logique utilise un déclencheur « d’interrogation » qui recherche tous les événements de calendrier mis à jour dans votre compte e-mail, en fonction de l’intervalle et de la fréquence spécifiés.

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique vide dans le concepteur visuel.

1. Dans la zone de recherche, entrez `office 365 outlook` en guise de filtre. Cet exemple sélectionne **Lorsqu’un événement à venir démarre bientôt**.

   ![Sélectionner un déclencheur pour démarrer votre application logique](./media/connectors-create-api-office365-outlook/office365-trigger.png)

1. Si vous n’avez pas encore de connexion active vers votre compte Outlook, vous êtes invité à vous connecter et à créer cette connexion. Pour vous connecter à Outlook avec un autre compte d’utilisateur, tel qu’un compte de service, consultez [Se connecter avec d’autres comptes](#connect-using-other-accounts). Sinon, entrez les informations pour les propriétés du déclencheur.

   > [!NOTE]
   > Votre connexion n’expire pas tant qu’elle n’est pas révoquée, même si vous modifiez vos informations d’identification de connexion. Pour plus d’informations, consultez [Durée de vie des jetons configurables dans Azure Active Directory](../active-directory/develop/active-directory-configurable-token-lifetimes.md).

   Cet exemple sélectionne le calendrier que le déclencheur vérifie, par exemple :

   ![Configurer les propriétés du déclencheur](./media/connectors-create-api-office365-outlook/select-calendar.png)

1. Dans le déclencheur, définissez les valeurs de **Fréquence** et **Intervalle**. Pour ajouter d’autres propriétés de déclencheur disponibles, telles que **Fuseau horaire**, sélectionnez ces propriétés dans la liste **Ajouter un nouveau paramètre**.

   Par exemple, si vous souhaitez que le déclencheur vérifie le calendrier toutes les 15 minutes, définissez **Fréquence** sur **Minute**, et **Intervalle** sur `15`.

   ![Définir la fréquence et l’intervalle pour le déclencheur](./media/connectors-create-api-office365-outlook/calendar-settings.png)

1. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

À présent, ajoutez une action qui s’exécute après l’activation du déclencheur. Par exemple, vous pouvez ajouter l’action Twilio **Envoyer un message**, qui envoie un texte lorsqu’un événement de calendrier démarre dans 15 minutes.

## <a name="add-an-action"></a>Ajouter une action

Une [action](../logic-apps/logic-apps-overview.md#logic-app-concepts) est une opération qui est définie par le workflow dans votre application logique. Cet exemple d’application logique crée un contact dans Office 365 Outlook. Vous pouvez utiliser la sortie d’un autre déclencheur ou d’une autre action pour créer le contact. Par exemple, supposez que votre application logique utilise le déclencheur Salesforce **Lorsqu’un enregistrement est créé**. Vous pouvez ajouter l’action Office 365 Outlook **Créer un contact** et utiliser les sorties du déclencheur pour créer le contact.

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le concepteur visuel.

1. Pour ajouter une action comme dernière étape à votre workflow, sélectionnez **Nouvelle étape**.

   Pour ajouter une action entre des étapes, placez votre pointeur au-dessus de la flèche qui les sépare. Cliquez sur le signe ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Dans la zone de recherche, entrez `office 365 outlook` en guise de filtre. Cet exemple sélectionne **Créer un contact**.

   ![Sélectionner l’action à exécuter dans votre application logique](./media/connectors-create-api-office365-outlook/office365-actions.png) 

1. Si vous n’avez pas encore de connexion active vers votre compte Outlook, vous êtes invité à vous connecter et à créer cette connexion. Pour vous connecter à Outlook avec un autre compte d’utilisateur, tel qu’un compte de service, consultez [Se connecter avec d’autres comptes](#connect-using-other-accounts). Sinon, entrez les informations pour les propriétés de l’action.

   > [!NOTE]
   > Votre connexion n’expire pas tant qu’elle n’est pas révoquée, même si vous modifiez vos informations d’identification de connexion. Pour plus d’informations, consultez [Durée de vie des jetons configurables dans Azure Active Directory](../active-directory/develop/active-directory-configurable-token-lifetimes.md).

   Cet exemple sélectionne le dossier de contacts dans lequel l’action crée le contact, par exemple :

   ![Configurer les propriétés de l’action](./media/connectors-create-api-office365-outlook/select-contacts-folder.png)

   Pour ajouter d’autres propriétés d’action disponibles, sélectionnez ces propriétés dans la liste **Ajouter un nouveau paramètre**.

1. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

<a name="connect-using-other-accounts"></a>

## <a name="connect-using-other-accounts"></a>Se connecter avec d’autres comptes

Si vous essayez de vous connecter à Outlook à l’aide d’un compte différent de celui qui est actuellement connecté à Azure, vous risquez d’obtenir des erreurs [d’authentification unique (SSO)](../active-directory/manage-apps/what-is-single-sign-on.md). Ce problème se produit lorsque vous vous connectez au portail Azure avec un compte, mais que vous utilisez un autre compte pour créer la connexion. Le concepteur s'attend à ce que vous utilisiez le compte connecté au portail Azure. Pour résoudre ce problème, vous disposez des options suivantes :

* Configurez l'autre compte avec le rôle de **Contributeur** dans le groupe de ressources de votre application logique.

  1. Dans le menu du groupe de ressources de votre application logique, sélectionnez **Contrôle d’accès (IAM)** . Configurez l’autre compte avec le rôle **Contributeur**. 
  
     Pour plus d’informations, consultez [Attribuer des rôles Azure en utilisant le portail Azure](../role-based-access-control/role-assignments-portal.md).

  1. Après avoir configuré ce rôle, connectez-vous au portail Azure avec le compte qui dispose désormais des autorisations de contributeur. Vous pouvez maintenant utiliser ce compte pour créer la connexion à Outlook.

* Configurez l’autre compte pour que votre compte professionnel ou scolaire dispose des autorisations « Envoyer en tant que ».

   Si vous disposez d'autorisations d'administrateur, dans la boîte aux lettres du compte de service, configurez votre compte professionnel ou scolaire avec les autorisations **Envoyer en tant que** ou **Envoyer de la part de**. Pour plus d’informations, consultez [Accorder des autorisations de boîte aux lettres à un autre utilisateur - Aide de l’administrateur](/microsoft-365/admin/add-users/give-mailbox-permissions-to-another-user). Vous pouvez ensuite créer la connexion à l’aide de votre compte professionnel ou scolaire. Désormais, dans les déclencheurs ou les actions où vous pouvez spécifier l’expéditeur, vous pouvez utiliser l’adresse e-mail du compte de service.

   Par exemple, l’action **Envoyer un e-mail** comporte un paramètre facultatif, **De (Envoyer en tant que)** , que vous pouvez ajouter à l’action, et l’adresse e-mail de votre compte de service servira d’expéditeur. Pour ajouter ce paramètre, procédez comme suit :

   1. Dans l’action **Envoyer un e-mail**, ouvrez la liste **Ajouter un paramètre**, puis sélectionnez le paramètre **De (Envoyer en tant que)** .

   1. Une fois que le paramètre apparaît sur l’action, entrez l’adresse e-mail du compte de service.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)
