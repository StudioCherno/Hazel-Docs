<div class="title"> 
    <img src="/res/HazelGradientLogo-Square.png" alt="Hazel Logo" />
    <h1> Environment and Building </h1>
</div>

> While Linux is in an experimental state, it is recommended that users build from the `dev` branch of the Hazel repo rather than a release branch; as patches are continuously applied in between releases which may not propogate to the release branch.

Hazel supports both native in-tree builds and Docker builds. The latter is the recommended format for end users not
making active changes.

**Contents**
 - [Docker Builds](#docker-builds)
 - [Native Builds](#native-builds)

## Docker Builds

**1. Setup & Prerequisites**

You will need the following tools to get started:

> It's recommended that you get these from your system maintainers (e.g. via. a package manager or from user repositories) as they're more likely to be properly configured for your system out of the box.

 - [Git](https://git-scm.com/downloads)
 - [Docker](https://www.docker.com/)

Docker requires a Docker daemon to be running in order to perform builds and create containers. Information for starting and controlling this daemon can be found [here](https://docs.docker.com/config/daemon/start/).

Other dependencies will be fetched inside the container during the build process and copied out.

Finally, you should clone Hazel using `git clone --recursive -b dev https://github.com/StudioCherno/Hazel`. Simply downloading the branch archive from Github or using non-recursive clones will not work -- Hazel handles many dependencies as submodules.

**2. Building**

The Docker build has been encapsulated in `scripts/Docker-Build.sh` which invokes docker with the appropriate flags and copies out the resultant binaries. At present, this process only produces a debug Hazelnut binary at present (alongside any binaries required from dependencies).

The script *does **NOT*** support out of tree builds at present, and attempting to execute it outside of the Hazel root directory will likely create broken orphaned containers. 

The script accepts passthrough flags to docker itself; these expose the following variables which can be set via. `--build-arg VAR=value`:
| Name       | Description                           | Default |
| ---        | ---                                   | ---     |
| `NPROC`    | Sets the build parallelism for `make` | `1`     |
| `CXXFLAGS` | Sets additional C++ compile flags     | `-`     |

The recommended invocation is:
```
scripts/Docker-Build.sh --build-arg NPROC=$(nproc) --build-arg CXXFLAGS=-Wno-everything
```
In order to make use of maximum parallelism and suppress all warnings.

> This process is known to take a long time on initial builds. Having adequate parallelism set in the `NPROC` variable mitigates this factor but a fresh clone will still likely take several minutes.

After the script is done -- the Hazelnut binary and required dependencies will have been copied out to `bin`. The Docker build also builds the Sandbox project for convenience, although the ScriptCore `.dll` is copied out to its appropriate location, so C# project builds can be performed outside of Docker as-usual.

**3. Running**

Before running Hazelnut from the Docker build -- you must set the following variables from the Hazel root directory:
```
export HAZEL_DIR=$(realpath .)
export VULKAN_SDK=$(realpath bin/x86_64)
export LD_LIBRARY_PATH=$(realpath bin/)
```
Then, to run Hazelnut, use:
```
cd Hazelnut
../bin/Hazelnut [project]
```
If no project is specified in `argv`, the Sandbox project will be opened.

## Native Builds

> This section assumes some familiarity with native toolchains and interaction with GNU Make.

**1. Setup & Prerequisites**

You will need the following tools to get started:

> It's recommended that you get these from your system maintainers (e.g. via. a package manager or from user repositories) as they're more likely to be properly configured for your system out of the box. Toolchains and associated libs/tooling are often provided as a "development" metapackage.

 - [Git](https://git-scm.com/downloads)
 - [Premake 5](https://github.com/premake/premake-core)
 - [GNU Make](https://www.gnu.org/software/make/)

Supported/Tested build toolchain elements are listed below. Please note that this list is not exhaustive, as many aspects of build tooling are entailed as prerequisites to other tools. Additionally,
whilst only `glibc` is tested at present, we are willing to maintain `musl` support if issues are submitted; otherwise, alternate C/C++ compilers/standard libraries other than those listed below
will be unlikely to recieve official support.
| Tool                 | Supported                                                                                                                                                   | Recommended        |
| ---                  | ---                                                                                                                                                         | ---                |
| C/C++ Compiler       | [GCC](https://www.gnu.org/software/gcc/)<br>[Clang](https://github.com/llvm/llvm-project)                                                                   | Clang              |
| `ar` Archive Tool    | [GNU Binutils `ar`](https://www.gnu.org/software/binutils/)<br>[LLVM `llvm-ar`](https://github.com/llvm/llvm-project)                                       | LLVM `llvm-ar`     |
| Linker               | [GNU Binutils `ld`](https://www.gnu.org/software/binutils/)<br>[LLVM `lld`](https://github.com/llvm/llvm-project)<br>[Mold](https://github.com/rui314/mold) | Mold               |
| C++ Standard Library | [GNU `libstdc++`](https://www.gnu.org/software/gcc/)<br>[LLVM `libc++`](https://github.com/llvm/llvm-project)                                               | `-`                |

The following dependencies will also need to be installed in order to use Hazel:
 - The Wayland/X11 packages listed [here](https://www.glfw.org/docs/latest/compile_guide.html#compile_deps_wayland)
 - [`gtk3`](https://www.gtk.org/)
 - [`zlib`](https://www.zlib.net/)
 - [`libdw`](https://sourceware.org/elfutils/)
 - [`libunwind`](https://github.com/libunwind/libunwind)
 - [Intel `tbb`](https://www.intel.com/content/www/us/en/developer/tools/oneapi/onetbb.html) (Recommend getting "standalone" if downloading from Intel directly)
 - [`dotnet`](https://dotnet.microsoft.com/en-us/)

You should then clone Hazel using `git clone --recursive -b dev https://github.com/StudioCherno/Hazel`. Simply downloading the branch archive from Github or using non-recursive clones will not work -- Hazel handles many dependencies as submodules.

When inside the Hazel root directory -- download the [VulkanSDK](https://vulkan.lunarg.com/sdk/home#linux) and unpack/rename it such that the SDK root is as follows: `Hazel/vendor/VulkanSDK/x86_64`.

**2. Building**

The script `build/Linux-Build.sh` performs appropriate configure and build steps for native Hazel code, managed Coral/ScriptCore code and the Sandbox project. The build config can be modified by
setting the `BUILD_CONFIG` variable to `Debug` or `Release`. The `argv` is passed through to the `make` invocation for Hazel native code -- this should be used to set build parallelism and
toolchain opts. in accordance with regular Makefile practice (`CXX`, `AR` and `LDLIBS` etc.).

The resultant outputs are placed into `bin/` in a subdirectory named based on the build config, platform and architecture.

**3. Running**

Before running Hazelnut -- you must set the following variables from the Hazel root directory:
```
export HAZEL_DIR=$(realpath .)
export VULKAN_SDK=$(realpath Hazel/vendor/VulkanSDK/x86_64)
```

Then, `cd` into the `Hazelnut/` directory and run `../bin/$BUILD_CONFIG-linux-x86_64/Hazelnut/Hazelnut`.
