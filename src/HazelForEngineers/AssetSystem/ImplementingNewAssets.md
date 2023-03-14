<div class="title"> 
    <img src="/res/HazelGradientLogo-Square.png" alt="Hazel Logo" />
    <h1> Implementing New Assets </h1>
</div>

## Converting Your Data Structure To Be An Asset
* * *

### The Data Structure

In order to make your data structure (class or struct) compatible with the Asset System, it must extend the `Asset` class.

For this example we will be writing a simplified version of the `PhysicsMaterial` class:
```cpp
struct PhysicsMaterial : public Asset
{
	float StaticFriction;
	float DynamicFriction;
	float Bounciness;

	PhysicsMaterial() = default;
	PhysicsMaterial(float staticFriction, float dynamicFriction, float bounciness);
};
```

### The Asset Type

Inheriting from the `Asset` class isn't enough though. The Asset System needs a way to identify any asset, which is why we need to expand the `AssetType` enum.

The enum currently looks like this:
```cpp
enum class AssetType : uint16_t
{
	None = 0,
	Scene = 1,
	MeshAsset = 2,
	Mesh = 3,
	Material = 4,
	Texture = 5,
	EnvMap = 6,
	Audio = 7
};
```

We will go ahead and add a value called `PhysicsMat`, and assign it to a value of 8.
```cpp
enum class AssetType : uint16_t
{
	None = 0,
	Scene = 1,
	MeshAsset = 2,
	Mesh = 3,
	Material = 4,
	Texture = 5,
	EnvMap = 6,
	Audio = 7,
	PhysicsMat = 8
};
```
**NOTE: You shouldn't change the value of the already existing enum values. Just add yours to the end, and give it the next available value!**

We also need to modify these two functions in order for our asset to be tracked properly:
```cpp
inline AssetType AssetTypeFromString(const std::string& assetType)
{
	if (assetType == "None")        return AssetType::None;
	if (assetType == "Scene")       return AssetType::Scene;
	if (assetType == "MeshAsset")   return AssetType::MeshAsset;
	if (assetType == "Mesh")        return AssetType::Mesh;
	if (assetType == "Material")    return AssetType::Material;
	if (assetType == "Texture")     return AssetType::Texture;
	if (assetType == "EnvMap")      return AssetType::EnvMap;
	if (assetType == "Audio")       return AssetType::Audio;

	HZ_CORE_ASSERT(false, "Unknown Asset Type");
	return AssetType::None;
}

inline const char* AssetTypeToString(AssetType assetType)
{
	switch (assetType)
	{
		case AssetType::None:        return "None";
		case AssetType::Scene:       return "Scene";
		case AssetType::MeshAsset:   return "MeshAsset";
		case AssetType::Mesh:        return "Mesh";
		case AssetType::Material:    return "Material";
		case AssetType::Texture:     return "Texture";
		case AssetType::EnvMap:      return "EnvMap";
		case AssetType::Audio:       return "Audio";
	}

	HZ_CORE_ASSERT(false, "Unknown Asset Type");
	return "None";
}
```

We need to ensure that our enum value can be properly converted from and to a string.
```cpp
inline AssetType AssetTypeFromString(const std::string& assetType)
{
	if (assetType == "None")        return AssetType::None;
	if (assetType == "Scene")       return AssetType::Scene;
	if (assetType == "MeshAsset")   return AssetType::MeshAsset;
	if (assetType == "Mesh")        return AssetType::Mesh;
	if (assetType == "Material")    return AssetType::Material;
	if (assetType == "Texture")     return AssetType::Texture;
	if (assetType == "EnvMap")      return AssetType::EnvMap;
	if (assetType == "Audio")       return AssetType::Audio;
	if (assetType == "PhysicsMat")	return AssetType::PhysicsMat;

	HZ_CORE_ASSERT(false, "Unknown Asset Type");
	return AssetType::None;
}

inline const char* AssetTypeToString(AssetType assetType)
{
	switch (assetType)
	{
		case AssetType::None:        return "None";
		case AssetType::Scene:       return "Scene";
		case AssetType::MeshAsset:   return "MeshAsset";
		case AssetType::Mesh:        return "Mesh";
		case AssetType::Material:    return "Material";
		case AssetType::Texture:     return "Texture";
		case AssetType::EnvMap:      return "EnvMap";
		case AssetType::Audio:       return "Audio";
		case AssetType::PhysicsMat:  return "PhysicsMat";
	}

	HZ_CORE_ASSERT(false, "Unknown Asset Type");
	return "None";
}
```

Our asset class will also need to override the `GetAssetType()` method from `Asset`, and it will need to provide a static method called `GetStaticType()`.

**NOTE: Your Asset won't work if you don't provide these methods! If you don't provide `GetStaticType()` the code may not even compile!**

```cpp
struct PhysicsMaterial : public Asset
{
	float StaticFriction;
	float DynamicFriction;
	float Bounciness;

	PhysicsMaterial(float staticFriction, float dynamicFriction, float bounciness);

	static AssetType GetStaticType() { return AssetType::PhysicsMat; }
	virtual AssetType GetAssetType() const override { return GetStaticType(); }
};
```

This is required by certain templated methods in the Asset System.

If you want to associate your asset with a specific file type, you need to modify the `s_AssetExtensionMap` map in `Hazel/Asset/AssetExtensions.h`.

