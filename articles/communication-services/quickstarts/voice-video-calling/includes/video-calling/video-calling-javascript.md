---
author: probableprime
ms.service: azure-communication-services
ms.topic: include
ms.date: 03/10/2021
ms.author: rifox
ms.openlocfilehash: 98e4f5aa5cf817ea16fb7337e809c49bf418b12f
ms.sourcegitcommit: 2eac9bd319fb8b3a1080518c73ee337123286fa2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2021
ms.locfileid: "123251538"
---
Prenez en main Azure Communication Services en utilisant le SDK d’appel Communication Services pour ajouter des appels vidéo 1 à 1 à votre application. Vous allez découvrir comment démarrer un appel vidéo et y répondre à l’aide du SDK d’appel Azure Communication Services pour JavaScript.

## <a name="sample-code"></a>Exemple de code

Si vous souhaitez passer à la fin, vous pouvez télécharger ce guide de démarrage rapide en guise d’exemple sur [GitHub](https://github.com/Azure-Samples/communication-services-javascript-quickstarts/tree/main/add-1-on-1-video-calling).

## <a name="prerequisites"></a>Prérequis
- Obtenez un compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Node.js](https://nodejs.org/en/), Active LTS et Maintenance LTS (version 8.11.1 et 10.14.1).
- Créez une ressource Communication Services active. [Créez une ressource Communication Services](../../../create-communication-resource.md?pivots=platform-azp&tabs=windows).
- Créez un jeton d’accès utilisateur pour instancier le client d’appel. [Découvrez comment créer et gérer des jetons d’accès utilisateur](../../../access-tokens.md?pivots=programming-language-csharp).

## <a name="setting-up"></a>Configuration
### <a name="create-a-new-nodejs-application"></a>Création d’une application Node.js
Ouvrez votre fenêtre de terminal ou de commande, créez un répertoire pour votre application, puis accédez-y.
```console
mkdir calling-quickstart && cd calling-quickstart
```
### <a name="install-the-package"></a>Installer le package
Utilisez la commande `npm install` pour installer le SDK Azure Communication Services Calling pour JavaScript.
> [!IMPORTANT]
> Ce guide de démarrage rapide utilise la version `1.1.0-beta.1` du SDK Azure Communication Services Calling. 
```console
npm install @azure/communication-common --save
npm install @azure/communication-calling@1.1.0-beta.1 --save
```
### <a name="set-up-the-app-framework"></a>Configurer le framework d’application

Ce guide de démarrage rapide utilise webpack pour regrouper les ressources de l’application. Exécutez la commande suivante pour installer les packages npm `webpack`, `webpack-cli` et `webpack-dev-server`, puis listez-les comme dépendances de développement dans votre fichier `package.json` :

```console
npm install webpack@4.42.0 webpack-cli@3.3.11 webpack-dev-server@3.10.3 --save-dev
```
Créez un fichier `index.html` dans le répertoire racine de votre projet. Nous utiliserons ce fichier pour configurer une disposition de base qui permettra à l’utilisateur d’effectuer un appel vidéo 1 à 1.

Voici le code :
```html
<!-- index.html -->
<!DOCTYPE html>
<html>
    <head>
        <title>Azure Communication Services - Calling Web SDK</title>
    </head>
    <body>
        <h4>Azure Communication Services - Calling Web SDK</h4>
        <input id="user-access-token"
            type="text"
            placeholder="User access token"
            style="margin-bottom:1em; width: 500px;"/>
        <button id="initialize-call-agent" type="button">Initialize Call Agent</button>
        <br>
        <br>
        <input id="callee-acs-user-id"
            type="text"
            placeholder="Enter callee's ACS user identity in format: '8:acs:resourceId_userId'"
            style="margin-bottom:1em; width: 500px; display: block;"/>
        <button id="start-call-button" type="button" disabled="true">Start Call</button>
        <button id="hangup-call-button" type="button" disabled="true">Hang up Call</button>
        <button id="accept-call-button" type="button" disabled="true">Accept Call</button>
        <button id="start-video-button" type="button" disabled="true">Start Video</button>
        <button id="stop-video-button" type="button" disabled="true">Stop Video</button>
        <br>
        <br>
        <div id="connectedLabel" style="color: #13bb13;" hidden>Call is connected!</div>
        <br>
        <div id="remoteVideoContainer" style="width: 40%;" hidden>Remote participants' video streams:</div>
        <br>
        <div id="localVideoContainer" style="width: 30%;" hidden>Local video stream:</div>
        <!-- points to the bundle generated from client.js -->
        <script src="./bundle.js"></script>
    </body>
</html>
```

## <a name="acs-calling-web-sdk-object-model"></a>Modèle objet du SDK web Appel d’ ACS

Les classes et les interfaces suivantes gèrent certaines des principales fonctionnalités du SDK Azure Communication Services Calling :

| Nom                                | Description                                                                                                                              |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `CallClient`                        | Point d’entrée principal du SDK Appel.                                                                                                 |
| `AzureCommunicationTokenCredential` | Implémente l’interface `CommunicationTokenCredential`, qui est utilisée pour instancier `callAgent`.                                       |
| `CallAgent`                         | Sert à démarrer et à gérer les appels.                                                                                                          |
| `DeviceManager`                     | Utilisé pour gérer les appareils multimédias.                                                                                                            |
| `Call`                              | Utilisé pour représenter un appel                                                                                                              |
| `LocalVideoStream`                  | Utilisé pour créer un flux vidéo local pour un appareil photo sur le système local.                                                          |
| `RemoteParticipant`                 | Utilisé pour représenter un participant distant dans l’appel                                                                                   |
| `RemoteVideoStream`                 | Utilisé pour représenter un flux vidéo distant d’un participant distant.        

Créez un fichier dans le répertoire racine de votre projet sous le nom `client.js` qui contiendra la logique d’application pour ce guide de démarrage rapide. Ajoutez le code suivant à client.js :
```JavaScript
// Make sure to install the necessary dependencies
const { CallClient, VideoStreamRenderer, LocalVideoStream } = require('@azure/communication-calling');
const { AzureCommunicationTokenCredential } = require('@azure/communication-common');
const { AzureLogger, setLogLevel } = require("@azure/logger");
// Set the log level and output
setLogLevel('verbose');
AzureLogger.log = (...args) => {
    console.log(...args);
};

// Calling web sdk objects
let callAgent;
let deviceManager;
let call;
let incomingCall;
let localVideoStream;
let localVideoStreamRenderer;

// UI widgets
let userAccessToken = document.getElementById('user-access-token');
let calleeAcsUserId = document.getElementById('callee-acs-user-id');
let initializeCallAgentButton = document.getElementById('initialize-call-agent');
let startCallButton = document.getElementById('start-call-button');
let hangUpCallButton = document.getElementById('hangup-call-button');
let acceptCallButton = document.getElementById('accept-call-button');
let startVideoButton = document.getElementById('start-video-button');
let stopVideoButton = document.getElementById('stop-video-button');
let connectedLabel = document.getElementById('connectedLabel');
let remoteVideoContainer = document.getElementById('remoteVideoContainer');
let localVideoContainer = document.getElementById('localVideoContainer');

/**
 * Using the CallClient, initialize a CallAgent instance with a CommunicationUserCredential which will enable us to make outgoing calls and receive incoming calls. 
 * You can then use the CallClient.getDeviceManager() API instance to get the DeviceManager.
 */
initializeCallAgentButton.onclick = async () => {
    try {
        const callClient = new CallClient(); 
        tokenCredential = new AzureCommunicationTokenCredential(userAccessToken.value.trim());
        callAgent = await callClient.createCallAgent(tokenCredential)
        // Set up a camera device to use.
        deviceManager = await callClient.getDeviceManager();
        await deviceManager.askDevicePermission({ video: true });
        await deviceManager.askDevicePermission({ audio: true });
        // Listen for an incoming call to accept.
        callAgent.on('incomingCall', async (args) => {
            try {
                incomingCall = args.incomingCall;
                acceptCallButton.disabled = false;
                startCallButton.disabled = true;
            } catch (error) {
                console.error(error);
            }
        });

        startCallButton.disabled = false;
        initializeCallAgentButton.disabled = true;
    } catch(error) {
        console.error(error);
    }
}

/**
 * Place a 1:1 outgoing video call to a user
 * Add an event listener to initiate a call when the `startCallButton` is clicked:
 * First you have to enumerate local cameras using the deviceManager `getCameraList` API.
 * In this quickstart we're using the first camera in the collection. Once the desired camera is selected, a
 * LocalVideoStream instance will be constructed and passed within `videoOptions` as an item within the
 * localVideoStream array to the call method. Once your call connects it will automatically start sending a video stream to the other participant. 
 */
startCallButton.onclick = async () => {
    try {
        const localVideoStream = await createLocalVideoStream();
        const videoOptions = localVideoStream ? { localVideoStreams: [localVideoStream] } : undefined;
        call = callAgent.startCall([{ communicationUserId: calleeAcsUserId.value.trim() }], { videoOptions });
        // Subscribe to the call's properties and events.
        subscribeToCall(call);
    } catch (error) {
        console.error(error);
    }
}

/**
 * Accepting an incoming call with video
 * Add an event listener to accept a call when the `acceptCallButton` is clicked:
 * After subscrigin to the `CallAgent.on('incomingCall')` event, you can accept the incoming call.
 * You can pass the local video stream which you want to use to accept the call with.
 */
acceptCallButton.onclick = async () => {
    try {
        const localVideoStream = await createLocalVideoStream();
        const videoOptions = localVideoStream ? { localVideoStreams: [localVideoStream] } : undefined;
        call = await incomingCall.accept({ videoOptions });
        // Subscribe to the call's properties and events.
        subscribeToCall(call);
    } catch (error) {
        console.error(error);
    }
}

// Subscribe to a call obj.
// Listen for property changes and collection udpates.
subscribeToCall = (call) => {
    try {
        // Inspect the initial call.id value.
        console.log(`Call Id: ${call.id}`);
        //Subsribe to call's 'idChanged' event for value changes.
        call.on('idChanged', () => {
            console.log(`Call Id changed: ${call.id}`); 
        });

        // Inspect the initial call.state value.
        console.log(`Call state: ${call.state}`);
        // Subscribe to call's 'stateChanged' event for value changes.
        call.on('stateChanged', async () => {
            console.log(`Call state changed: ${call.state}`);
            if(call.state === 'Connected') {
                connectedLabel.hidden = false;
                acceptCallButton.disabled = true;
                startCallButton.disabled = true;
                hangUpCallButton.disabled = false;
                startVideoButton.disabled = false;
                stopVideoButton.disabled = false;
            } else if (call.state === 'Disconnected') {
                connectedLabel.hidden = true;
                startCallButton.disabled = false;
                hangUpCallButton.disabled = true;
                startVideoButton.disabled = true;
                stopVideoButton.disabled = true;
                console.log(`Call ended, call end reason={code=${call.callEndReason.code}, subCode=${call.callEndReason.subCode}}`);
            }   
        });

        call.localVideoStreams.forEach(async (lvs) => {
            localVideoStream = lvs;
            await displayLocalVideoStream();
        });
        call.on('localVideoStreamsUpdated', e => {
            e.added.forEach(async (lvs) => {
                localVideoStream = lvs;
                await displayLocalVideoStream();
            });
            e.removed.forEach(lvs => {
               removeLocalVideoStream();
            });
        });
        
        // Inspect the call's current remote participants and subscribe to them.
        call.remoteParticipants.forEach(remoteParticipant => {
            subscribeToRemoteParticipant(remoteParticipant);
        });
        // Subscribe to the call's 'remoteParticipantsUpdated' event to be
        // notified when new participants are added to the call or removed from the call.
        call.on('remoteParticipantsUpdated', e => {
            // Subscribe to new remote participants that are added to the call.
            e.added.forEach(remoteParticipant => {
                subscribeToRemoteParticipant(remoteParticipant)
            });
            // Unsubscribe from participants that are removed from the call
            e.removed.forEach(remoteParticipant => {
                console.log('Remote participant removed from the call.');
            });
        });
    } catch (error) {
        console.error(error);
    }
}

// Subscribe to a remote participant obj.
// Listen for property changes and collection udpates.
subscribeToRemoteParticipant = (remoteParticipant) => {
    try {
        // Inspect the initial remoteParticipant.state value.
        console.log(`Remote participant state: ${remoteParticipant.state}`);
        // Subscribe to remoteParticipant's 'stateChanged' event for value changes.
        remoteParticipant.on('stateChanged', () => {
            console.log(`Remote participant state changed: ${remoteParticipant.state}`);
        });

        // Inspect the remoteParticipants's current videoStreams and subscribe to them.
        remoteParticipant.videoStreams.forEach(remoteVideoStream => {
            subscribeToRemoteVideoStream(remoteVideoStream)
        });
        // Subscribe to the remoteParticipant's 'videoStreamsUpdated' event to be
        // notified when the remoteParticiapant adds new videoStreams and removes video streams.
        remoteParticipant.on('videoStreamsUpdated', e => {
            // Subscribe to new remote participant's video streams that were added.
            e.added.forEach(remoteVideoStream => {
                subscribeToRemoteVideoStream(remoteVideoStream)
            });
            // Unsubscribe from remote participant's video streams that were removed.
            e.removed.forEach(remoteVideoStream => {
                console.log('Remote participant video stream was removed.');
            })
        });
    } catch (error) {
        console.error(error);
    }
}

/**
 * Subscribe to a remote participant's remote video stream obj.
 * You have to subscribe to the 'isAvailableChanged' event to render the remoteVideoStream. If the 'isAvailable' property
 * changes to 'true', a remote participant is sending a stream. Whenever availability of a remote stream changes
 * you can choose to destroy the whole 'Renderer', a specific 'RendererView' or keep them, but this will result in displaying blank video frame.
 */
subscribeToRemoteVideoStream = async (remoteVideoStream) => {
    // Create a video stream renderer for the remote video stream.
    let videoStreamRenderer = new VideoStreamRenderer(remoteVideoStream);
    let view;
    const renderVideo = async () => {
        try {
            // Create a renderer view for the remote video stream.
            view = await videoStreamRenderer.createView();
            // Attach the renderer view to the UI.
            remoteVideoContainer.hidden = false;
            remoteVideoContainer.appendChild(view.target);
        } catch (e) {
            console.warn(`Failed to createView, reason=${e.message}, code=${e.code}`);
        }   
    }
    
    remoteVideoStream.on('isAvailableChanged', async () => {
        // Participant has switched video on.
        if (remoteVideoStream.isAvailable) {
            await renderVideo();

        // Participant has switched video off.
        } else {
            if (view) {
                view.dispose();
                view = undefined;
            }
        }
    });

    // Participant has video on initially.
    if (remoteVideoStream.isAvailable) {
        await renderVideo();
    }
}

// Start your local video stream.
// This will send your local video stream to remote participants so they can view it.
startVideoButton.onclick = async () => {
    try {
        const localVideoStream = await createLocalVideoStream();
        await call.startVideo(localVideoStream);
    } catch (error) {
        console.error(error);
    }
}

// Stop your local video stream.
// This will stop your local video stream from being sent to remote participants.
stopVideoButton.onclick = async () => {
    try {
        await call.stopVideo(localVideoStream);
    } catch (error) {
        console.error(error);
    }
}

/**
 * To render a LocalVideoStream, you need to create a new instance of VideoStreamRenderer, and then
 * create a new VideoStreamRendererView instance using the asynchronous createView() method.
 * You may then attach view.target to any UI element. 
 */
// Create a local video stream for your camera device
createLocalVideoStream = async () => {
    const camera = (await deviceManager.getCameras())[0];
    if (camera) {
        return new LocalVideoStream(camera);
    } else {
        console.error(`No camera device found on the system`);
    }
}
// Display your local video stream preview in your UI
displayLocalVideoStream = async () => {
    try {
        localVideoStreamRenderer = new VideoStreamRenderer(localVideoStream);
        const view = await localVideoStreamRenderer.createView();
        localVideoContainer.hidden = false;
        localVideoContainer.appendChild(view.target);
    } catch (error) {
        console.error(error);
    } 
}
// Remove your local video stream preview from your UI
removeLocalVideoStream = async() => {
    try {
        localVideoStreamRenderer.dispose();
        localVideoContainer.hidden = true;
    } catch (error) {
        console.error(error);
    } 
}

// End the current call
hangUpCallButton.addEventListener("click", async () => {
    // end the current call
    await call.hangUp();
});
```                                                          

## <a name="run-the-code"></a>Exécuter le code
Utilisez `webpack-dev-server` pour créer et exécuter votre application. Exécutez la commande suivante pour créer un bundle de l’application hôte sur un serveur web local :

```console
npx webpack-dev-server --entry ./client.js --output bundle.js --debug --devtool inline-source-map
```
Ouvrez votre navigateur, puis accédez à http://localhost:8080/ sous deux onglets. Les éléments suivants doivent s’afficher : :::image type="content" source="../../media/javascript/1-on-1-video-calling-a.png" alt-text="Page d’appel vidéo 1 sur 1 - a":::

Sous le premier onglet, entrez un jeton d’accès utilisateur valide et, sous l’autre onglet, entrez un jeton d’accès utilisateur valide différent. (Reportez-vous à la [documentation sur les jetons d’accès utilisateur](../../../access-tokens.md) si vous n’avez pas encore de jetons pouvant être utilisés.)
Sous les deux onglets, cliquez sur le bouton « Initialiser l’agent d’appel ». Les éléments suivants doivent s’afficher : :::image type="content" source="../../media/javascript/1-on-1-video-calling-b.png" alt-text="Page d’appel vidéo 1 sur 1 - b":::

Sous le premier onglet, entrez l’identité de l’utilisateur ACS du deuxième onglet, puis cliquez sur le bouton « Démarrer l’appel ». Le premier onglet démarre l’appel sortant au deuxième onglet, et le bouton « Accepter l’appel » du deuxième onglet est maintenant activé : :::image type="content" source="../../media/javascript/1-on-1-video-calling-c.png" alt-text="Page d’appel vidéo 1 sur 1 - c":::

Sous le deuxième onglet, cliquez sur le bouton « Accepter l’appel ». L’appel obtient alors une réponse et est connecté. Les éléments suivants doivent s’afficher : :::image type="content" source="../../media/javascript/1-on-1-video-calling-d.png" alt-text="Page d’appel vidéo 1 sur 1 - d":::

Les deux onglets se trouvent maintenant dans un appel vidéo 1 à 1. Les deux onglets peuvent entendre l’audio de l’autre et voir le flux vidéo de l’autre.
