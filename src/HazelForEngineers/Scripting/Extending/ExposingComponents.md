<div class="title"> 
    <img src="/res/Hazel-IconLogo-2023.png" alt="Hazel Logo" />
    <h1> Exposing Components from C++ to C# </h1>
</div>
This page will tell you all the necessary steps you'll need to follow in order to expose a component from C++ to C#.

Here are the outline of the step you'll need to complete to expose a component to C#:
1. Add `HZ_REGISTER_MANAGED_COMPONENT(MyCustomComponent);` to `ScriptGlue::RegisterComponentTypes`.
2. Implement the C# component by defining a class in `Components.cs` and make it inherit from `Component`.
3. Add the necessary internal calls to let the C# component interact with the C++ component and engine systems.

Please try to keep the `HZ_REGISTER_MANAGED_COMPONENT` in the same order as the the components internal calls, so if your internal calls are defined underneath e.g the `RigidBodyComponent` in `ScriptGlue.h` you should register your component and internal calls underneath the `RigidBodyComponent` in `ScriptGlue::RegisterComponentTypes` and `ScriptGlue::RegisterInternalCalls`. This helps keep the files organized.

The way you write a C# component highly depends on the component, but the basics are this:
1. You need to add a class to `Components.cs` that inherits from `Component`, like this: `public class MyComponent : Component`.
2. You need to add methods or properties that calls the necessary internal methods. (Refer to the previous page for info on how to do this)
3. Any internal method that needs to interface with your component has to take in the ID of the entity, you can access the ID from within the component by accessing `Entity.ID`.
4. Entity IDs are 64-bit unsigned integers, the C# type for that is called `ulong`, and the C++ type is called `uint64_t`.

## Quick Note on Heap Allocations in C#
One thing that has been a problem in the Hazel script core since the start are heap allocations when calling internal methods. I won't go too deep into heap allocations as a concept, but in C# when you create a new instance of a class by calling `new MyClass()`, that will result in that object being allocated on the heap.

In Hazels script core we've often been returning a new instance of a class after calling an internal method, here's a quick example from the `MeshColliderComponent`:
```cs

// This is not great because every time we call `MeshColliderComponent.ColliderMesh`
// we allocate a brand new `Mesh` instance every time even if the mesh hasn't changed internally.
public Mesh ColliderMesh
{
	get
	{
		unsafe 
		{
			AssetHandle meshHandle;
			InternalCalls.MeshColliderComponent_GetColliderMesh(Entity.ID, &meshHandle);
			return new Mesh(meshHandle);
		}
	}
}

// This is the better way of implementing this property:
// First we add a private member called m_ColliderMesh.
private Mesh m_ColliderMesh = null;

// Secondly we add an AssetHandle property called ColliderMeshHandle,
// this is what actually get's the handle of the Mesh
public AssetHandle ColliderMeshHandle
{
	get
	{
		unsafe 
		{
			AssetHandle colliderHandle;
			if (!InternalCalls.MeshColliderComponent_GetColliderMesh(Entity.ID, &colliderHandle))
			{
				return AssetHandle.Invalid;
			}
			return colliderHandle;
		}
	}
}

// Lastly we add a method that will return the Mesh.
public Mesh GetColliderMesh()
{
	// Here we make sure that the mesh handle that's stored internally is still valid
	// and return null if it isn't
	if (!ColliderMeshHandle.IsValid())
		return null;

	// Here we check if we either haven't set m_ColliderMesh or if the internal
	// mesh has changed, if so we create a new Mesh instance
	if (m_ColliderMesh == null || m_ColliderMesh.Handle != ColliderMeshHandle)
		m_ColliderMesh = new Mesh(ColliderMeshHandle);

	// Otherwise we just return the cached version of the Mesh instance
	return m_ColliderMesh;
}

```
I will say that this likely doesn't improve performance, in fact it might have *slightly* more overhead (it's still not noticeable), but this **does** reduce the number of heap allocations.

So *why* do we want to avoid heap allocations? Well for one they can cause performance issues if you allocate objects on the heap every frame, but it would also cause the C# Garbage Collector to run more often. I won't go into a deep explanation as to what the Garbage Collector is or why it's useful, the important part is that it cleans up unused C# objects, and doing so can cause a significant frame drop. So we want to prevent the Garbage Collector from running as much as possible, it will eventually run regardless so it's impossible to completely prevent it from running, and we don't want to prevent it, but we want to delay it.

Caching objects in C# isn't always the best thing to do, you don't really need to cache primitive types unless you really want to, and structs are *usually* allocated on the stack (although they *can* be allocated on the heap sometimes) so they're not as necessary to cache.

## Properties or Methods?
Should you use properties or methods for interacting with the C++ side of things? Well, it depends. I'd say if you're doing caching that would result in similar code as the example above you should probably use a method, not a property.

I won't go into too much detail on this since it's more a question of good C# code rather than Hazel specific code.