In theory this is all the code necessary in order to have the Asset System recognize your asset.
This isn't enough to have the Asset Manager import your asset correctly however. In order to have that functionality we need to implement an `AssetSerializer`.

## Importing/Exporting The Asset
* * *

### Writing the Serializer

In order to allow the Asset System to import the asset correctly we need to provide a class that the it can interface with.
This is done by creating a class that inherits from the `AssetSerializer` class.

The `AssetSerializer` class contains two pure virtual methods.
```cpp
virtual void Serialize(const AssetMetadata& metadata, const Ref<Asset>& asset) const = 0;
virtual bool TryLoadData(const AssetMetadata& metadata, Ref<Asset>& asset) const = 0;
```

`AssetSerializer::Serialize` is called when the Asset System requires an asset to be saved to disk.
`AssetSerializer::TryLoadData` is called when an asset that hasn't been imported yet is requested by the engine.

Let's go ahead and implement those methods in a class called `PhysicsMaterialSerializer`.

**NOTE: Hazel serializes data using YAML for certain assets. This isn't always the case though.**

```cpp
class PhysicsMaterialSerializer : public AssetSerializer
{
public:
	virtual void Serialize(const AssetMetadata& metadata, const Ref<Asset>& asset) const override
	{
		// We can call .As<>() on a Ref instance in order to cast it to a different type
		Ref<PhysicsMaterial> material = asset.As<PhysicsMaterial>();

		YAML::Emitter out;

		out << YAML::BeginMap;
		out << YAML::Key << "StaticFriction" << material->StaticFriction;
		out << YAML::Key << "DynamicFriction" << material->DynamicFriction;
		out << YAML::Key << "Bounciness" << material->Bounciness;
		out << YAML::EndMap;

		// It's important that you use AssetManager::GetFileSystemPath(metadata) here since metadata.FilePath isn't in the correct format
		std::ofstream fout(AssetManager::GetFileSystemPath(metadata.FilePath));
		fout << out.c_str();
	}

	virtual bool TryLoadData(const AssetMetadata& metadata, Ref<Asset>& asset) const override
	{
		// Load the YAML file from disk
		// It's important that you use AssetManager::GetFileSystemPath(metadata) here since metadata.FilePath isn't in the correct format
		std::ifstream stream(AssetManager::GetFileSystemPath(metadata));
		if (!stream.is_open())
			return false; // We couldn't load the file for some reason, signal the Asset System that we failed
 
		std::stringstream strStream;
		strStream << stream.rdbuf();

		YAML::Node data = YAML::Load(strStream.str());

		float staticFriction = data["StaticFriction"].as<float>();
		float dynamicFriction = data["DynamicFriction"].as<float>();
		float bounciness = data["Bounciness"].as<float>();

		// In order to create a RefCounted object we need to call Ref<Type>::Create(Args)
		asset = Ref<PhysicsMaterial>::Create(staticFriction, dynamicFriction, bounciness);

		// Here we assign the asset id to this instance of the asset
		asset->Handle = metadata.Handle;

		// We successfully loaded the asset
		return true;
	}
};
```

It's worth noting that the way you load the asset data heavily depends on the type of asset you're implementing.
Here's an example of how Hazel loads Textures:
```cpp
bool TextureSerializer::TryLoadData(const AssetMetadata& metadata, Ref<Asset>& asset) const
{
	asset = Texture2D::Create(AssetManager::GetFileSystemPathString(metadata));
	asset->Handle = metadata.Handle;

	bool result = asset.As<Texture2D>()->Loaded();

	if (!result)
		asset->SetFlag(AssetFlag::Invalid, true);

	return result;
}
```

### Registering the Serializer

Now that we've written the serializer implementation, we need to register it to our `AssetType` value, so that the Asset Manager can understand that our serializer is meant to be use with our new asset type.

In order to do this we must modify the `AssetImporter::Init()` method in `Hazel/Asset/AssetImporter.cpp`. That method currently looks like this:
```cpp
void AssetImporter::Init()
{
	s_Serializers[AssetType::Texture] = CreateScope<TextureSerializer>();
	s_Serializers[AssetType::MeshAsset] = CreateScope<MeshAssetSerializer>();
	s_Serializers[AssetType::Mesh] = CreateScope<MeshSerializer>();
	s_Serializers[AssetType::EnvMap] = CreateScope<EnvironmentSerializer>();
}
```

As you can see we call `CreateScope<T>` with the serializer we want to register as the template argument, and store it in a map that has an `AssetType` as a key.
This allows the Asset Manager to query the map for a given `AssetType` and get the serializer for that type back.

We need to add our new serializer to this map, making sure to have our own `AssetType` as the key.

```cpp
void AssetImporter::Init()
{
	s_Serializers[AssetType::Texture] = CreateScope<TextureSerializer>();
	s_Serializers[AssetType::MeshAsset] = CreateScope<MeshAssetSerializer>();
	s_Serializers[AssetType::Mesh] = CreateScope<MeshSerializer>();
	s_Serializers[AssetType::EnvMap] = CreateScope<EnvironmentSerializer>();

	// Associate our serializer with our AssetType
	s_Serializers[AssetType::PhysicsMat] = CreateScope<PhysicsMaterialSerializer>();
}
```

And that should be all you need to do in order to integrate your asset with the Asset Manager! This will also ensure that the ContentBrowserPanel deals with your asset properly!
