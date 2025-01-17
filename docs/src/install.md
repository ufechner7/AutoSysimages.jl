# Installation and usage

You can easily install the package using the standard packaging system:
``` julia
] add AutoSysimages
```

The easiest way to use this package is to insert a small script somewhere into your path depending on your operation system.

## Script for Linux

On Linux, you can use the following bash script that is provided in `scripts/linux/asysimg`. The recommended location is `~/.local/bin/asysimg`.

``` bash
#!/usr/bin/env bash
JULIA_EXE=julia     # or [INSERT-YOUR-PATH]/julia
asysimg_args=`$JULIA_EXE -e "using AutoSysimages; print(julia_args()); exit();" "$@"`
$JULIA_EXE $asysimg_args "$@"
```

## Script for Windows

On Windows, you can use the following batch script that is provided in `scripts/windows/asysimg.bat`. It's recomended add Julia to `PATH` during installation, and but `asysimg.bat` into the binary file (e.g., `"C:\\Users\\xxx\\AppData\\Local\\Programs\\Julia-1.X.X\\bin"`).

``` bash
@echo off
for /f "tokens=1-4" %%i in ('julia.exe -e "using AutoSysimages; print(julia_args()); exit()"') do set A=%%i %%j %%k %%l 
@"%~dp0\julia.exe" %A% %*
```

## Basic usage

Once you install the package and save the script, you can easily run Julia from terminal, using one of the following options with any additional arguments, as normal. It automatically loads the latest system image for your project and start snooping for new precompile statements.
``` bash
asysimg
```
``` bash
asysimg --project
```
``` bash
asysimg --project=examples/ExampleWithPlots
```

## How it works?

In the first step, `asysimg` script detect if there exists a system image for the current project by calling `julia_args()` and print argument for Julia to load such image. Example of the produced arguments.

The first argument (`-J`) loads the latest system image and is present only if such an image is found. 
``` bash
-J /home/user/.julia/asysimg/1.X.X/4KnVCS/asysimg-2022-09-01T14:54:50.395.so
```
where `4KnVCS` is a hash of the project path.

The second arguments (`-L []/src/start.jl`) initializes this package and start snooping for new precompiles statements.
``` bash
-L [AutoSysimages-DIR]/src/start.jl
```

Once the snooping is started it records precompile statements into a temporary files. When the Julia session is terminate, these statements are copied to project-specific file, like
```
/home/user/.julia/asysimg/1.X.X/4KnVCS/snoop-file.jl
```

## Select packages

You can select packages to be included into the project-specific system images. There is an interactive selection process that can be triggered by calling
``` julia
using AutoSysimages
select_packages()
```
That shows the manual selection (`asysimg --project=examples/ExampleWithPlots/`)
``` julia
asysimg> select_packages()
[ Info: Please select packages to be included into sysimage:
[press: Enter=toggle, a=all, n=none, d=done, q=abort]
 > [ ] LinearAlgebra
   [ ] OhMyREPL
   [X] Plots
   [ ] Printf
```

The settings are stored in `SysimagePreferences.toml`file just next to the current `Project.toml` file. You can modify the settings manually in the file.
```
[AutoSysimages]
include = ["Plots"]
exclude = []
```

To check which versions of the packages will be included, you can run
``` julia
asysimg> status()
    Project `/home/petr/repos/AutoSysimages/examples/ExampleWithPlots/Project.toml`
   Settings `/home/petr/repos/AutoSysimages/examples/ExampleWithPlots/SysimagePreferences.toml`
Packages to be included into sysimage:
  [91a5bcdd] Plots v1.31.7
```



## (Re)build sysimage

You can rebuild your system image at any time, but the changes will be reflected only after you restart Julia (using `asysimg`). It automatically includes all the selected packages and recorded precompile statements.

```
using AutoSysimages
build_sysimage()
```

The sysimage will be generated by [PackageCompiler](https://github.com/JuliaLang/PackageCompiler.jl), or by very experimental [chained builds](@ref chained_build).
