---
layout: default
title: Asset System
has_children: true
nav_order: 10
---

# Defintions

## Asset
An Asset can be any resource required in order for the project to function properly.

Some examples are:
*	Textures
*	Meshes/Models
*	Sound Effects

* * *

## Asset Manager
The **Asset Manager** is the system responsible for managing and serving Assets to the rest of the engine or project.
There's many ways that an Asset Manager can serve Assets, from a file on disk, from memory, from a network location or from a Virtual File System (Binary Blob).

Currently Hazels Asset Manager can only serve assets from a file on disk.

* * *

# Definitions

| Name				| Description										|
|:------------------|:--------------------------------------------------|
| AssetManager		| Main Asset System interface						|
| Asset				| Base Asset class that all assets inherit from		|
| AssetImporter		| Interface for serializing/deserializing assets	|
| AssetSerializer	| Asset (de)serialization implementation			|
| AssetMetadata		| Asset metadata									|
| AssetType			| Enum used to identify the type of Assets			|
| AssetFlag			| Enum with flags used by the AssetManager			|


## Asset Metadata
The `AssetMetadata` struct contains useful information about a given asset.

Here are the fields stored, and what they represent:

| Name								| Type						| Description																|
|:----------------------------------|:--------------------------|:--------------------------------------------------------------------------|
| Handle							| AssetHandle				| The unique ID used to represent this asset								|
| Type								| AssetType					| The type of this asset													|
| FilePath							| std::filesystem::path		| The file path of this asset relative to the projects "Assets" directory	|
| IsDataLoaded						| bool						| A boolean that determines if the asset data has been loaded				|

In order to get the `AssetMetadata` for a specific asset you need to get it from the `AssetManager` class.
```cpp
const AssetMetadata& metadata = AssetManager::GetMetadata(assetId);
// OR
const AssetMetadata& metadata = AssetManager::GetMetadata("some_file_path");
```

It's not guaranteed that the metadata is valid, if the asset doesn't exist the function will return an invalid `AssetMetadata` struct.
You can check if it's valid by calling the `IsValid()` method on the metadata instance.

## The Asset Registry

The Asset Registry is a file saved on disk that identifies what file belongs to what `AssetHandle`. This file should **NEVER** be manually modified or deleted.
If your project makes use of a version control system, always make sure that "AssetRegistry.hzr" is commited or you will end up having missing references in Hazelnut.

You should never interface with the Asset Registry directly through code, as it's managed entierly by the Asset Manager.
