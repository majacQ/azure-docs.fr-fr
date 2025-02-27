---
title: Utiliser le Stockage Blob comme magasin de points de contrôle sur Azure Stack Hub
description: Cet article explique comment utiliser le Stockage Blob comme magasin de points de contrôle dans Event Hubs sur Azure Stack Hub.
ms.topic: how-to
ms.date: 09/28/2021
ms.openlocfilehash: e3d75d4cd85bbf4e1bc8fc6fd581d510c35b0cb4
ms.sourcegitcommit: e8c34354266d00e85364cf07e1e39600f7eb71cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/29/2021
ms.locfileid: "129213512"
---
# <a name="use-blob-storage-as-checkpoint-store---event-hubs-on-azure-stack-hub"></a>Utilisation du Stockage Blob comme magasin de points de contrôle – Event Hubs sur Azure Stack Hub
Si vous utilisez Stockage Blob Azure comme magasin de points de contrôle dans un environnement qui prend en charge une autre version du Kit de développement logiciel (SDK) Stockage Blob que ceux généralement disponibles sur Azure, vous devez utiliser du code pour remplacer la version de l’API du service de stockage par la version prise en charge par cet environnement. Par exemple, si vous exécutez [Event Hubs sur Azure Stack Hub version 2002](/azure-stack/user/event-hubs-overview), la version la plus élevée disponible pour le service Stockage est la version 2017-11-09. Dans ce cas, vous devez utiliser du code pour cibler la version de l’API du service Stockage sur 2017-11-09. Pour obtenir un exemple sur la façon de cibler une version spécifique de l’API Stockage, consultez les exemples suivants sur GitHub : 

- [.NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs.Processor/samples/)
- [Java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob/EventProcessorWithCustomStorageVersion.java). 
- [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/main/sdk/eventhub/eventhubs-checkpointstore-blob/samples/v1/javascript/receiveEventsWithApiSpecificStorage.js) ou [TypeScript](https://github.com/Azure/azure-sdk-for-js/blob/main/sdk/eventhub/eventhubs-checkpointstore-blob/samples/v1/typescript/src/receiveEventsWithApiSpecificStorage.ts) 
- Python : [synchrone](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub-checkpointstoreblob/samples/receive_events_using_checkpoint_store_storage_api_version.py), [asynchrone](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub-checkpointstoreblob-aio/samples/receive_events_using_checkpoint_store_storage_api_version_async.py)

Si vous exécutez le récepteur Event Hubs qui utilise Stockage Blob comme magasin de points de contrôle sans cibler la version prise en charge par Azure Stack Hub, vous recevrez le message d’erreur suivant :

```
The value for one of the HTTP headers is not in the correct format
```


## <a name="sample-error-message-in-python"></a>Exemple de message d’erreur en Python
Pour Python, une erreur `azure.core.exceptions.HttpResponseError` est transmise au gestionnaire d’erreurs `on_error(partition_context, error)` de `EventHubConsumerClient.receive()`. Toutefois, la méthode `receive()` ne lève pas d’exception. `print(error)` imprimera les informations d’exception suivantes :

```bash
The value for one of the HTTP headers is not in the correct format.

RequestId:f048aee8-a90c-08ba-4ce1-e69dba759297
Time:2020-03-17T22:04:13.3559296Z
ErrorCode:InvalidHeaderValue
Error:None
HeaderName:x-ms-version
HeaderValue:2019-07-07
```

L’enregistreur d’événements consignera deux avertissements comme ceux-ci :

```bash
WARNING:azure.eventhub.extensions.checkpointstoreblobaio._blobstoragecsaio: 
An exception occurred during list_ownership for namespace '<namespace-name>.eventhub.<region>.azurestack.corp.microsoft.com' eventhub 'python-eh-test' consumer group '$Default'. 

Exception is HttpResponseError('The value for one of the HTTP headers is not in the correct format.\nRequestId:f048aee8-a90c-08ba-4ce1-e69dba759297\nTime:2020-03-17T22:04:13.3559296Z\nErrorCode:InvalidHeaderValue\nError:None\nHeaderName:x-ms-version\nHeaderValue:2019-07-07')

WARNING:azure.eventhub.aio._eventprocessor.event_processor:EventProcessor instance '26d84102-45b2-48a9-b7f4-da8916f68214' of eventhub 'python-eh-test' consumer group '$Default'. An error occurred while load-balancing and claiming ownership. 

The exception is HttpResponseError('The value for one of the HTTP headers is not in the correct format.\nRequestId:f048aee8-a90c-08ba-4ce1-e69dba759297\nTime:2020-03-17T22:04:13.3559296Z\nErrorCode:InvalidHeaderValue\nError:None\nHeaderName:x-ms-version\nHeaderValue:2019-07-07'). Retrying after 71.45254944090853 seconds
```



## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour en savoir plus sur le partitionnement et les points de contrôle : [Équilibrer la charge de partition sur plusieurs instances de votre application](event-processor-balance-partition-load.md)
