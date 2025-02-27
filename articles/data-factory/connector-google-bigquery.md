---
title: Copier les données de Google BigQuery
titleSuffix: Azure Data Factory & Azure Synapse
description: Découvrez comment utiliser l’activité de copie dans un pipeline Azure Data Factory ou Synapse Analytics pour copier des données de Google BigQuery vers des banques de données réceptrices prises en charge.
ms.author: jianleishen
author: jianleishen
ms.service: data-factory
ms.subservice: data-movement
ms.topic: conceptual
ms.custom: synapse
ms.date: 09/09/2021
ms.openlocfilehash: aa59787675870483941f271778bbf5b907467668
ms.sourcegitcommit: 0770a7d91278043a83ccc597af25934854605e8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/13/2021
ms.locfileid: "124815136"
---
# <a name="copy-data-from-google-bigquery-using-azure-data-factory-or-synapse-analytics"></a>Copier des données à partir de Google BigQuery avec Azure Data Factory ou Synapse Analytics
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article décrit comment utiliser l’activité de copie dans des pipelines Azure Data Factory et Azure Synapse Analytics pour copier des données à partir de Google BigQuery. Il s’appuie sur l’article [Vue d’ensemble de l’activité de copie](copy-activity-overview.md).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur Google BigQuery est pris en charge pour les activités suivantes :

- [Activité Copy](copy-activity-overview.md) avec [prise en charge de la matrice source/du récepteur](copy-activity-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)

