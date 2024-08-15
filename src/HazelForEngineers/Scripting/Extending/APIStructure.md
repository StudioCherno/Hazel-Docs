<div class="title"> 
    <img src="/res/Hazel-IconLogo-2023.png" alt="Hazel Logo" />
    <h1> C# API Structure </h1>
</div>
This page will simply explain the general structure of the C# API, what features / systems belong where, what files are relevant to what topic, etc...

## Hazel-ScriptCore Folder Structure
All the C# source files belong in `Hazel-ScriptCore/Source`, no core .cs files should ever exist outside this folder.

### Attributes/
This folder isn't too useful for extending **just** the C# API, it's more useful if you want to extend the Hazelnut Editor. But in short this folder should contain any custom C# attributes that we want to add.

### Audio/
Contains all the Audio data / utility classes, but NOT the `AudioComponent`

### Core/
This folder contains some useful core classes, e.g `Application.cs`, `Input.cs` and `Log.cs`. These classes are usually implemented as `static` for simplicity and ease-of-use.

This folder also contains one of the most important files you'll need for extending the C# API: `InternalCalls.cs`. Which is the file that contains all methods that we need to implement in C++.

### Math/
Contains all the math classes, you'll rarely need to add new files to this folder, but you may want to add methods to the classes that are already defined in those files.

`Mathf.cs` is where you'd put general math utility methods like `Abs`, `Min`, `Max`, `Lerp` etc...

### Physics/
Contains all Physics data and utility classes / structs, it's the folder where all the collider data classes are defined, it also contains the static `Physics` class that contains useful physics related methods like `Raycast`, `Overlap` methods, etc...

### Renderer/
This folder contains all the renderer / graphics related classes like `Mesh`, `StaticMesh`, and `Material`, usually you won't have to extend these classes in any way.

### Scene/
This is the main folder that you'll be interested in if you want to extend the core C# API, it contains the `Components.cs`, `Entity.cs` and `Scene.cs`.

The `Components.cs` file is where you'll modify existing components, and add new components when you want to expose them from C++. I'll provide an example later on that shows roughly how to expose or extend components.

The `Entity.cs` file contains the Hazel Entity class, which all (or most) game scripts will be sub classed from, you'll very rarely have to modify this file, but if you have to keep in mind that most of the modifications you want to make can probably be done through a component instead.
And please try not to break backwards compatibility when you change this file, we want to avoid breaking game scripts as much as possible.

The `Scene.cs` will most likely not be used by game scripts directly too much, it contains methods related to interacting with the scene, spawning prefabs, creating and destroying entities, finding entities by ID or by name. Most game scripts will be interacting with the equivalent methods in `Entity.cs` instead of interacting with the `Scene` class directly.
