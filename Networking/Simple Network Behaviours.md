# Simple Network Behaviour

## Creating the behaviour class

Network Behaviours can be created by extending the `HawkNetworkBehaviour` class like so:

```c#
public class MyNetworkBehaviour : HawkNetworkBehaviour
{

}
```

Then we can implement a simple RPC by overriding the RegisterRPCs method:

```c#
public class MyNetworkBehaviour : HawkNetworkBehaviour
{
    private byte RPC_TEST;
    
    protected override void RegisterRPCs(HawkNetworkObject networkObject)
    {
        base.RegisterRPCs(networkObject);

        RPC_TEST = networkObject.RegisterRPC(ClientTestRpc);
    }

    private void ClientTestRpc(HawkNetReader reader, HawkRPCInfo info)
    {
        Plugin.Logger.LogMessage("Hello World from " + info.sender.Name);
    }
}
```

In this example calling `networkObject.SendRPC(RPC_TEST, RPCRecievers.All);` would trigger the `ClientTestRpc` method on all clients in the lobby with this plugin. 
For more info on RPCs check out [the official documention](https://www.rubberbandgames.com/wobblylife/modsdkdocs/networksendingbasicrpcspage.html) 
(it's not the same names as in the plugin but the concept stays the same). 
We can send the rpc anywhere where the `networkObject` has been initialized such as the `NetworkPost` method:

```c#
protected override void NetworkPost(HawkNetworkObject networkObject)
{
    base.NetworkPost(networkObject);

    networkObject.SendRPC(RPC_TEST, RPCRecievers.All);
}
```

It's best to put a check at the start of methods where you are not sure if it's been initialized or not:

```c#
if(networkObject == null)
{
    return;
}
```

If you want a method to only be able to be called by the host you can add this:

```c#
if(networkObject == null || !networkObject.IsServer())
{
    return;
}
```

## Creating and registering the prefab

To be able to use it you need to register your behaviour, but this is a bit tricky as there are no official APIs to do so. 
The most reliable way i know is by creating a prefab and instantiating it. 
This also allows you to create multiple instances of your behaviour.

In the `Awake` method of our `Plugin` class we can create the prefab Game Object:

```c#
public class Plugin : BaseUnityPlugin
{
    internal static new ManualLogSource Logger;
    internal static GameObject networkedObjectPrefab;

    private void Awake()
    {
        Logger = base.Logger;
        
        // create prefab object
        networkedObjectPrefab = new GameObject("Test Networked Object");

        // ...
        
        Logger.LogInfo($"Plugin {MyPluginInfo.PLUGIN_GUID} is loaded!");
    }
}
```

To make the original not appear in game and stay loaded we need to move it to the `HideAndDontSave` scene by doing:

```c#
networkedObjectPrefab.hideFlags = HideFlags.HideAndDontSave;
```

Then we can add out components, including out network behaviour like so:

```c#
var networkBehaviour = networkedObjectPrefab.AddComponent<MyNetworkBehaviour>();
```

To be able to register the prefab and spawn it, it needs an assetID. This has to be set using reflection:

```c#
const string guid = "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee";
var assetIdField = typeof(HawkNetworkBehaviour).GetField("assetID", BindingFlags.Instance | BindingFlags.NonPublic);
assetIdField.SetValue(networkBehaviour, guid);
```

Replace the GUID with a new one generated using some tool such as [this website](https://guidgenerator.com/).

> [!IMPORTANT]
> This GUID has to be the same for all clients and has to be unique. Do not use them twice.

Finally you can register your prefab:

```c#
HawkNetworkManager.DefaultInstance.RegisterPrefab(networkBehaviour);
```

## Spawning the prefab

This prefab can then be spawned using either `NetworkPrefab.SpawnNetworkPrefab` or `HawkNetworkManager.DefaultInstance.InstantiateNetworkPrefab`. For example when loading into the Wobbly Island scene:

```c#
SceneManager.sceneLoaded += (scene, loadMode) =>
{
    if (scene.name == "WobblyIsland" && HawkNetworkManager.DefaultInstance.GetMe().IsHost)
    {
        HawkNetworkManager.DefaultInstance.InstantiateNetworkPrefab(networkedObjectPrefab, Vector3.zero,
        Quaternion.identity);
    }
};
```

Other scene names include: "MainMenu" and "ArcadeLobby". You can see them in the Object Explorer window of [Cinematic Unity Explorer](https://github.com/originalnicodr/CinematicUnityExplorer) / [Unity Explorer](https://github.com/sinai-dev/UnityExplorer).

## Full Code

`Plugin.cs`
```c#
using System.Reflection;
using BepInEx;
using BepInEx.Logging;
using HawkNetworking;
using UnityEngine;
using UnityEngine.SceneManagement;

namespace WLExampleMod;

[BepInPlugin(MyPluginInfo.PLUGIN_GUID, MyPluginInfo.PLUGIN_NAME, MyPluginInfo.PLUGIN_VERSION)]
public class Plugin : BaseUnityPlugin
{
    internal static new ManualLogSource Logger;

    internal static GameObject networkedObjectPrefab;

    private void Awake()
    {
        Logger = base.Logger;
        
        // create prefab object
        networkedObjectPrefab = new GameObject("Test Networked Object");
        
        // hide prefab
        networkedObjectPrefab.hideFlags = HideFlags.HideAndDontSave;

        // create network behaviour
        var networkBehaviour = networkedObjectPrefab.AddComponent<MyNetworkBehaviour>();

        // get asset id field and set it to a new guid
        const string guid = "ceadee34-8d8a-4546-9bbd-fc5a460405d4";
        var assetIdField = typeof(HawkNetworkBehaviour).GetField("assetID", BindingFlags.Instance | BindingFlags.NonPublic);
        assetIdField.SetValue(networkBehaviour, guid);

        // register prefab
        HawkNetworkManager.DefaultInstance.RegisterPrefab(networkBehaviour);

        SceneManager.sceneLoaded += (scene, loadMode) =>
        {
            if (scene.name == "WobblyIsland" && HawkNetworkManager.DefaultInstance.GetMe().IsHost)
            {
                // spawn prefab when wobbly island loaded
                HawkNetworkManager.DefaultInstance.InstantiateNetworkPrefab(networkedObjectPrefab, Vector3.zero,
                    Quaternion.identity);
            }
        };

        Logger.LogInfo($"Plugin {MyPluginInfo.PLUGIN_GUID} is loaded!");
    }
}
```

`MyNetworkBehaviour.cs`
```c#
using HawkNetworking;

namespace WLExampleMod;

public class MyNetworkBehaviour : HawkNetworkBehaviour
{
    private byte RPC_TEST;
    
    protected override void RegisterRPCs(HawkNetworkObject networkObject)
    {
        base.RegisterRPCs(networkObject);

        RPC_TEST = networkObject.RegisterRPC(ClientTestRpc);
    }

    protected override void NetworkPost(HawkNetworkObject networkObject)
    {
        base.NetworkPost(networkObject);
        
        networkObject.SendRPC(RPC_TEST, RPCRecievers.All);
    }

    private void ClientTestRpc(HawkNetReader reader, HawkRPCInfo info)
    {
        Plugin.Logger.LogMessage("Hello World from " + info.sender.Name);
    }
}
```

## For lazy people

Check [cs/Utils/NetworkPrefabHelper.cs](https://github.com/lstwo/wobbly-life-modding-guide/blob/main/cs/Utils/NetworkPrefabHelper.cs) for a helper class that does all this for you as long as you pass it the name, type and guid (and optionally components). Example:

```c#
var myNetworkPrefab = NetworkPrefabHelper.CreateNetworkPrefab(
    "My Network Object",
    typeof(MyNetworkBehaviour),
    "ce81feac-d5e1-44b6-8e1c-8a754ffd31e2",
    typeof(BoxCollider));

HawkNetworkManager.DefaultInstance.InstantiateNetworkPrefab(myNetworkPrefab, Vector3.zero, Quaternion.identity);
```

also has a Singleton helper for people who are even lazier:

```c#
NetworkPrefabHelper.AddNetworkSingleton(
    myNetworkPrefab.GetComponent<MyNetworkBehaviour>,
    (scene, mode) => scene.name == "WobblyIsland",
    (behaviour) => behaviour.transform.SetParent(anotherTransform));
```

This gets instantiated when the wobbly island scene is loaded and stays until you exit wobbly island. It also parents the behaviour's object to a different transform.
