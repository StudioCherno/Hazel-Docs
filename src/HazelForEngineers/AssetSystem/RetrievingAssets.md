<div class="title"> 
    <img src="/res/HazelGradientLogo-Square.png" alt="Hazel Logo" />
    <h1> Retrieving Assets </h1>
</div>

Assets in Hazel are [reference counted](https://en.wikipedia.org/wiki/Reference_counting), and should always be wrapped in a ```Ref``` instance. Meaning you should pretty much never allocate an Asset on the stack, since Hazel uses intrusive reference counting, meaning the reference count is stored in the asset instance, not in the ```Ref``` instance.

In order to retrieve an Asset you have to request it from the ```AssetManager``` class.

## Example 1
```cpp
...
Ref<PhysicsMaterial> material = AssetManager::GetAsset<PhysicsMaterial>(assetID);
...
```

## Example 2

It's also possible to get an asset by the file path (relative to the projects "Assets" directory).

```cpp
AssetHandle assetID = AssetManager::GetAssetHandleFromFilePath("Physics/PlayerMaterial.hpm");
Ref<PhysicsMaterial> material = AssetManager::GetAsset<PhysicsMaterial>(assetID);
// OR
Ref<PhysicsMaterial> material = AssetManager::GetAsset<PhysicsMaterial>("Physics/PlayerMaterial.hpm");
```

**NOTE: Referencing Assets by ID is the preferred method**

# Creating New Assets

Most of the time you won't need to create assets programmatically, but Hazel does support doing so.

```cpp
Ref<PhysicsMaterial> newMaterial = AssetManager::CreateNewAsset<PhysicsMaterial>("MyMaterial.hpm", "Physics/", ...);
```

The first two parameters are ``` filename ``` and ``` directoryPath ```. After the first two parameters you can pass a variable number of parameters that will be used to construct the Asset instance.

```cpp
Ref<SomeTextAsset> asset = AssetManager::CreateNewAsset<SomeTextAsset>("MyTextFile.txt", "Docs/", "This string will be passed to SomeTextAsset's constructor!");
```
