---
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.date: 11/20/2020
ms.topic: include
ms.author: parkerra
ms.service: azure-spatial-anchors
ms.openlocfilehash: b3aed457f3c623a1c9916cc1ff0f5a5d6787064b
ms.sourcegitcommit: e41827d894a4aa12cbff62c51393dfc236297e10
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2021
ms.locfileid: "131570736"
---
## <a name="putting-everything-together"></a>Regroupement

Voici comment le fichier de classe `AzureSpatialAnchorsScript` complet doit se présenter une fois que les différents éléments ont été mis en ensemble. Vous pouvez vous en servir d’élément de comparaison pour votre propre fichier et pour repérer les différences.

```csharp
using Microsoft.Azure.SpatialAnchors;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using UnityEngine;
using UnityEngine.XR.WSA;
using UnityEngine.XR.WSA.Input;

public class AzureSpatialAnchorsScript : MonoBehaviour
{   
    /// <summary>
    /// The sphere prefab.
    /// </summary>
    public GameObject spherePrefab;

    /// <summary>
    /// Set this string to the Spatial Anchors account id provided in the Spatial Anchors resource.
    /// </summary>
    protected string SpatialAnchorsAccountId = "Set me";

    /// <summary>
    /// Set this string to the Spatial Anchors account key provided in the Spatial Anchors resource.
    /// </summary>
    protected string SpatialAnchorsAccountKey = "Set me";

    /// <summary>
    /// Set this string to the Spatial Anchors account domain provided in the Spatial Anchors resource.
    /// </summary>
    protected string SpatialAnchorsAccountDomain = "Set me";

    /// <summary>
    /// Our queue of actions that will be executed on the main thread.
    /// </summary>
    private readonly Queue<Action> dispatchQueue = new Queue<Action>();

    /// <summary>
    /// Use the recognizer to detect air taps.
    /// </summary>
    private GestureRecognizer recognizer;

    protected CloudSpatialAnchorSession cloudSpatialAnchorSession;

    /// <summary>
    /// The CloudSpatialAnchor that we either 1) placed and are saving or 2) just located.
    /// </summary>
    protected CloudSpatialAnchor currentCloudAnchor;

    /// <summary>
    /// True if we are 1) creating + saving an anchor or 2) looking for an anchor.
    /// </summary>
    protected bool tapExecuted = false;

    /// <summary>
    /// The ID of the CloudSpatialAnchor that was saved. Use it to find the CloudSpatialAnchor
    /// </summary>
    protected string cloudSpatialAnchorId = "";

    /// <summary>
    /// The sphere rendered to show the position of the CloudSpatialAnchor.
    /// </summary>
    protected GameObject sphere;
    protected Material sphereMaterial;

    /// <summary>
    /// Indicate if we are ready to save an anchor. We can save an anchor when value is greater than 1.
    /// </summary>
    protected float recommendedForCreate = 0;

    // Start is called before the first frame update
    void Start()
    {
        recognizer = new GestureRecognizer();

        recognizer.StartCapturingGestures();

        recognizer.SetRecognizableGestures(GestureSettings.Tap);

        recognizer.Tapped += HandleTap;

        InitializeSession();
    }

    // Update is called once per frame
    void Update()
    {
        lock (dispatchQueue)
        {
            if (dispatchQueue.Count > 0)
            {
                dispatchQueue.Dequeue()();
            }
        }
    }

    /// <summary>
    /// Queues the specified <see cref="Action"/> on update.
    /// </summary>
    /// <param name="updateAction">The update action.</param>
    protected void QueueOnUpdate(Action updateAction)
    {
        lock (dispatchQueue)
        {
            dispatchQueue.Enqueue(updateAction);
        }
    }

    /// <summary>
    /// Cleans up objects.
    /// </summary>
    public void CleanupObjects()
    {
        if (sphere != null)
        {
            Destroy(sphere);
            sphere = null;
        }

        if (sphereMaterial != null)
        {
            Destroy(sphereMaterial);
            sphereMaterial = null;
        }

        currentCloudAnchor = null;
    }

    /// <summary>
    /// Cleans up objects and stops the CloudSpatialAnchorSessions.
    /// </summary>
    public void ResetSession(Action completionRoutine = null)
    {
        Debug.Log("ASA Info: Resetting the session.");

        if (cloudSpatialAnchorSession.GetActiveWatchers().Count > 0)
        {
            Debug.LogError("ASA Error: We are resetting the session with active watchers, which is unexpected.");
        }

        CleanupObjects();

        this.cloudSpatialAnchorSession.Reset();

        lock (this.dispatchQueue)
        {
            this.dispatchQueue.Enqueue(() =>
            {
                if (cloudSpatialAnchorSession != null)
                {
                    cloudSpatialAnchorSession.Stop();
                    cloudSpatialAnchorSession.Dispose();
                    Debug.Log("ASA Info: Session was reset.");
                    completionRoutine?.Invoke();
                }
                else
                {
                    Debug.LogError("ASA Error: cloudSpatialAnchorSession was null, which is unexpected.");
                }
            });
        }
    }

    /// <summary>
    /// Initializes a new CloudSpatialAnchorSession.
    /// </summary>
    void InitializeSession()
    {
        Debug.Log("ASA Info: Initializing a CloudSpatialAnchorSession.");

        if (string.IsNullOrEmpty(SpatialAnchorsAccountId))
        {
            Debug.LogError("No account id set.");
            return;
        }

        if (string.IsNullOrEmpty(SpatialAnchorsAccountKey))
        {
            Debug.LogError("No account key set.");
            return;
        }

        cloudSpatialAnchorSession = new CloudSpatialAnchorSession();

        cloudSpatialAnchorSession.Configuration.AccountId = SpatialAnchorsAccountId.Trim();
        cloudSpatialAnchorSession.Configuration.AccountKey = SpatialAnchorsAccountKey.Trim();
        cloudSpatialAnchorSession.Configuration.AccountDomain = SpatialAnchorsAccountDomain.Trim();

        cloudSpatialAnchorSession.LogLevel = SessionLogLevel.All;

        cloudSpatialAnchorSession.Error += CloudSpatialAnchorSession_Error;
        cloudSpatialAnchorSession.OnLogDebug += CloudSpatialAnchorSession_OnLogDebug;
        cloudSpatialAnchorSession.SessionUpdated += CloudSpatialAnchorSession_SessionUpdated;
        cloudSpatialAnchorSession.AnchorLocated += CloudSpatialAnchorSession_AnchorLocated;
        cloudSpatialAnchorSession.LocateAnchorsCompleted += CloudSpatialAnchorSession_LocateAnchorsCompleted;

        cloudSpatialAnchorSession.Start();

        Debug.Log("ASA Info: Session was initialized.");
    }

    private void CloudSpatialAnchorSession_Error(object sender, SessionErrorEventArgs args)
    {
        Debug.LogError("ASA Error: " + args.ErrorMessage );
    }

    private void CloudSpatialAnchorSession_OnLogDebug(object sender, OnLogDebugEventArgs args)
    {
        Debug.Log("ASA Log: " + args.Message);
        System.Diagnostics.Debug.WriteLine("ASA Log: " + args.Message);
    }

    private void CloudSpatialAnchorSession_SessionUpdated(object sender, SessionUpdatedEventArgs args)
    {
        Debug.Log("ASA Log: recommendedForCreate: " + args.Status.RecommendedForCreateProgress);
        recommendedForCreate = args.Status.RecommendedForCreateProgress;
    }

    private void CloudSpatialAnchorSession_AnchorLocated(object sender, AnchorLocatedEventArgs args)
    {
        switch (args.Status)
        {
            case LocateAnchorStatus.Located:
                Debug.Log("ASA Info: Anchor located! Identifier: " + args.Identifier);
                QueueOnUpdate(() =>
                {
                    // Create a green sphere.
                    sphere = GameObject.Instantiate(spherePrefab, Vector3.zero, Quaternion.identity) as GameObject;
                    sphere.AddComponent<WorldAnchor>();
                    sphereMaterial = sphere.GetComponent<MeshRenderer>().material;
                    sphereMaterial.color = Color.green;

                    // Get the WorldAnchor from the CloudSpatialAnchor and use it to position the sphere.
                    sphere.GetComponent<UnityEngine.XR.WSA.WorldAnchor>().SetNativeSpatialAnchorPtr(args.Anchor.LocalAnchor);

                    // Clean up state so that we can start over and create a new anchor.
                    cloudSpatialAnchorId = "";
                    tapExecuted = false;
                });
                break;
            case LocateAnchorStatus.AlreadyTracked:
                Debug.Log("ASA Info: Anchor already tracked. Identifier: " + args.Identifier);
                break;
            case LocateAnchorStatus.NotLocated:
                Debug.Log("ASA Info: Anchor not located. Identifier: " + args.Identifier);
                break;
            case LocateAnchorStatus.NotLocatedAnchorDoesNotExist:
                Debug.LogError("ASA Error: Anchor not located does not exist. Identifier: " + args.Identifier);
                break;
        }
    }

    private void CloudSpatialAnchorSession_LocateAnchorsCompleted(object sender, LocateAnchorsCompletedEventArgs args)
    {
        Debug.Log("ASA Info: Locate anchors completed. Watcher identifier: " + args.Watcher.Identifier);
    }

    /// <summary>
    /// Called by GestureRecognizer when a tap is detected.
    /// </summary>
    /// <param name="tapEvent">The tap.</param>    
    public void HandleTap(TappedEventArgs tapEvent)
    {
        if (tapExecuted)
        {
            return;
        }
        
        tapExecuted = true;

        // We have saved an anchor, so we will now look for it.
        if (!String.IsNullOrEmpty(cloudSpatialAnchorId))
        {
            Debug.Log("ASA Info: We will look for a placed anchor.");

            ResetSession(() =>
            {
                InitializeSession();

                // Create a Watcher to look for the anchor we created.
                AnchorLocateCriteria criteria = new AnchorLocateCriteria();
                criteria.Identifiers = new string[] { cloudSpatialAnchorId };
                cloudSpatialAnchorSession.CreateWatcher(criteria);

                Debug.Log("ASA Info: Watcher created. Number of active watchers: " + cloudSpatialAnchorSession.GetActiveWatchers().Count);
            });
            return;
        }

        Debug.Log("ASA Info: We will create a new anchor.");

        // Clean up any anchors that have been placed.
        CleanupObjects();

        // Construct a Ray using forward direction of the HoloLens.
        Ray GazeRay = new Ray(tapEvent.headPose.position, tapEvent.headPose.forward);

        // Raycast to get the hit point in the real world.
        RaycastHit hitInfo;
        Physics.Raycast(GazeRay, out hitInfo, float.MaxValue);

        this.CreateAndSaveSphere(hitInfo.point);
    }

    /// <summary>
    /// Creates a sphere at the hit point, and then saves a CloudSpatialAnchor there.
    /// </summary>
    /// <param name="hitPoint">The hit point.</param>
    protected virtual void CreateAndSaveSphere(Vector3 hitPoint)
    {
        // Create a white sphere.
        sphere = GameObject.Instantiate(spherePrefab, hitPoint, Quaternion.identity) as GameObject;
        sphere.AddComponent<WorldAnchor>();
        sphereMaterial = sphere.GetComponent<MeshRenderer>().material;
        sphereMaterial.color = Color.white;
        Debug.Log("ASA Info: Created a local anchor.");

        // Create the CloudSpatialAnchor.
        currentCloudAnchor = new CloudSpatialAnchor();

        // Set the LocalAnchor property of the CloudSpatialAnchor to the WorldAnchor component of our white sphere.
        WorldAnchor worldAnchor = sphere.GetComponent<WorldAnchor>();
        if (worldAnchor == null)
        {
            throw new Exception("ASA Error: Couldn't get the local anchor pointer.");
        }

        // Save the CloudSpatialAnchor to the cloud.
        currentCloudAnchor.LocalAnchor = worldAnchor.GetNativeSpatialAnchorPtr();
        Task.Run(async () =>
        {
            // Wait for enough data about the environment.
            while (recommendedForCreate < 1.0F)
            {
                await Task.Delay(330);
            }

            bool success = false;
            try
            {
                QueueOnUpdate(() =>
                {
                    // We are about to save the CloudSpatialAnchor to the Azure Spatial Anchors, turn it yellow.
                    sphereMaterial.color = Color.yellow;
                });

                await cloudSpatialAnchorSession.CreateAnchorAsync(currentCloudAnchor);
                success = currentCloudAnchor != null;

                if (success)
                {
                    // Allow the user to tap again to clear state and look for the anchor.
                    tapExecuted = false;

                    // Record the identifier to locate.
                    cloudSpatialAnchorId = currentCloudAnchor.Identifier;

                    QueueOnUpdate(() =>
                    {
                        // Turn the sphere blue.
                        sphereMaterial.color = Color.blue;
                    });

                    Debug.Log("ASA Info: Saved anchor to Azure Spatial Anchors! Identifier: " + cloudSpatialAnchorId);
                }
                else
                {
                    sphereMaterial.color = Color.red;
                    Debug.LogError("ASA Error: Failed to save, but no exception was thrown.");
                }
            }
            catch (Exception ex)
            {
                QueueOnUpdate(() =>
                {
                    sphereMaterial.color = Color.red;
                });
                Debug.LogError("ASA Error: " + ex.Message);
            }
        });
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez découvert l’utilisation d’Azure Spatial Anchors dans une nouvelle application Unity HoloLens. Pour en savoir plus sur l’utilisation d’Azure Spatial Anchors dans une nouvelle application Android, passez au tutoriel suivant.

> [!div class="nextstepaction"]
> [Démarrage d’une nouvelle application Android](../articles/spatial-anchors/tutorials/tutorial-new-android-app.md)
