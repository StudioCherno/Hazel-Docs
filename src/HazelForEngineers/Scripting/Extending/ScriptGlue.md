<div class="title"> 
    <img src="../res/Hazel-IconLogo-2023.png" alt="Hazel Logo" />
    <h1> New Script Glue Development Guidelines </h1>
</div>

> ### Note
> This is marked as "new"
> [until ScriptGen is merged](https://github.com/StudioCherno/Hazel/pull/91)
> as well as to keep the previous workflow docs around until they age out of
> release.

### Overview

Files in `Hazel/src/Hazel/Script/ScriptGlue` are treated as sources for script
glue code generation. They are compiled as-usual but to ensure interop
functions are correctly selected for generation they need to be named with
`ScriptGlue_` as the start of their identifier. They must also be in the
`Hazel::InternalCalls` namespace.

These are processed into both the script glue binding code -- i.e. assigning
function pointers into the managed `InternalCalls` delegates, and initial
generation of those delegates themselves. This nominally means that the only
requirement to add a new script-accessible function is to write it into a
source file in this folder with a compatible prototype.

There is some special type handling which can be found in the generation script
at `scripts/Hazel-ScriptGen.py` -- which translates integral types and Coral
types. Other typenames are treated as-is and directly copied into the C#
declaration.

### Prerequisites

The `clang` compiler from the LLVM project must be installed, and a matching
version of the Python [`clang` package](https://pypi.org/project/clang/)
alongside it.

On Windows under VC workflows LLVM can be installed either from
the [LLVM project releases page](https://github.com/llvm/llvm-project/releases)
under `LLVM-{version}-win64.exe` or via. the Visual Studio installer as the
"C++ Clang Compiler For Windows" workload.

### Workflow
1) New function that you want exposed to managed code is added to a source file 
in `Hazel/src/Hazel/Script/ScriptGlue`. The function name must begin with
`ScriptGlue_` and be in the `Hazel::InternalCalls` namespace  to be discovered.
```c++
namespace Hazel::InternalCalls {
    // Would be exposed in managed code as
    // `InternalCalls.ScriptGlue_MyNewThing_NewFunction`
    void ScriptGlue_MyNewThing_NewFunction(int someParameter) {
        ...
    }
}
```

2) Run `scripts/ScriptGen.py` from the Hazel root directory to generate the
C++ side interface. The generation script may take several minutes to generate
so this should only be performed once the final API changes for your commit
are prepared. During local dev cycles it is okay to simply insert your
declarations into the generated sources to avoid this time penalty.
```bash
python3 scripts/ScriptGen.py -t cxx -C Hazel/src/Hazel/Script/ScriptGlue -o Hazel/src/Hazel/Script/ScriptGenOutput.cpp -l Hazel-ScriptGen/Source/Template.cpp -s {{CLANG SYSTEM PATH}}
```
Where `{{CLANG SYSTEM PATH}}` is replaced with the path to standard include
files from your installation -- some example locations would be:

| Installation Type           | Path                                                                                            |
|-----------------------------|-------------------------------------------------------------------------------------------------|
| Visual Studio 2022          | `C:\Program Files\Microsoft Visual Studio\2022\Professional\VC\Tools\Llvm\lib\clang\19\include` |
| Sysroot Install             | `/usr/lib/clang/19/include`                                                                     |

If no path is provided the script will attempt to deduce the install location

3) Run `scripts/ScriptGen.py` from the Hazel root directory to generate the
C# side interface:
```bash
python3 scripts/ScriptGen.py -t cs -C Hazel/src/Hazel/Script/ScriptGlue -o Hazel-ScriptCore/Source/Hazel/Core/ScriptGenOutput.cs -l Hazel-ScriptGen/Source/Template.cs -s {{CLANG SYSTEM PATH}}
```

4) You should ideally expose the function in a nicer manner than the raw
interop call -- a structure such as this is typical for wrapping internal
behaviour from managed code:
```cs
namespace Hazel
{
    public static class MyNewThing
    {
        public static void NewFunction(int someParameter)
        {
            unsafe { return InternalCalls.ScriptGlue_MyNewThing_NewFunction(someParameter); }
        }
    }
}
```

5) When committing changes, ensure that you **do include** the generated
`Hazel/src/Hazel/Script/ScriptGenOutput.cpp` and
`Hazel-ScriptCore/Source/Hazel/Core/ScriptGenOutput.cs`.