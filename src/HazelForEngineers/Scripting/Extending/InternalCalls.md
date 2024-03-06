<div class="title"> 
    <img src="/res/HazelGradientLogo-Square.png" alt="Hazel Logo" />
    <h1> Internal Calls </h1>
</div>

# Adding Internal Calls (Interop with C# and C++)
You should only add an internal call if you want C# code to call some function in C++, you might want this if you want the added performance of C++, or because you need to interface with, or expose a C++ API to C#.

I'll start by providing guidelines for adding internal calls, *please* make sure that you follow these guidelines, they're here to make it far easier for others to understand and maintain code that you write, and for the sake of consistency.

## Guidelines for writing internal calls in C#
When you add the internal calls to the C# API here are some things you need to consider:
1. Internal methods should **always** be defined inside the `InternalCalls.cs` file. Never in the classes that use these methods.
2. Internal methods should **always** be defined inside of a `#region` block to keep the file organized. You can take a look at how other regions are written.
3. Internal methods should **always** be named in this format: `ClassName_FunctionName`, e.g `TransformComponent_GetTranslation`, or `MeshComponent_GetMaterial`.
4. Internal methods should **always** be defined as `internal static delegate* unmanaged`
5. Internal methods should **always** have the same name in C++ as in C#, so a method called `TransformComponent_GetTranslation` should also be called that in C++.
6. If you need to pass `bool` or `object` to an internal call they should be passed as `Coral.Bool32` and `Coral.NativeInstance<object>`.
7. Internal calls follow `Func` style syntax for declaring parameters and return types, e.g `internal static delegate* unmanaged<int, int, Coral.Bool32>` will take two integers and return a `Coral.Bool32`.

These are some of the basic things you need to keep in mind, feel free to let me know if you think these guidelines should be updated!

## Guidelines for writing internal calls in C++
When you add the internal calls to the C++ API these are some things you need to keep in mind:
1. The C++ implementation of an internal call should **always** be declared in the `ScriptGlue.h` file, and defined in `ScripGlue.cpp`.
2. In order to register the internal call with the C++ API you have to add this line: `HZ_ADD_INTERNAL_CALL(ClassName_FunctionName);` to `ScriptGlue::RegisterInternalCalls`, preferably in the same order that it's declared in `InternalCalls.cs`.
3. If an internal method as to log a message it should preferably log these messages to the Hazelnut console, e.g using `HZ_CONSOLE_LOG_INFO`. This isn't required though and mainly depends on what you're logging.
4. Always add internal calls *inside* the `InternalCalls` namespace in C++. This is **required** in order to register the function

There's understandably a lot more to keep in mind, but the key point is to follow these guidelines, and to make sure the code you add is consistent with the code that's already in the C++ and C# API.

## Example
Here's a very basic example of how to add an internal call to the scripting API, we'll start with declaring the internal call in C#:

Imagine we have a custom struct in C#, and we want to populate that struct with some data from C++, and have our internal call return `true` if it succeeds. Here's what struct might look like:
```cs
// You have to add this attribute to structs that you want to pass to C++
[StructLayout(LayoutKind.Sequential)]
public struct MyCustomData
{
	public float MyFloat;
	public int SomeInt;
}
```

```cs
// InternalCalls.cs

...

#region MyCustomComponent

internal static delegate* unmanaged<ulong, MyCustomData*, Coral.Bool32> MyCustomComponent_GetCustomStruct;

#endregion

...

```

Here you can see we define the internal call, it returns a `bool`, and takes in two parameters, the `ulong` parameter is just for consistency here, all C# components have to pass the entity's ID to C++ so the engine can know what entity the component belongs to.

And it also takes a pointer to our custom struct, this means that we expect C++ to write some data into this struct, but it can also read from it.

That's really all you need to do in order to define the method in C#. Then in C++ we'll add this code:
```cpp
// MyCustomData struct (can be a class in C++ but use struct where possible)
struct MyCustomData
{
	float MyFloat;
	int SomeInt;
};

// ScriptGlue.h

namespace InternalCalls {

...

#pragma region MyCustomComponent

Coral::Bool32 MyCustomComponent_GetCustomStruct(uint64_t entityID, OutParam<MyCustomData> outData);

#pragma endregion

...
}
```
and then in `ScriptGlue.cpp` we add the implementation of this function:
```cpp
// ScriptGlue.cpp

namespace InternalCalls {

...

#pragma region MyCustomComponent

Coral::Bool32 MyCustomComponent_GetCustomStruct(uint64_t entityID, OutParam<MyCustomData> outData)
{
	// NOTE: Dummy code, look at the existing internal calls for reference
	if (!is_entity_valid)
	{
		// Log some error here
		return false;
	}

	if (!entity_has_MyCustomComponent)
	{
		// Should almost never happen but log error if it does
		return false;
	}

	*outData = myCustomComponent.MyData;
	// OR:
	outData->MyFloat = someFloatValue;
	outData->SomeInt = someIntValue;
	return true;
}

#pragma endregion

...
}
```

This just covers the basics of adding internal calls, but it should give you an idea as to how it's done.

Also keep in mind that your C++ and C# struct need to have the exact same memory layout, including things like padding. This *can* involve manually adding padding to your C# struct.