# Getting Started
 
## Building and Running
One of our goals is to make Hazel as easy as possible to build - if you're having any difficulties or weird errors, please let me know. We currently only support building on Windows 10 with Visual Studio 2019 or 2022 (2022 is default). You also need the following installed and in your path:

 - [Git](https://git-scm.com/downloads)
 - [Git-LFS](https://git-lfs.github.com/)
 - [Python 3](https://www.python.org/downloads/)

Assuming that's all installed and ready to go, you can proceed with the following:

1. Clone the repository: `git clone --recursive https://gitlab.com/ChernoProjects/Hazel-dev.git`
2. Run `Scripts/Setup.bat` - this will download required libraries and make sure everything is setup correctly
3. Open `Hazel.sln` and build either `Debug` or `Release` `x64` - Hazelnut should be the startup project so really you can just hit `F5` to build and debug the startup project. By default, this will load the _Sandbox_ project found in `Hazelnut/SandboxProject/Sandbox.hproj`
4. Open `Hazelnut/SandboxProject/Sandbox.sln` and build in either `Debug` or `Release`. This will build the C# module of the _Sandbox_ project which is necessary to properly play the various scenes in the project.

## Pulling the latest code
The `master` branch is required to always be stable, so there should never be any build errors or major faults. Once you've pulled the latest code, remember to run `Scripts/Setup.bat` **again** to make sure that any new files or configuration changes will be applied to your local environment.


## Next Steps
Probably an overview of Hazelnut and the basic architecture.
