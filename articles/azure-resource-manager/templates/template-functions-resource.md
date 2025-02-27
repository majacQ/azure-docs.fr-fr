---
title: Fonctions de modèle - Ressources
description: Décrit les fonctions à utiliser dans un modèle Azure Resource Manager (modèle ARM) pour récupérer des valeurs sur les ressources.
ms.topic: conceptual
ms.date: 09/09/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 24bf175f6b6099a1265e411daf51913c3673d6f2
ms.sourcegitcommit: f6e2ea5571e35b9ed3a79a22485eba4d20ae36cc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/24/2021
ms.locfileid: "128613039"
---
# <a name="resource-functions-for-arm-templates"></a>Fonctions de ressource pour les modèles ARM

Resource Manager fournit les fonctions suivantes pour obtenir des valeurs de ressource dans votre modèle Azure Resource Manager (modèle ARM) :

* [extensionResourceId](#extensionresourceid)
* [list*](#list)
* [pickZones](#pickzones)
* [providers (déconseillé)](#providers)
* [reference](#reference)
* [resourceGroup](#resourcegroup)
* [resourceId](#resourceid)
* [subscription](#subscription)
* [subscriptionResourceId](#subscriptionresourceid)
* [tenantResourceId](#tenantresourceid)

Pour obtenir des valeurs de paramètres, de variables ou du déploiement actuel, consultez [Fonctions de valeur de déploiement](template-functions-deployment.md).

## <a name="extensionresourceid"></a>extensionResourceId

`extensionResourceId(baseResourceId, resourceType, resourceName1, [resourceName2], ...)`

Retourne l’ID d’une [ressource d’extension](../management/extension-resource-types.md), un type de ressource qui s’applique à une autre ressource pour y ajouter des fonctionnalités.

### <a name="parameters"></a>Paramètres

| Paramètre | Obligatoire | Type | Description |
|:--- |:--- |:--- |:--- |
| baseResourceId |Oui |string |ID de la ressource à laquelle s’applique la ressource d’extension. |
| resourceType |Oui |string |Type de ressource d’extension, y compris l'espace de noms du fournisseur de ressources. |
| nom_ressource1 |Oui |string |Nom de la ressource d’extension. |
| nom_ressource2 |Non |string |Segment de nom de ressource suivant si nécessaire. |

Continuez à ajouter des noms de ressource en paramètres lorsque le type de ressource contient plus de segments.

### <a name="return-value"></a>Valeur retournée

Le format de base de l’ID de ressource retourné par cette fonction est le suivant :

```json
{scope}/providers/{extensionResourceProviderNamespace}/{extensionResourceType}/{extensionResourceName}
```

Le segment d’étendue varie en fonction de la ressource étendue. Par exemple, l’ID d’un abonnement a des segments différents de l’ID d’un groupe de ressources.

Lorsque la ressource d’extension est appliquée à une **ressource**, l’ID de la ressource est retourné au format suivant :

```json
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{baseResourceProviderNamespace}/{baseResourceType}/{baseResourceName}/providers/{extensionResourceProviderNamespace}/{extensionResourceType}/{extensionResourceName}
```

Lorsque la ressource d’extension est appliquée à un **groupe de ressources**, le format retourné est le suivant :

```json
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{extensionResourceProviderNamespace}/{extensionResourceType}/{extensionResourceName}
```

Un exemple d’utilisation de cette fonction avec un groupe de ressources est présenté dans la section suivante.

Lorsque la ressource d’extension est appliquée à un **abonnement**, le format retourné est le suivant :

```json
/subscriptions/{subscriptionId}/providers/{extensionResourceProviderNamespace}/{extensionResourceType}/{extensionResourceName}
```

Lorsque la ressource d’extension est appliquée à un **groupe d’administration**, le format retourné est le suivant :

```json
/providers/Microsoft.Management/managementGroups/{managementGroupName}/providers/{extensionResourceProviderNamespace}/{extensionResourceType}/{extensionResourceName}
```

Un exemple d’utilisation de cette fonction avec un groupe d'administration est présenté dans la section suivante.

### <a name="extensionresourceid-example"></a>Exemple extensionResourceId

L’exemple suivant retourne l’ID de la ressource pour un verrou de groupe de ressources.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/functions/resource/extensionresourceid.json":::

Une définition de stratégie personnalisée déployée sur un groupe d’administration est implémentée en tant que ressource d’extension. Pour créer et affecter une stratégie, déployez le modèle suivant dans un groupe d’administration.

:::code language="json" source="~/quickstart-templates/managementgroup-deployments/mg-policy/azuredeploy.json":::

Les définitions de stratégie intégrées sont des ressources de niveau locataire. Pour obtenir un exemple de déploiement d’une définition de stratégie intégrée, consultez [tenantResourceId](#tenantresourceid).

<a id="listkeys"></a>
<a id="list"></a>

## <a name="list"></a>list*

`list{Value}(resourceName or resourceIdentifier, apiVersion, functionValues)`

La syntaxe de cette fonction varie en fonction du nom des opérations de liste. Chaque implémentation retourne des valeurs pour le type de ressource qui prend en charge une opération de liste. Le nom de l’opération doit commencer par `list` et peut avoir un suffixe. Voici quelques utilisations courantes : `list`, `listKeys`, `listKeyValue` et `listSecrets`.

### <a name="parameters"></a>Paramètres

| Paramètre | Obligatoire | Type | Description |
|:--- |:--- |:--- |:--- |
| nom_ressource ou identificateur_ressource |Oui |string |Identificateur unique pour la ressource. |
| apiVersion |Oui |string |Version d'API de l'état d'exécution des ressources. En règle générale, au format, **aaaa-mm-jj**. |
| functionValues |Non |object | Objet qui contient les valeurs de la fonction. Fournissez uniquement cet objet pour les fonctions qui prennent en charge la réception d’un objet avec des valeurs de paramètre, comme **listAccountSas** sur un compte de stockage. Un exemple de transmission de valeurs de fonction est illustré dans cet article. |

### <a name="valid-uses"></a>Utilisations valides

Les fonctions de liste peuvent être utilisés dans les propriétés d’une définition de ressource. N’utilisez pas de fonction de liste qui expose des informations sensibles dans la section de sortie d’un modèle. Les valeurs de sortie sont stockées dans l’historique de déploiement et peuvent être récupérées par un utilisateur malveillant.

Quand elles sont utilisées avec une [itération de propriété](copy-properties.md), vous pouvez utiliser les fonctions list pour `input`, car l’expression est affectée à la propriété de ressource. Vous ne pouvez pas les utiliser avec `count`, car le nombre doit être déterminé avant que la fonction list ne soit résolue.

### <a name="implementations"></a>Implémentations

Les utilisations possibles de list* sont indiquées dans le tableau suivant.

| Type de ressource | Nom de la fonction |
| ------------- | ------------- |
| Microsoft.Addons/supportProviders | listsupportplaninfo |
| Microsoft.AnalysisServices/servers | [listGatewayStatus](/rest/api/analysisservices/servers/listgatewaystatus) |
| Microsoft.ApiManagement/service/authorizationServers | [listSecrets](/rest/api/apimanagement/2020-06-01-preview/authorization-server/list-secrets) |
| Microsoft.ApiManagement/service/gateways | [listKeys](/rest/api/apimanagement/2020-06-01-preview/gateway/list-keys) |
| Microsoft.ApiManagement/service/identityProviders | [listSecrets](/rest/api/apimanagement/2020-06-01-preview/identity-provider/list-secrets) |
| Microsoft.ApiManagement/service/namedValues | [listValue](/rest/api/apimanagement/2020-06-01-preview/named-value/list-value) |
| Microsoft.ApiManagement/service/openidConnectProviders | [listSecrets](/rest/api/apimanagement/2020-06-01-preview/openid-connect-provider/list-secrets) |
| Microsoft.ApiManagement/service/subscriptions | [listSecrets](/rest/api/apimanagement/2020-06-01-preview/subscription/list-secrets) |
| Microsoft.AppConfiguration/configurationStores | [Listkeys](/rest/api/appconfiguration/configurationstores/listkeys) |
| Microsoft.AppPlatform/Spring | [listTestKeys](/rest/api/azurespringcloud/services/listtestkeys) |
| Microsoft.Automation/automationAccounts | [listKeys](/rest/api/automation/keys/listbyautomationaccount) |
| Microsoft.Batch/batchAccounts | [listkeys](/rest/api/batchmanagement/batchaccount/getkeys) |
| Microsoft.BatchAI/workspaces/experiments/jobs | listoutputfiles |
| Microsoft.Blockchain/blockchainMembers | [listApiKeys](/rest/api/blockchain/2019-06-01-preview/blockchainmembers/listapikeys) |
| Microsoft.Blockchain/blockchainMembers/transactionNodes | [listApiKeys](/rest/api/blockchain/2019-06-01-preview/transactionnodes/listapikeys) |
| Microsoft.BotService/botServices/channels | [listChannelWithKeys](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/botservice/resource-manager/Microsoft.BotService/stable/2020-06-02/botservice.json#L553) |
| Microsoft.Cache/redis | [listKeys](/rest/api/redis/redis/listkeys) |
| Microsoft.CognitiveServices/accounts | [listKeys](/rest/api/cognitiveservices/accountmanagement/accounts/listkeys) |
| Microsoft.ContainerRegistry/registries | [listBuildSourceUploadUrl](/rest/api/containerregistry/registries%20(tasks)/get-build-source-upload-url) |
| Microsoft.ContainerRegistry/registries | [listCredentials](/rest/api/containerregistry/registries/listcredentials) |
| Microsoft.ContainerRegistry/registries | [listUsages](/rest/api/containerregistry/registries/listusages) |
| Microsoft.ContainerRegistry/registries/agentpools | listQueueStatus |
| Microsoft.ContainerRegistry/registries/buildTasks | listSourceRepositoryProperties |
| Microsoft.ContainerRegistry/registries/buildTasks/steps | listBuildArguments |
| Microsoft.ContainerRegistry/registries/taskruns | listDetails |
| Microsoft.ContainerRegistry/registries/webhooks | [listEvents](/rest/api/containerregistry/webhooks/listevents) |
| Microsoft.ContainerRegistry/registries/runs | [listLogSasUrl](/rest/api/containerregistry/runs/getlogsasurl) |
| Microsoft.ContainerRegistry/registries/tasks | [listDetails](/rest/api/containerregistry/tasks/getdetails) |
| Microsoft.ContainerService/managedClusters | [listClusterAdminCredential](/rest/api/aks/managedclusters/listclusteradmincredentials) |
| Microsoft.ContainerService/managedClusters | [listClusterMonitoringUserCredential](/rest/api/aks/managedclusters/listclustermonitoringusercredentials) |
| Microsoft.ContainerService/managedClusters | [listClusterUserCredential](/rest/api/aks/managedclusters/listclusterusercredentials) |
| Microsoft.ContainerService/managedClusters/accessProfiles | [listCredential](/rest/api/aks/managedclusters/getaccessprofile) |
| Microsoft.DataBox/jobs | listCredentials |
| Microsoft.DataFactory/datafactories/gateways | listauthkeys |
| Microsoft.DataFactory/factories/integrationruntimes | [listauthkeys](/rest/api/datafactory/integrationruntimes/listauthkeys) |
| Microsoft.DataLakeAnalytics/accounts/storageAccounts/Containers | [listSasTokens](/rest/api/datalakeanalytics/storageaccounts/listsastokens) |
| Microsoft.DataShare/accounts/shares | [listSynchronizations](/rest/api/datashare/2020-09-01/shares/listsynchronizations) |
| Microsoft.DataShare/accounts/shareSubscriptions | [listSourceShareSynchronizationSettings](/rest/api/datashare/2020-09-01/sharesubscriptions/listsourcesharesynchronizationsettings) |
| Microsoft.DataShare/accounts/shareSubscriptions | [listSynchronizationDetails](/rest/api/datashare/2020-09-01/sharesubscriptions/listsynchronizationdetails) |
| Microsoft.DataShare/accounts/shareSubscriptions | [listSynchronizations](/rest/api/datashare/2020-09-01/sharesubscriptions/listsynchronizations) |
| Microsoft.Devices/iotHubs | [listkeys](/rest/api/iothub/iothubresource/listkeys) |
| Microsoft.Devices/iotHubs/iotHubKeys | [listkeys](/rest/api/iothub/iothubresource/getkeysforkeyname) |
| Microsoft.Devices/provisioningServices/keys | [listkeys](/rest/api/iot-dps/iotdpsresource/listkeysforkeyname) |
| Microsoft.Devices/provisioningServices | [listkeys](/rest/api/iot-dps/iotdpsresource/listkeys) |
| Microsoft.DevTestLab/labs | [ListVhds](/rest/api/dtl/labs/listvhds) |
| Microsoft.DevTestLab/labs/schedules | [ListApplicable](/rest/api/dtl/schedules/listapplicable) |
| Microsoft.DevTestLab/labs/users/serviceFabrics | [ListApplicableSchedules](/rest/api/dtl/servicefabrics/listapplicableschedules) |
| Microsoft.DevTestLab/labs/virtualMachines | [ListApplicableSchedules](/rest/api/dtl/virtualmachines/listapplicableschedules) |
| Microsoft.DocumentDB/databaseAccounts | [listConnectionStrings](/rest/api/cosmos-db-resource-provider/2021-04-15/database-accounts/list-connection-strings) |
| Microsoft.DocumentDB/databaseAccounts | [listKeys](/rest/api/cosmos-db-resource-provider/2021-04-15/database-accounts/list-keys) |
| Microsoft.DocumentDB/databaseAccounts/notebookWorkspaces | [listConnectionInfo](/rest/api/cosmos-db-resource-provider/2021-04-15/notebook-workspaces/list-connection-info) |
| Microsoft.DomainRegistration | [listDomainRecommendations](/rest/api/appservice/domains/listrecommendations) |
| Microsoft.DomainRegistration/topLevelDomains | [listAgreements](/rest/api/appservice/topleveldomains/listagreements) |
| Microsoft.EventGrid/domains | [listKeys](/rest/api/eventgrid/version2020-06-01/domains/listsharedaccesskeys) |
| Microsoft.EventGrid/topics | [listKeys](/rest/api/eventgrid/version2020-06-01/topics/listsharedaccesskeys) |
| Microsoft.EventHub/namespaces/authorizationRules | [listkeys](/rest/api/eventhub) |
| Microsoft.EventHub/namespaces/disasterRecoveryConfigs/authorizationRules | [listkeys](/rest/api/eventhub) |
| Microsoft.EventHub/namespaces/eventhubs/authorizationRules | [listkeys](/rest/api/eventhub) |
| Microsoft.ImportExport/jobs | [listBitLockerKeys](/rest/api/storageimportexport/bitlockerkeys/list) |
| Microsoft.Kusto/Clusters/Databases | [ListPrincipals](/rest/api/azurerekusto/databases/listprincipals) |
| Microsoft.LabServices/users | [ListEnvironments](/rest/api/labservices/globalusers/listenvironments) |
| Microsoft.LabServices/users | [ListLabs](/rest/api/labservices/globalusers/listlabs) |
| Microsoft.Logic/integrationAccounts/agreements | [listContentCallbackUrl](/rest/api/logic/agreements/listcontentcallbackurl) |
| Microsoft.Logic/integrationAccounts/assemblies | [listContentCallbackUrl](/rest/api/logic/integrationaccountassemblies/listcontentcallbackurl) |
| Microsoft.Logic/integrationAccounts | [listCallbackUrl](/rest/api/logic/integrationaccounts/getcallbackurl) |
| Microsoft.Logic/integrationAccounts | [listKeyVaultKeys](/rest/api/logic/integrationaccounts/listkeyvaultkeys) |
| Microsoft.Logic/integrationAccounts/maps | [listContentCallbackUrl](/rest/api/logic/maps/listcontentcallbackurl) |
| Microsoft.Logic/integrationAccounts/partners | [listContentCallbackUrl](/rest/api/logic/partners/listcontentcallbackurl) |
| Microsoft.Logic/integrationAccounts/schemas | [listContentCallbackUrl](/rest/api/logic/schemas/listcontentcallbackurl) |
| Microsoft.Logic/workflows | [listCallbackUrl](/rest/api/logic/workflows/listcallbackurl) |
| Microsoft.Logic/workflows | [listSwagger](/rest/api/logic/workflows/listswagger) |
| Microsoft.Logic/workflows/runs/actions | [listExpressionTraces](/rest/api/logic/workflowrunactions/listexpressiontraces) |
| Microsoft.Logic/workflows/runs/actions/repetitions | [listExpressionTraces](/rest/api/logic/workflowrunactionrepetitions/listexpressiontraces) |
| Microsoft.Logic/workflows/triggers | [listCallbackUrl](/rest/api/logic/workflowtriggers/listcallbackurl) |
| Microsoft.Logic/workflows/versions/triggers | [listCallbackUrl](/rest/api/logic/workflowversions/listcallbackurl) |
| Microsoft.MachineLearning/webServices | [listkeys](/rest/api/machinelearning/webservices/listkeys) |
| Microsoft.MachineLearning/Workspaces | listworkspacekeys |
| Microsoft.MachineLearningServices/workspaces/computes | [listKeys](/rest/api/azureml/compute/list-keys) |
| Microsoft.MachineLearningServices/workspaces/computes | [listNodes](/rest/api/azureml/compute/list-nodes) |
| Microsoft.MachineLearningServices/workspaces | [listKeys](/rest/api/azureml/workspaces/list-keys) |
| Microsoft.Maps/accounts | [listKeys](/rest/api/maps-management/accounts/listkeys) |
| Microsoft.Media/mediaservices/assets | [listContainerSas](/rest/api/media/assets/listcontainersas) |
| Microsoft.Media/mediaservices/assets | [listStreamingLocators](/rest/api/media/assets/liststreaminglocators) |
| Microsoft.Media/mediaservices/streamingLocators | [listContentKeys](/rest/api/media/streaminglocators/listcontentkeys) |
| Microsoft.Media/mediaservices/streamingLocators | [listPaths](/rest/api/media/streaminglocators/listpaths) |
| Microsoft.Network/applicationSecurityGroups | listIpConfigurations |
| Microsoft.NotificationHubs/Namespaces/authorizationRules | [listkeys](/rest/api/notificationhubs/namespaces/listkeys) |
| Microsoft.NotificationHubs/Namespaces/NotificationHubs/authorizationRules | [listkeys](/rest/api/notificationhubs/notificationhubs/listkeys) |
| Microsoft.OperationalInsights/workspaces | [list](/rest/api/loganalytics/workspaces/list) |
| Microsoft.OperationalInsights/workspaces | listKeys |
| Microsoft.PolicyInsights/remediations | [listDeployments](/rest/api/policy/remediations/listdeploymentsatresourcegroup) |
| Microsoft.RedHatOpenShift/openShiftClusters | [listCredentials](/rest/api/openshift/openshiftclusters/listcredentials) |
| Microsoft.Relay/namespaces/authorizationRules | [listkeys](/rest/api/relay/namespaces/listkeys) |
| Microsoft.Relay/namespaces/disasterRecoveryConfigs/authorizationRules | listkeys |
| Microsoft.Relay/namespaces/HybridConnections/authorizationRules | [listkeys](/rest/api/relay/hybridconnections/listkeys) |
| Microsoft.Relay/namespaces/WcfRelays/authorizationRules | [listkeys](/rest/api/relay/wcfrelays/listkeys) |
| Microsoft.Search/searchServices | [listAdminKeys](/rest/api/searchmanagement/2021-04-01-preview/admin-keys/get) |
| Microsoft.Search/searchServices | [listQueryKeys](/rest/api/searchmanagement/2021-04-01-preview/query-keys/list-by-search-service) |
| Microsoft.ServiceBus/namespaces/authorizationRules | [listkeys](/rest/api/servicebus/stable/namespaces-authorization-rules/list-keys) |
| Microsoft.ServiceBus/namespaces/disasterRecoveryConfigs/authorizationRules | [listkeys](/rest/api/servicebus/stable/disasterrecoveryconfigs/listkeys) |
| Microsoft.ServiceBus/namespaces/queues/authorizationRules | [listkeys](/rest/api/servicebus/stable/queues-authorization-rules/list-keys) |
| Microsoft.ServiceBus/namespaces/topics/authorizationRules | [listkeys](/rest/api/servicebus/stable/topics%20%E2%80%93%20authorization%20rules/list-keys) |
| Microsoft.SignalRService/SignalR | [listkeys](/rest/api/signalr/signalr/listkeys) |
| Microsoft.Storage/storageAccounts | [listAccountSas](/rest/api/storagerp/storageaccounts/listaccountsas) |
| Microsoft.Storage/storageAccounts | [listkeys](/rest/api/storagerp/storageaccounts/listkeys) |
| Microsoft.Storage/storageAccounts | [listServiceSas](/rest/api/storagerp/storageaccounts/listservicesas) |
| Microsoft.StorSimple/managers/devices | [listFailoverSets](/rest/api/storsimple/devices/listfailoversets) |
| Microsoft.StorSimple/managers/devices | [listFailoverTargets](/rest/api/storsimple/devices/listfailovertargets) |
| Microsoft.StorSimple/managers | [listActivationKey](/rest/api/storsimple/managers/getactivationkey) |
| Microsoft.StorSimple/managers | [listPublicEncryptionKey](/rest/api/storsimple/managers/getpublicencryptionkey) |
| Microsoft.Synapse/workspaces/integrationRuntimes | [listauthkeys](/rest/api/synapse/integrationruntimeauthkeys/list) |
| Microsoft.Web/connectionGateways | ListStatus |
| microsoft.web/connections | listconsentlinks |
| Microsoft.Web/customApis | listWsdlInterfaces |
| microsoft.web/locations | listwsdlinterfaces |
| microsoft.web/apimanagementaccounts/apis/connections | listconnectionkeys |
| microsoft.web/apimanagementaccounts/apis/connections | `listSecrets` |
| microsoft.web/sites/backups | [list](/rest/api/appservice/webapps/listbackups) |
| Microsoft.Web/sites/config | [list](/rest/api/appservice/webapps/listconfigurations) |
| microsoft.web/sites/functions | [listkeys](/rest/api/appservice/webapps/listfunctionkeys)
| microsoft.web/sites/functions | [listSecrets](/rest/api/appservice/webapps/listfunctionsecrets) |
| microsoft.web/sites/hybridconnectionnamespaces/relays | [listkeys](/rest/api/appservice/appserviceplans/listhybridconnectionkeys) |
| microsoft.web/sites | [listsyncfunctiontriggerstatus](/rest/api/appservice/webapps/listsyncfunctiontriggers) |
| microsoft.web/sites/slots/functions | [listSecrets](/rest/api/appservice/webapps/listfunctionsecretsslot) |
| microsoft.web/sites/slots/backups | [list](/rest/api/appservice/webapps/listbackupsslot) |
| Microsoft.Web/sites/slots/config | [list](/rest/api/appservice/webapps/listconfigurationsslot) |
| microsoft.web/sites/slots/functions | [listSecrets](/rest/api/appservice/webapps/listfunctionsecretsslot) |

Pour déterminer les types de ressources qui ont une opération de liste, utilisez les options suivantes :

* Affichez les [opérations d’API REST](/rest/api/) pour un fournisseur de ressources et recherchez les opérations de liste. Par exemple, les comptes de stockage présentent l’[opération listKeys](/rest/api/storagerp/storageaccounts).
* Utilisez la cmdlet PowerShell [Get-AzureRmProviderOperation](/powershell/module/az.resources/get-azprovideroperation). L’exemple ci-dessous obtient toutes les opérations de liste pour les comptes de stockage :

  ```powershell
  Get-AzProviderOperation -OperationSearchString "Microsoft.Storage/*" | where {$_.Operation -like "*list*"} | FT Operation
  ```

* Utilisez la commande Azure CLI suivante pour filtrer uniquement les opérations de liste :

  ```azurecli
  az provider operation show --namespace Microsoft.Storage --query "resourceTypes[?name=='storageAccounts'].operations[].name | [?contains(@, 'list')]"
  ```

### <a name="return-value"></a>Valeur retournée

L’objet retourné varie selon la fonction de liste que vous utilisez. Par exemple, la fonction listKeys pour un compte de stockage retourne le format suivant :

```json
{
  "keys": [
    {
      "keyName": "key1",
      "permissions": "Full",
      "value": "{value}"
    },
    {
      "keyName": "key2",
      "permissions": "Full",
      "value": "{value}"
    }
  ]
}
```

D’autres fonctions de liste ont différents formats de retour. Pour afficher le format d’une fonction, incluez-le dans la section des sorties comme indiqué dans l’exemple de modèle.

### <a name="remarks"></a>Notes

Spécifiez la ressource en utilisant le nom de la ressource ou la [fonction resourceId](#resourceid). Lorsque vous utilisez une fonction de liste dans le modèle qui déploie la ressource référencée, utilisez le nom de la ressource.

Si vous utilisez une fonction `list` dans une ressource qui est déployée conditionnellement, la fonction est évaluée, même si la ressource n’est pas déployée. Vous obtenez une erreur si la fonction `list` fait référence à une ressource qui n’existe pas. Utilisez la fonction `if` pour vous assurer que la fonction est évaluée lors du déploiement de la ressource. Consultez la [fonction if](template-functions-logical.md#if) pour un exemple de modèle qui utilise « if » et « list » avec une ressource déployée de manière conditionnelle.

### <a name="list-example"></a>Exemple de liste

L’exemple suivant utilise listKeys lors de la définition d’une valeur pour [les scripts de déploiement](deployment-script-template.md).

```json
"storageAccountSettings": {
  "storageAccountName": "[variables('storageAccountName')]",
  "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value]"
}
```

L’exemple suivant montre une fonction de liste qui accepte un paramètre. Dans ce cas, la fonction est toujours **listAccountSas**. Passez un objet pour l’heure d’expiration. L’heure d’expiration doit être dans le futur.

```json
"parameters": {
  "accountSasProperties": {
    "type": "object",
    "defaultValue": {
      "signedServices": "b",
      "signedPermission": "r",
      "signedExpiry": "2020-08-20T11:00:00Z",
      "signedResourceTypes": "s"
    }
  }
},
...
"sasToken": "[listAccountSas(parameters('storagename'), '2018-02-01', parameters('accountSasProperties')).accountSasToken]"
```

## <a name="pickzones"></a>pickZones

`pickZones(providerNamespace, resourceType, location, [numberOfZones], [offset])`

Détermine si un type de ressource prend en charge les zones pour l’emplacement ou la région spécifié(e).  Cette fonction ne prend en charge que les ressources zonales, les services redondants dans une zone retournent un tableau vide.  Pour plus d’informations, consultez [Services Azure prenant en charge les Zones de disponibilité](../../availability-zones/az-region.md).  Pour utiliser la fonction pickZones avec des services redondants dans une zone, consultez les exemples ci-dessous.

### <a name="parameters"></a>Paramètres

| Paramètre | Obligatoire | Type | Description |
|:--- |:--- |:--- |:--- |
| espacedenoms_fournisseur | Oui | string | Espace de noms du fournisseur du type de ressource pour lequel la prise en charge des zones doit être vérifiée. |
| resourceType | Oui | string | Type de ressource pour lequel la prise en charge des zones doit être vérifiée. |
| location | Oui | string | Région pour laquelle la prise en charge des zones doit être vérifiée. |
| numberOfZones | Non | entier | Nombre de zones logiques à retourner. La valeur par défaut est 1. Le nombre doit être un entier positif compris entre 1 et 3.  Utilisez 1 pour les ressources à une seule zone. Pour les ressources multizones, la valeur doit être inférieure ou égale au nombre de zones prises en charge. |
| offset | Non | entier | Décalage par rapport à la zone logique de départ. La fonction retourne une erreur si le décalage plus le nombre de zones (numberOfZones) dépasse le nombre de zones prises en charge. |

### <a name="return-value"></a>Valeur retournée

Tableau avec les zones prises en charge. Quand les valeurs par défaut pour offset et numberOfZones sont utilisées, un type de ressource et une région qui prend en charge les zones retournent le tableau suivant :

```json
[
    "1"
]
```

Quand le paramètre `numberOfZones` est défini sur 3, il retourne :

```json
[
    "1",
    "2",
    "3"
]
```

Quand le type de ressource ou la région ne prend pas en charge les zones, un tableau vide est retourné.  Un tableau vide est également retourné pour les services redondants dans une zone.

```json
[
]
```

### <a name="remarks"></a>Remarques

Il existe différentes catégories pour les Zones de disponibilité Azure : zonal et redondant interzone.  La fonction pickZones peut être utilisée pour retourner un numéro de zone de disponibilité ou des nombres pour une ressource zonale.  Pour les services redondants interzone (ZRS), la fonction retourne un tableau vide.  Les ressources zonales peuvent généralement être identifiées par l’utilisation d’une propriété `zones` sur l’en-tête de ressource.  Les services redondants dans une zone présentent différentes façons d’identifier et d’utiliser les zones de disponibilité par ressource. Utilisez la documentation d’un service spécifique pour déterminer la catégorie de prise en charge des zones de disponibilité.  Pour plus d’informations, consultez [Services Azure prenant en charge les Zones de disponibilité](../../availability-zones/az-region.md).

Pour déterminer si une région ou un emplacement Azure donné prend en charge les zones de disponibilité, appelez la fonction pickZones () avec un type de ressource zonale, par exemple `Microsoft.Storage/storageAccounts`.  Si la réponse n’est pas vide, la région prend en charge les zones de disponibilité.

### <a name="pickzones-example"></a>Exemple de pickZones

Le modèle suivant présente trois résultats pour l’utilisation de la fonction pickZones.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/functions/resource/pickzones.json":::

La sortie des exemples précédents retourne trois tableaux.

| Nom | Type | Valeur |
| ---- | ---- | ----- |
| supported | tableau | [ "1" ] |
| notSupportedRegion | tableau | [] |
| notSupportedType | tableau | [] |

Vous pouvez vous servir de la réponse de pickZones pour déterminer s’il convient de fournir une valeur null pour des zones ou affecter des machines virtuelles à différentes zones. L’exemple suivant définit une valeur pour la zone en fonction de la disponibilité des zones.

```json
"zones": {
  "value": "[if(not(empty(pickZones('Microsoft.Compute', 'virtualMachines', 'westus2'))), string(add(mod(copyIndex(),3),1)), json('null'))]"
},
```

L’exemple suivant montre comment utiliser la fonction pickZones pour activer la redondance de zone pour Cosmos DB.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/functions/resource/pickzones-cosmosdb.json":::

## <a name="providers"></a>fournisseurs

**La fonction de fournisseurs est déconseillée.** Nous ne recommandons plus son utilisation. Si vous avez utilisé cette fonction pour obtenir une version d’API pour le fournisseur de ressources, nous vous recommandons de fournir une version d’API spécifique dans votre modèle. L’utilisation d’une version d’API retournée dynamiquement peut rompre votre modèle si les propriétés changent d’une version à l’autre.

## <a name="reference"></a>reference

`reference(resourceName or resourceIdentifier, [apiVersion], ['Full'])`

Renvoie un objet représentant l’état d’exécution d’une ressource.

### <a name="parameters"></a>Paramètres

| Paramètre | Obligatoire | Type | Description |
|:--- |:--- |:--- |:--- |
| nom_ressource ou identificateur_ressource |Oui |string |Nom ou identificateur unique d’une ressource. Lorsque vous référencez une ressource dans le modèle actuel, indiquez uniquement le nom de la ressource en tant que paramètre. Lorsque vous référencez une ressource précédemment déployée, ou lorsque le nom de la ressource est ambigu, fournissez l’ID de la ressource. |
| apiVersion |Non |string |Version d’API de la ressource spécifiée. **Ce paramètre est obligatoire lorsque la ressource n’est pas approvisionnée dans le même modèle.** En règle générale, au format, **aaaa-mm-jj**. Pour obtenir les versions d’API valides pour votre ressource, consultez [Référence de modèle](/azure/templates/). |
| 'Full' |Non |string |Valeur qui spécifie si l’objet de ressource complet doit être retourné. Si vous ne spécifiez pas `'Full'`, seul l’objet properties de la ressource est retourné. L’objet complet comprend des valeurs telles que l’ID de ressource et l’emplacement. |

### <a name="return-value"></a>Valeur retournée

Chaque type de ressource retourne des propriétés différentes pour la fonction de référence. La fonction ne retourne pas un format prédéfini unique. En outre, la valeur retournée diffère selon la valeur de l’argument `'Full'`. Pour afficher les propriétés d’un type de ressource, renvoyez l’objet dans la section des sorties comme indiqué dans l’exemple.

### <a name="remarks"></a>Notes

La fonction de référence récupère l’état d’exécution d’une ressource déployée précédemment ou déployée dans le modèle actuel. Cet article montre des exemples pour les deux scénarios.

En règle générale, vous utilisez la fonction de `reference` pour renvoyer une valeur particulière d’un objet, telle que l’URI du point de terminaison d’objet blob ou le nom de domaine complet.

```json
"outputs": {
  "BlobUri": {
    "type": "string",
    "value": "[reference(resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))).primaryEndpoints.blob]"
  },
  "FQDN": {
    "type": "string",
    "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', parameters('ipAddressName'))).dnsSettings.fqdn]"
  }
}
```

Utilisez `'Full'` quand vous avez besoin de valeurs de ressource qui ne font pas partie du schéma de propriétés. Par exemple, pour définir des stratégies d’accès à des coffres de clés, obtenez les propriétés d’identité d’une machine virtuelle.

```json
{
  "type": "Microsoft.KeyVault/vaults",
  "apiVersion": "2019-09-01",
  "name": "vaultName",
  "properties": {
    "tenantId": "[subscription().tenantId]",
    "accessPolicies": [
      {
        "tenantId": "[reference(resourceId('Microsoft.Compute/virtualMachines', variables('vmName')), '2019-03-01', 'Full').identity.tenantId]",
        "objectId": "[reference(resourceId('Microsoft.Compute/virtualMachines', variables('vmName')), '2019-03-01', 'Full').identity.principalId]",
        "permissions": {
          "keys": [
            "all"
          ],
          "secrets": [
            "all"
          ]
        }
      }
    ],
    ...
```

### <a name="valid-uses"></a>Utilisations valides

La fonction de référence ne peut être utilisée que dans les propriétés d’une définition de ressource et dans la section de sortie d’un modèle ou d’un déploiement. Lorsqu’elle est utilisée avec une [itération de propriété](copy-properties.md), vous pouvez utiliser la fonction de référence pour `input`, car l’expression est affectée à la propriété de ressource.

Vous ne pouvez pas utiliser la fonction de référence pour définir la valeur de la propriété `count` dans une boucle de copie. Vous pouvez l'utiliser pour définir d’autres propriétés de la boucle. La référence est bloquée pour la propriété de nombre car cette propriété doit être déterminée préalablement à la résolution de la fonction de référence.

Pour utiliser la fonction `reference` ou n'importe quelle fonction `list*` dans la section outputs d'un modèle imbriqué, vous devez définir les `expressionEvaluationOptions` de manière à utiliser l'évaluation [avec portée interne](linked-templates.md#expression-evaluation-scope-in-nested-templates) ou utiliser un modèle lié plutôt qu'un modèle imbriqué.

Si vous utilisez une fonction `reference` dans une ressource qui est déployée conditionnellement, la fonction est évaluée même si la ressource n’est pas déployée.  Vous obtenez une erreur si la fonction `reference` fait référence à une ressource qui n’existe pas. Utilisez la fonction `if` pour vous assurer que la fonction est évaluée lors du déploiement de la ressource. Consultez la [fonction if](template-functions-logical.md#if) pour un exemple de modèle qui utilise « if » et « reference » avec une ressource déployée de manière conditionnelle.

### <a name="implicit-dependency"></a>Dépendance implicite

En utilisant la fonction reference, vous déclarez de manière implicite qu’une ressource dépend d’une autre ressource si la ressource référencée est configurée dans le même modèle et vous désignez cette ressource par son nom (pas par son ID). Vous n’avez pas besoin d’utiliser également la propriété dependsOn. La fonction n’est pas évaluée tant que le déploiement de la ressource référencée n’est pas terminé.

### <a name="resource-name-or-identifier"></a>Nom ou identificateur de la ressource

Lorsque vous faites référence à une ressource déployée dans le même modèle, indiquez le nom de la ressource.

```json
"value": "[reference(parameters('storageAccountName'))]"
```

Lorsque vous faites référence à une ressource qui n’est pas déployée dans le même modèle, fournissez l’ID de ressource et `apiVersion`.

```json
"value": "[reference(resourceId(parameters('storageResourceGroup'), 'Microsoft.Storage/storageAccounts', parameters('storageAccountName')), '2018-07-01')]"
```

Pour éviter toute ambiguïté quant à la ressource à laquelle vous faites référence, vous pouvez fournir un identificateur de ressource complet.

```json
"value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', parameters('ipAddressName')))]"
```

Quand vous créez une référence complète à une ressource, l’ordre utilisé pour combiner les segments de type et de nom n’est pas une simple concaténation des deux. Au lieu de cela, utilisez après l’espace de noms une séquence de paires *type/nom* du moins spécifique au plus spécifique :

`{resource-provider-namespace}/{parent-resource-type}/{parent-resource-name}[/{child-resource-type}/{child-resource-name}]`

Par exemple :

`Microsoft.Compute/virtualMachines/myVM/extensions/myExt` est correct `Microsoft.Compute/virtualMachines/extensions/myVM/myExt` n’est pas correct

Pour simplifier la création d’un ID de ressource, utilisez les fonctions `resourceId()` décrites dans ce document au lieu de la fonction `concat()`.

### <a name="get-managed-identity"></a>Obtenir une identité managée

[Les identités managées pour ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md) sont des [types de ressources d’extension](../management/extension-resource-types.md) créés implicitement pour certaines ressources. L’identité managée n’étant pas définie explicitement dans le modèle, vous devez référencer la ressource à laquelle elle est appliquée. Utilisez `Full` pour obtenir toutes les propriétés, y compris l’identité créée implicitement.

Le modèle est :

`"[reference(resourceId(<resource-provider-namespace>, <resource-name>), <API-version>, 'Full').Identity.propertyName]"`

Par exemple, pour obtenir l'ID de principal d'une identité managée appliquée à une machine virtuelle, utilisez :

```json
"[reference(resourceId('Microsoft.Compute/virtualMachines', variables('vmName')),'2019-12-01', 'Full').identity.principalId]",
```

Ou, pour obtenir l'ID de locataire d'une identité managée appliquée à un groupe de machines virtuelles identiques, utilisez :

```json
"[reference(resourceId('Microsoft.Compute/virtualMachineScaleSets',  variables('vmNodeType0Name')), 2019-12-01, 'Full').Identity.tenantId]"
```

### <a name="reference-example"></a>Exemple de référence

L’exemple de modèle suivant déploie une ressource, puis référence cette ressource.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/functions/resource/referencewithstorage.json":::

L’exemple précédent retourne les deux objets. L’objet de propriétés retourné présente le format suivant :

```json
{
   "creationTime": "2017-10-09T18:55:40.5863736Z",
   "primaryEndpoints": {
     "blob": "https://examplestorage.blob.core.windows.net/",
     "file": "https://examplestorage.file.core.windows.net/",
     "queue": "https://examplestorage.queue.core.windows.net/",
     "table": "https://examplestorage.table.core.windows.net/"
   },
   "primaryLocation": "southcentralus",
   "provisioningState": "Succeeded",
   "statusOfPrimary": "available",
   "supportsHttpsTrafficOnly": false
}
```

L’objet complet retourné présente le format suivant :

```json
{
  "apiVersion":"2021-04-01",
  "location":"southcentralus",
  "sku": {
    "name":"Standard_LRS",
    "tier":"Standard"
  },
  "tags":{},
  "kind":"Storage",
  "properties": {
    "creationTime":"2017-10-09T18:55:40.5863736Z",
    "primaryEndpoints": {
      "blob":"https://examplestorage.blob.core.windows.net/",
      "file":"https://examplestorage.file.core.windows.net/",
      "queue":"https://examplestorage.queue.core.windows.net/",
      "table":"https://examplestorage.table.core.windows.net/"
    },
    "primaryLocation":"southcentralus",
    "provisioningState":"Succeeded",
    "statusOfPrimary":"available",
    "supportsHttpsTrafficOnly":false
  },
  "subscriptionId":"<subscription-id>",
  "resourceGroupName":"functionexamplegroup",
  "resourceId":"Microsoft.Storage/storageAccounts/examplestorage",
  "referenceApiVersion":"2021-04-01",
  "condition":true,
  "isConditionTrue":true,
  "isTemplateResource":false,
  "isAction":false,
  "provisioningOperation":"Read"
}
```

L’exemple de modèle suivant référence un compte de stockage qui n’est pas déployé dans ce modèle. Le compte de stockage existe déjà dans le même abonnement.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/functions/resource/reference.json":::

## <a name="resourcegroup"></a>resourceGroup

`resourceGroup()`

Renvoie un objet qui représente le groupe de ressources actuel.

### <a name="return-value"></a>Valeur retournée

L’objet renvoyé présente le format suivant :

```json
{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}",
  "name": "{resourceGroupName}",
  "type":"Microsoft.Resources/resourceGroups",
  "location": "{resourceGroupLocation}",
  "managedBy": "{identifier-of-managing-resource}",
  "tags": {
  },
  "properties": {
    "provisioningState": "{status}"
  }
}
```

La propriété **ManagedBy** est retournée uniquement pour les groupes de ressources qui contiennent des ressources gérées par un autre service. Pour Managed Applications, Databricks et AKS, la valeur de la propriété est l’ID de ressource de la ressource de gestion.

### <a name="remarks"></a>Notes

La fonction `resourceGroup()` ne peut pas être utilisée dans un modèle qui est [déployé au niveau abonnement](deploy-to-subscription.md). Elle n’est utilisable que dans les modèles déployés sur un groupe de ressources. Vous pouvez utiliser la fonction `resourceGroup()` dans un [modèle lié ou imbriqué (avec portée interne)](linked-templates.md) qui cible un groupe de ressources, même lorsque le modèle parent est déployé dans l’abonnement. Dans ce scénario, le modèle lié ou imbriqué est déployé au niveau du groupe de ressources. Pour plus d’informations sur le ciblage d’un groupe de ressources dans un déploiement au niveau de l’abonnement, consultez [Déployer des ressources Azure sur plusieurs groupes de ressources et des abonnements](./deploy-to-resource-group.md).

Une utilisation courante de la fonction resourceGroup consiste à créer des ressources dans le même emplacement que le groupe de ressources. L’exemple suivant utilise l’emplacement du groupe de ressources pour une valeur de paramètre par défaut.

```json
"parameters": {
  "location": {
    "type": "string",
    "defaultValue": "[resourceGroup().location]"
  }
}
```

Vous pouvez également utiliser la fonction `resourceGroup` pour appliquer des balises du groupe de ressources à une ressource. Pour plus d’informations, voir [Appliquer les balises d’un groupe de ressources](../management/tag-resources.md#apply-tags-from-resource-group).

Quand vous utilisez des modèles imbriqués pour effectuer un déploiement sur plusieurs groupes de ressources, vous pouvez spécifier l’étendue de l’évaluation de la fonction `resourceGroup`. Pour plus d’informations, voir [Déployer des ressources Azure sur plusieurs groupes de ressources et des abonnements](./deploy-to-resource-group.md).

### <a name="resource-group-example"></a>Exemple de groupe de ressources

L’exemple suivant retourne les propriétés du groupe de ressources.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/functions/resource/resourcegroup.json":::

L’exemple précédent renvoie un objet dans le format suivant :

```json
{
  "id": "/subscriptions/{subscription-id}/resourceGroups/examplegroup",
  "name": "examplegroup",
  "type":"Microsoft.Resources/resourceGroups",
  "location": "southcentralus",
  "properties": {
    "provisioningState": "Succeeded"
  }
}
```

## <a name="resourceid"></a>resourceId

`resourceId([subscriptionId], [resourceGroupName], resourceType, resourceName1, [resourceName2], ...)`

Retourne l'identificateur unique d'une ressource. Vous utilisez cette fonction lorsque le nom de la ressource est ambigu ou non configuré dans le même modèle. Le format de l’identificateur retourné varie selon que le déploiement se produit à l’échelle d’un groupe de ressources, d’un abonnement, d’un groupe d’administration ou d’un locataire.

### <a name="parameters"></a>Paramètres

| Paramètre | Obligatoire | Type | Description |
|:--- |:--- |:--- |:--- |
| subscriptionId |Non |string (au format GUID) |La valeur par défaut est l’abonnement actuel. Spécifiez cette valeur lorsque vous devez récupérer une ressource se trouvant dans un autre abonnement. Fournissez cette valeur uniquement lors du déploiement à l’échelle d’un groupe de ressources ou d’un abonnement. |
| resourceGroupName |Non |string |La valeur par défaut est le groupe de ressources actuel. Spécifiez cette valeur lorsque vous devez récupérer une ressource se trouvant dans un autre groupe de ressources. Fournissez cette valeur uniquement lors du déploiement à l’échelle d’un groupe de ressources. |
| resourceType |Oui |string |Type de ressource, y compris l'espace de noms du fournisseur de ressources. |
| nom_ressource1 |Oui |string |Nom de la ressource. |
| nom_ressource2 |Non |string |Segment de nom de ressource suivant si nécessaire. |

Continuez à ajouter des noms de ressource en paramètres lorsque le type de ressource contient plus de segments.

### <a name="return-value"></a>Valeur retournée

Lorsque le modèle est déployé à l’échelle d’un groupe de ressources, l’ID de ressource est retourné au format suivant :

```json
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
```

Vous pouvez utiliser la fonction resourceId pour d’autres étendues de déploiement, mais le format de l’ID change.

Si vous utilisez la fonction resourceId lors du déploiement sur un abonnement, l’ID de ressource est retourné au format suivant :

```json
/subscriptions/{subscriptionId}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
```

Si vous utilisez la fonction resourceId lors du déploiement sur un client ou un groupe d’administration, l’ID de ressource est retourné au format suivant :

```json
/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
```

Pour éviter toute confusion, nous vous recommandons de ne pas utiliser la fonction `resourceId`lorsque vous travaillez avec des ressources déployées sur l’abonnement, le groupe d’administration ou le locataire. Utilisez plutôt la fonction ID conçue pour l’étendue.

Pour les [ressources au niveau de l’abonnement](deploy-to-subscription.md), utilisez la fonction [subscriptionResourceId](#subscriptionresourceid).

Pour les [ressources au niveau du groupe d’administration](deploy-to-management-group.md), utilisez la fonction [extensionResourceId](#extensionresourceid) pour référencer une ressource qui est implémentée en tant qu’extension d’un groupe d’administration. Par exemple, des définitions de stratégie personnalisée déployées sur un groupe d’administration sont des extensions de celui-ci. Utilisez la fonction [tenantResourceId](#tenantresourceid) pour référencer les ressources déployées sur le locataire, mais disponibles dans votre groupe d’administration. Par exemple, les définitions de stratégie intégrées sont implémentées en tant que ressources au niveau locataire.

Pour les [ressources au niveau locataire](deploy-to-tenant.md), utilisez la fonction [tenantResourceId](#tenantresourceid). Utilisez la fonction tenantResourceId pour les définitions de stratégie intégrées, car elles sont implémentées au niveau locataire.

### <a name="remarks"></a>Notes

Le nombre de paramètres que vous fournissez varie selon qu’il s'agit d’une ressource parent ou d’une ressource enfant et selon que la ressource fait partie du même abonnement ou du même groupe de ressources.

Pour obtenir l’ID de ressource d’une ressource parent se trouvant dans le même abonnement et le même groupe de ressources, indiquez le type et le nom de la ressource.

```json
"[resourceId('Microsoft.ServiceBus/namespaces', 'namespace1')]"
```

Pour obtenir l’ID de ressource d’une ressource enfant, faites attention au nombre de segments dans le type de ressource. Indiquez un nom de ressource pour chaque segment du type de ressource. Le nom du segment correspond à la ressource qui existe pour cette partie de la hiérarchie.

```json
"[resourceId('Microsoft.ServiceBus/namespaces/queues/authorizationRules', 'namespace1', 'queue1', 'auth1')]"
```

Pour obtenir l’ID de ressource d’une ressource se trouvant dans le même abonnement mais dans un groupe de ressources différent, indiquez le nom du groupe de ressources.

```json
"[resourceId('otherResourceGroup', 'Microsoft.Storage/storageAccounts', 'examplestorage')]"
```

Pour obtenir l’ID de ressource d’une ressource se trouvant dans un abonnement et un groupe de ressources différents, indiquez l’ID d’abonnement et le nom du groupe de ressources.

```json
"[resourceId('xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx', 'otherResourceGroup', 'Microsoft.Storage/storageAccounts','examplestorage')]"
```

Souvent, vous devez utiliser cette fonction lorsque vous utilisez un compte de stockage ou un réseau virtuel se trouvant dans un autre groupe de ressources. L'exemple suivant montre comment une ressource d'un groupe de ressources externe peut être facilement utilisée :

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/functions/resource/resourceid-external.json":::

### <a name="resource-id-example"></a>Exemple d’ID de ressource

L’exemple suivant retourne l’ID de ressource pour un compte de stockage dans le groupe de ressources :

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/functions/resource/resourceid.json":::

La sortie de l’exemple précédent avec les valeurs par défaut se présente comme suit :

| Nom | Type | Valeur |
| ---- | ---- | ----- |
| sameRGOutput | String | /subscriptions/{current-sub-id}/resourceGroups/examplegroup/providers/Microsoft.Storage/storageAccounts/examplestorage |
| differentRGOutput | String | /subscriptions/{current-sub-id}/resourceGroups/otherResourceGroup/providers/Microsoft.Storage/storageAccounts/examplestorage |
| differentSubOutput | String | /subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/otherResourceGroup/providers/Microsoft.Storage/storageAccounts/examplestorage |
| nestedResourceOutput | String | /subscriptions/{current-sub-id}/resourceGroups/examplegroup/providers/Microsoft.SQL/servers/serverName/databases/databaseName |

## <a name="subscription"></a>subscription

`subscription()`

Retourne des détails concernant l’abonnement pour le déploiement actuel.

### <a name="return-value"></a>Valeur retournée

La fonction retourne les informations au format suivant :

```json
{
  "id": "/subscriptions/{subscription-id}",
  "subscriptionId": "{subscription-id}",
  "tenantId": "{tenant-id}",
  "displayName": "{name-of-subscription}"
}
```

### <a name="remarks"></a>Notes

Quand vous utilisez des modèles imbriqués pour effectuer un déploiement sur plusieurs abonnements, vous pouvez spécifier l’étendue de l’évaluation de la fonction subscription. Pour plus d’informations, voir [Déployer des ressources Azure sur plusieurs groupes de ressources et des abonnements](./deploy-to-resource-group.md).

### <a name="subscription-example"></a>Exemple d’abonnement

L’exemple suivant montre la fonction subscription appelée dans la section outputs.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/functions/resource/subscription.json":::

## <a name="subscriptionresourceid"></a>subscriptionResourceId

`subscriptionResourceId([subscriptionId], resourceType, resourceName1, [resourceName2], ...)`

Retourne l’identificateur unique d’une ressource déployée au niveau de l’abonnement.

### <a name="parameters"></a>Paramètres

| Paramètre | Obligatoire | Type | Description |
|:--- |:--- |:--- |:--- |
| subscriptionId |Non |string (au format GUID) |La valeur par défaut est l’abonnement actuel. Spécifiez cette valeur lorsque vous devez récupérer une ressource se trouvant dans un autre abonnement. |
| resourceType |Oui |string |Type de ressource, y compris l'espace de noms du fournisseur de ressources. |
| nom_ressource1 |Oui |string |Nom de la ressource. |
| nom_ressource2 |Non |string |Segment de nom de ressource suivant si nécessaire. |

Continuez à ajouter des noms de ressource en paramètres lorsque le type de ressource contient plus de segments.

### <a name="return-value"></a>Valeur retournée

L'identificateur est retourné au format suivant :

```json
/subscriptions/{subscriptionId}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
```

### <a name="remarks"></a>Notes

Utilisez cette fonction pour récupérer l’ID des ressources [déployées dans l’abonnement](deploy-to-subscription.md) plutôt qu’un groupe de ressources. L’ID retourné diffère de la valeur retournée par la fonction [resourceId](#resourceid) en ce qu’il n’inclut pas de valeur de groupe de ressources.

### <a name="subscriptionresourceid-example"></a>Exemple subscriptionResourceID

Le modèle suivant attribue un rôle intégré. Vous pouvez le déployer soit sur un groupe de ressources, soit sur un abonnement. Il utilise la fonction `subscriptionResourceId` pour récupérer l’ID de ressource pour les rôles intégrés.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/functions/resource/subscriptionresourceid.json":::

## <a name="tenantresourceid"></a>tenantResourceId

`tenantResourceId(resourceType, resourceName1, [resourceName2], ...)`

Retourne l’identificateur unique d’une ressource déployée au niveau du tenant.

### <a name="parameters"></a>Paramètres

| Paramètre | Obligatoire | Type | Description |
|:--- |:--- |:--- |:--- |
| resourceType |Oui |string |Type de ressource, y compris l'espace de noms du fournisseur de ressources. |
| nom_ressource1 |Oui |string |Nom de la ressource. |
| nom_ressource2 |Non |string |Segment de nom de ressource suivant si nécessaire. |

Continuez à ajouter des noms de ressource en paramètres lorsque le type de ressource contient plus de segments.

### <a name="return-value"></a>Valeur retournée

L'identificateur est retourné au format suivant :

```json
/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
```

### <a name="remarks"></a>Notes

Cette fonction permet de récupérer l’ID d’une ressource déployée sur le tenant. L’ID retourné diffère des valeurs retournées par d’autres fonctions d’ID de ressource en ce qu’il n’inclut pas de valeurs de groupe de ressources ou d’abonnement.

### <a name="tenantresourceid-example"></a>Exemple tenantResourceId

Les définitions de stratégie intégrées sont des ressources de niveau locataire. Pour déployer une attribution de stratégie qui fait référence à une définition de stratégie intégrée, utilisez la fonction `tenantResourceId`.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/functions/resource/tenantresourceid.json":::

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir une description des sections d’un modèle ARM, consultez [Comprendre la structure et la syntaxe des modèles ARM](./syntax.md).
* Pour fusionner plusieurs modèles, consultez [Utilisation de modèles liés et imbriqués lors du déploiement de ressources Azure](linked-templates.md).
* Pour itérer un nombre spécifié lors de la création d’un type de ressource, consultez [Itération de ressource dans les modèles ARM](copy-resources.md).
* Pour découvrir comment déployer le modèle que vous avez créé, consultez [Déployer des ressources avec des modèles ARM et Azure PowerShell](deploy-powershell.md).