Vous pouvez copier les données depuis Google BigQuery vers toute banque de données réceptrice prise en charge. Pour obtenir la liste des banques de données prises en charge en tant que sources ou récepteurs par l’activité de copie, consultez le tableau [banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Le service fournit un pilote intégré pour permettre la connectivité. Vous n’avez donc pas besoin d’installer manuellement un pilote pour utiliser ce connecteur.

>[!NOTE]
>Ce connecteur Google BigQuery repose sur les API BigQuery. N’oubliez pas que BigQuery limite le taux maximal de requêtes entrantes et applique des quotas appropriés sur une base par projet. Reportez-vous à [Quotas et limites - requêtes d’API](https://cloud.google.com/bigquery/quotas#api_requests). Assurez-vous que vous ne déclenchez pas trop de demandes simultanées pour le compte.

## <a name="get-started"></a>Bien démarrer

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

## <a name="create-a-linked-service-to-google-bigquery-using-ui"></a>Créer un service lié à Google BigQuery à l’aide de l’interface utilisateur

Suivez les étapes suivantes pour créer un service lié à Google BigQuery dans l’interface utilisateur du portail Azure.

1. Accédez à l’onglet Gérer dans votre espace de travail Azure Data Factory ou Synapse, sélectionnez Services liés, puis cliquez sur Nouveau :

    # <a name="azure-data-factory"></a>[Azure Data Factory](#tab/data-factory).

    :::image type="content" source="media/doc-common-process/new-linked-service.png" alt-text="Capture d’écran montrant la création d’un service lié avec l’interface utilisateur Azure Data Factory.":::

    # <a name="azure-synapse"></a>[Azure Synapse](#tab/synapse-analytics)

    :::image type="content" source="media/doc-common-process/new-linked-service-synapse.png" alt-text="Capture d’écran de la création d’un service lié avec l’interface utilisateur Azure Synapse.":::

2. Recherchez Google et sélectionnez le connecteur Google BigQuery.

    :::image type="content" source="media/connector-google-bigquery/google-bigquery-connector.png" alt-text="Capture d’écran du connecteur Google BigQuery.":::    

1. Configurez les informations du service, testez la connexion et créez le nouveau service lié.

    :::image type="content" source="media/connector-google-bigquery/configure-google-bigquery-linked-service.png" alt-text="Capture d’écran de la configuration du service lié pour Google BigQuery.":::

## <a name="connector-configuration-details"></a>Informations de configuration du connecteur

Les sections suivantes fournissent des informations détaillées sur les propriétés utilisées pour définir les entités spécifiques du connecteur Google BigQuery.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié Google BigQuery sont les suivantes.

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur **GoogleBigQuery**. | Oui |
| project | L’ID du projet BigQuery par défaut sur lequel exécuter la requête.  | Oui |
| additionalProjects | Liste séparée par des virgules des ID de projets BigQuery publics accessibles.  | Non |
| requestGoogleDriveScope | Pour demander l’accès à Google Drive. Autoriser l’accès à Google Drive active la prise en charge des tables fédérées qui combinent les données BigQuery avec les données issues de Google Drive. La valeur par défaut est **false**.  | Non |
| authenticationType | Mécanisme d’authentification OAuth 2.0 utilisé pour l’authentification. ServiceAuthentication ne peut être utilisé que sur un runtime d’intégration auto-hébergé. <br/>Les valeurs autorisées sont **UserAuthentication** et **ServiceAuthentication**. Reportez-vous aux sections suivant ce tableau pour accéder à d’autres propriétés et à des exemples JSON sur ces types d’authentification. | Oui |

### <a name="using-user-authentication"></a>Utiliser l’authentification utilisateur

Définissez la valeur de la propriété « authenticationType » sur **UserAuthentication** et spécifiez les propriétés suivantes ainsi que les propriétés génériques décrites dans la section précédente :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| clientId | ID de l’application utilisée pour générer le jeton d’actualisation. | Non |
| clientSecret | Secret de l’application utilisée pour générer le jeton d’actualisation. Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Non |
| refreshToken | Le jeton d’actualisation obtenu de Google servant à autoriser l’accès à BigQuery. Découvrez comment en obtenir un en consultant [Obtention de jetons d’accès OAuth 2.0](https://developers.google.com/identity/protocols/OAuth2WebServer#obtainingaccesstokens) et [ce blog de communauté](https://jpd.ms/getting-your-bigquery-refresh-token-for-azure-datafactory-f884ff815a59). Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Non |

**Exemple :**

```json
{
    "name": "GoogleBigQueryLinkedService",
    "properties": {
        "type": "GoogleBigQuery",
        "typeProperties": {
            "project" : "<project ID>",
            "additionalProjects" : "<additional project IDs>",
            "requestGoogleDriveScope" : true,
            "authenticationType" : "UserAuthentication",
            "clientId": "<id of the application used to generate the refresh token>",
            "clientSecret": {
                "type": "SecureString",
                "value":"<secret of the application used to generate the refresh token>"
            },
            "refreshToken": {
                "type": "SecureString",
                "value": "<refresh token>"
            }
        }
    }
}
```

### <a name="using-service-authentication"></a>Utiliser l’authentification du service

Définissez la valeur de la propriété « authenticationType » sur **ServiceAuthentication** et spécifiez les propriétés suivantes ainsi que les propriétés génériques décrites dans la section précédente. Ce type d’authentification ne peut être utilisé que sur un runtime d’intégration auto-hébergé.

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| email | ID d’e-mail du compte de service utilisé pour ServiceAuthentication. Il ne peut être utilisé que sur un runtime d’intégration auto-hébergé.  | Non |
| keyFilePath | Chemin complet du fichier de clé .p12 utilisé pour authentifier l’adresse e-mail du compte de service. | Non |
| trustedCertPath | Chemin complet du fichier .pem qui contient les certificats d'autorité de certification approuvés utilisés pour vérifier le serveur lorsque vous vous connectez via TLS. Cette propriété ne peut être définie que lorsque vous utilisez TLS sur le runtime d'intégration auto-hébergé. Valeur par défaut : le fichier cacerts.pem installé avec le runtime d’intégration.  | Non |
| useSystemTrustStore | Indique s’il faut utiliser un certificat d’autorité de certification provenant du magasin de confiance du système ou d’un fichier .pem spécifié. La valeur par défaut est **false**.  | Non |

**Exemple :**

```json
{
    "name": "GoogleBigQueryLinkedService",
    "properties": {
        "type": "GoogleBigQuery",
        "typeProperties": {
            "project" : "<project id>",
            "requestGoogleDriveScope" : true,
            "authenticationType" : "ServiceAuthentication",
            "email": "<email>",
            "keyFilePath": "<.p12 key path on the IR machine>"
        },
        "connectVia": {
            "referenceName": "<name of Self-hosted Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Jeux de données](concepts-datasets-linked-services.md). Cette section fournit la liste des propriétés prises en charge par le jeu de données Google BigQuery.

Pour copier des données à partir de Google BigQuery, définissez la propriété type du jeu de données sur **GoogleBigQueryObject**. Les propriétés prises en charge sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type du jeu de données doit être définie sur : **GoogleBigQueryObject** | Oui |
| dataset | Nom du jeu de données Google BigQuery. |Non (si « query » dans la source de l’activité est spécifié)  |
| table | Nom de la table. |Non (si « query » dans la source de l’activité est spécifié)  |
| tableName | Nom de la table. Cette propriété est prise en charge pour la compatibilité descendante. Pour les nouvelles charges de travail, utilisez `dataset` et `table`. | Non (si « query » dans la source de l’activité est spécifié) |

**Exemple**

```json
{
    "name": "GoogleBigQueryDataset",
    "properties": {
        "type": "GoogleBigQueryObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<GoogleBigQuery linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par le type de source Google BigQuery.

### <a name="googlebigquerysource-as-a-source-type"></a>GoogleBigQuerySource en tant que type de source

Pour copier des données à partir de Google BigQuery, définissez le type de source sur **GoogleBigQuerySource** dans l’activité de copie. Les propriétés suivantes sont prises en charge dans la section **source** de l’activité de copie.

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type de la source d’activité de copie doit être définie sur **GoogleBigQuerySource**. | Oui |
| query | Utiliser la requête SQL personnalisée pour lire les données. par exemple `"SELECT * FROM MyTable"`. | Non (si « tableName » est spécifié dans dataset) |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromGoogleBigQuery",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<GoogleBigQuery input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "GoogleBigQuerySource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour en savoir plus sur les propriétés, consultez [Activité Lookup](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Étapes suivantes
Consultez les [magasins de données pris en charge](copy-activity-overview.md#supported-data-stores-and-formats) pour obtenir la liste des sources et magasins de données pris en charge en tant que récepteurs par l’activité de copie.
