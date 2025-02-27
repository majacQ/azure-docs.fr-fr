---
author: probableprime
ms.service: azure-communication-services
ms.topic: include
ms.date: 09/08/2021
ms.author: rifox
ms.openlocfilehash: 0ba2baf8d6517a4719f63bf4cec37541e223540a
ms.sourcegitcommit: 0770a7d91278043a83ccc597af25934854605e8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/13/2021
ms.locfileid: "128700478"
---
[!INCLUDE [Install SDK](../install-sdk/install-sdk-ios.md)]

Avec votre SDK iOS, vous pouvez vous abonner à la plupart des propriétés et des collections pour être averti quand des valeurs changent.

## <a name="properties"></a>Propriétés
Pour vous abonner aux événements `property changed`, utilisez le code suivant.

```swift
call.delegate = self
// Get the property of the call state by getting on the call's state member
public func call(_ call: Call, didChangeState args: PropertyChangedEventArgs) {
{
    print("Callback from SDK when the call state changes, current state: " + call.state.rawValue)
}

// to unsubscribe
self.call.delegate = nil
```

## <a name="collections"></a>Regroupements
Pour vous abonner aux événements `collection updated`, utilisez le code suivant.

```swift
call.delegate = self
// Collection contains the streams that were added or removed only
public func call(_ call: Call, didUpdateLocalVideoStreams args: LocalVideoStreamsUpdatedEventArgs) {
{
    print(args.addedStreams.count)
    print(args.removedStreams.count)
}
// to unsubscribe
self.call.delegate = nil
```