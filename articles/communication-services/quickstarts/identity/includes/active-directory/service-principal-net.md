---
ms.openlocfilehash: f34320c0ba70da94315858ed09ffd1ca3e90d846
ms.sourcegitcommit: d2738669a74cda866fd8647cb9c0735602642939
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/13/2021
ms.locfileid: "113656970"
---
> [!NOTE]
> Vous trouverez le code finalisé pour ce guide de démarrage rapide sur [GitHub](https://github.com/Azure-Samples/communication-services-dotnet-quickstarts/tree/main/use-managed-Identity).

## <a name="setting-up"></a>Configuration

### <a name="create-a-new-c-application"></a>Créer une application C#

Dans une fenêtre de console (par exemple cmd, PowerShell ou Bash), utilisez la commande `dotnet new` pour créer une application console avec le nom `ActiveDirectoryQuickstart`. Cette commande crée un projet C# « Hello World » simple avec un seul fichier source : `Program.cs`.

```console
dotnet new console -o ActiveDirectoryAuthenticationQuickstart
```

Remplacez votre répertoire par le dossier d’application que vous venez de créer, puis utilisez la commande `dotnet build` pour compiler votre application.

```console
cd ActiveDirectoryAuthenticationQuickstart
dotnet build
```

### <a name="install-the-sdk-packages"></a>Installer les packages du kit SDK

```console
dotnet add package Azure.Communication.Identity
dotnet add package Azure.Communication.Sms
dotnet add package Azure.Identity
```

### <a name="use-the-sdk-packages"></a>Utiliser les packages du kit SDK

Ajoutez les directives `using` suivantes à `Program.cs` pour utiliser les Kits de développement logiciel (SDK) Azure Identity et Stockage Azure.

```csharp
using Azure.Identity;
using Azure.Communication.Identity;
using Azure.Communication.Sms;
using Azure.Core;
using Azure;
```

## <a name="create-a-defaultazurecredential"></a>Créer un DefaultAzureCredential

Nous utiliserons le [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential) pour ce démarrage rapide. Ces informations d’identification sont appropriées pour les environnements de production et de développement. Comme elles sont nécessaires pour chaque opération, créons-les dans la classe `Program.cs`. Ajoutez ce qui suit au début du fichier.

```csharp
private DefaultAzureCredential credential = new DefaultAzureCredential();
```

## <a name="issue-a-token-with-service-principals"></a>Émettre un jeton avec des principaux de service

Nous allons maintenant ajouter du code qui utilise les informations d’identification créées pour émettre un jeton d’accès VoIP. Nous appellerons ce code plus tard :

```csharp
public Response<AccessToken> CreateIdentityAndGetTokenAsync(Uri resourceEndpoint)
{
    var client = new CommunicationIdentityClient(resourceEndpoint, this.credential);
    var identityResponse = client.CreateUser();
    var identity = identityResponse.Value;

    var tokenResponse = client.GetToken(identity, scopes: new[] { CommunicationTokenScope.VoIP });

    return tokenResponse;
}
```

## <a name="send-an-sms-with-service-principals"></a>Envoyer un SMS avec des principaux de service

Comme autre exemple d’utilisation des principaux de service, nous allons ajouter ce code qui utilise les mêmes informations d’identification pour envoyer un SMS :

```csharp
public SmsSendResult SendSms(Uri resourceEndpoint, string from, string to, string message)
{
    SmsClient smsClient = new SmsClient(resourceEndpoint, this.credential);
    SmsSendResult sendResult = smsClient.Send(
            from: from,
            to: to,
            message: message,
            new SmsSendOptions(enableDeliveryReport: true) // optional
        );

    return sendResult;
}
```

## <a name="write-the-main-method"></a>Écrire la méthode main

Votre `Program.cs` doit déjà disposer d’une méthode main. Ajoutons du code qui appellera notre code précédemment créé pour démontrer l’utilisation des principaux de service :

```csharp
static void Main(string[] args)
{
    // You can find your endpoint and access key from your resource in the Azure portal
    // e.g. "https://<RESOURCE_NAME>.communication.azure.com";
    Uri endpoint = new("https://<RESOURCENAME>.communication.azure.com/");

    // We need an instance of the program class to use within this method.
    Program instance = new();

    Console.WriteLine("Retrieving new Access Token, using Service Principals");
    Response<AccessToken> response = instance.CreateIdentityAndGetTokenAsync(endpoint);
    Console.WriteLine($"Retrieved Access Token: {response.Value.Token}");

    Console.WriteLine("Sending SMS using Service Principals");

    // You will need a phone number from your resource to send an SMS.
    SmsSendResult result = instance.SendSms(endpoint, "<Your ACS Phone Number>", "<The Phone Number you'd like to send the SMS to.>", "Hello from using Service Principals");
    Console.WriteLine($"Sms id: {result.MessageId}");
    Console.WriteLine($"Send Result Successful: {result.Successful}");
}
```

Le fichier `Program.cs` final doit ressembler à ce qui suit :

```csharp
class Program
     {
          private DefaultAzureCredential credential = new DefaultAzureCredential();
          static void Main(string[] args)
          {
               // You can find your endpoint and access key from your resource in the Azure portal
               // e.g. "https://<RESOURCE_NAME>.communication.azure.com";
               Uri endpoint = new("https://acstestingrifox.communication.azure.com/");

               // We need an instance of the program class to use within this method.
               Program instance = new();

               Console.WriteLine("Retrieving new Access Token, using Service Principals");
               Response<AccessToken> response = instance.CreateIdentityAndGetTokenAsync(endpoint);
               Console.WriteLine($"Retrieved Access Token: {response.Value.Token}");

               Console.WriteLine("Sending SMS using Service Principals");

               // You will need a phone number from your resource to send an SMS.
               SmsSendResult result = instance.SendSms(endpoint, "<Your ACS Phone Number>", "<The Phone Number you'd like to send the SMS to.>", "Hello from Service Principals");
               Console.WriteLine($"Sms id: {result.MessageId}");
               Console.WriteLine($"Send Result Successful: {result.Successful}");
          }
          public Response<AccessToken> CreateIdentityAndGetTokenAsync(Uri resourceEndpoint)
          {
               var client = new CommunicationIdentityClient(resourceEndpoint, this.credential);
               var identityResponse = client.CreateUser();
               var identity = identityResponse.Value;

               var tokenResponse = client.GetToken(identity, scopes: new[] { CommunicationTokenScope.VoIP });

               return tokenResponse;
          }
          public SmsSendResult SendSms(Uri resourceEndpoint, string from, string to, string message)
          {
               SmsClient smsClient = new SmsClient(resourceEndpoint, this.credential);
               SmsSendResult sendResult = smsClient.Send(
                    from: from,
                    to: to,
                    message: message,
                    new SmsSendOptions(enableDeliveryReport: true) // optional
               );

               return sendResult;
          }
    }
```

## <a name="run-the-program"></a>Exécuter le programme

Vous devez maintenant être en mesure d’exécuter votre application en utilisant `dotnet run` à partir de votre dossier d’application. La sortie doit ressembler à ce qui suit :
```
Retrieving new Access Token, using Service Principals
Retrieved Access Token: ey....
Sending SMS using Service Principals
Sms id: Outgoing_..._noam
Send Result Successful: True
```
