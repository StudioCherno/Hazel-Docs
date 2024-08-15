<div class="title"> 
    <img src="/res/Hazel-IconLogo-2023.png" alt="Hazel Logo" />
    <h1> Workflow Changes </h1>
</div>

**Contents**
 - [Building Projects](#building-projects)

## Building Projects

Hazelnut's built-in functionality for configuring and building projects does not currently work on Linux. Instead, users
must externally use premake to configure their C# project, and `dotnet` to build it. The template `premake5.lua` files
should still work appropriately.

When configuring a project, you must ensure `HAZEL_DIR` is set appropriately to the absolute path of the Hazel root
directory. `premake` should be set to the `vs2022` target in order to generate `dotnet`-compatible slns. The `gmake`
backends will not work with .NET Core projects.

To build -- use `dotnet build MyProj.sln` with any appropriate additional parameters.
