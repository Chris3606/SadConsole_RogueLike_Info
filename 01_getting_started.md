# Getting Started
This page will serve as a general guide for setting up a project for using SadConsole and GoRogue.  It is not a step-by-step tutorial, per-se, and tends to assume (though does not require) a basic knowledge of .NET development, including tasks such as installing Nuget packages and similar.  There are a multitude of resources available on the internet for these topics, including official documentation from Microsoft, if you encounter something you're unfamiliar with.  Note that each library involved also provides its own documentation, as linked in the [readme](README.md#documentation); and this set of write-ups is NOT intended to be a substitute therein.

# Prequisites
Before creating a project, it is recommended that you consider the following items:

## Install Appropriate .NET Version
SadConsole and GoRogue all support .NET Standard 2.1 or greater, .NET Core 3.1 or greater, or .NET 5 or greater; however the most recent .NET release is generally recommended for new projects.  At the time of writing, that is [.NET 6](https://dotnet.microsoft.com/download/dotnet/6.0) (SDK).

Note that .NET Framework is NOT supported.

## Decide on a Rendering Back-End
As outlined in the [readme](README.md#back-end-flexibility), SadConsole v9 offers support for multiple back-end renderers.  It is possible as a library user to implement support for a custom back-end; however doing so is beyond the scope of this article.  Currently, the SadConsole author provides back-ends for MonoGame and SFML:
- [SadConsole.Host.MonoGame](https://www.nuget.org/packages/SadConsole.Host.MonoGame/)
    - All other things equal, this is generally the recommended back-end for starting out.  Simply put, it has the most users, and comes with the easiest setup and fewest caveats cross-platform.
- [SadConsole.Host.SFML](https://www.nuget.org/packages/SadConsole.Host.SFML/)
    - Note that, at the time of writing, the current C# package for SFML distributed by those developers has some caveats regarding its usage in Linux, which are beyond the control of SadConsole.  If you intend to develop on Linux or distribute Linux packages, be prepared to test against and deal with these caveats to ensure your product will work as expected.

## Decide Whether to Use the Integration Library
As outlined in the [readme](README.md#first-class-integration-support), an "integration" library has been created by the developers of GoRogue and SadConsole, as well as a few people from their communities.  This library is designed to provide a set of APIs that integrate GoRogue and SadConsole seamlessly, and thereby ease the process of using the two libraries together.  It is useful in many cases; but may not fit all projects out of the box.  Details on when may or may not want to use it can be found at the linked readme, as well as [here](03_integration_library_overview.md#based-on-gameframework).

If you're unsure whether to use it, my recommendation is to start by using it, and then modify and/or pivot away from it as needed if you find a case where it isn't suiting your needs.  The integration library does aim to support a wide variety of use cases, and regardless some of the boilerplate needed to integrate SadConsole and GoRogue is non-trivial to write correctly.  Therefore, using the integration library initially will save you needing deal with that process immediately.  Furthermore, it may serve as an example for you if you do end up needing to deviate.

In either case, the project creation instructions will differ a little depending on whether you decide to use the integration library or not, as detailed below.

# Create Project
From here, you can create your project.  All packages involved use NuGet, so you can rely on NuGet to do appropriate dependency resolving; however, the path you follow here will differ slightly depending on whether or not you intend to use the GoRogue-SadConsole integration library.

## Project With Integration Library
The integration library is currently available on NuGet, and the authors have also provided an additional package which provides templates for the `dotnet new` command that make creating a new project fast and easy.

1. From your CLI, run `dotnet new --install TheSadRogue.Integration.Templates`.  This will install two `dotnet` templates on your system:
    - `gorogue-sadconsole-mg`: A template for an integration library project using the MonoGame back-end for SadConsole
    - `gorogue-sadconsole-sfml`: A template for an integration library project using the SFML back-end for SadConsole
2. Run `dotnet new gorogue-sadconsole-[mg | sfml] -o <NameOfGame>`.  Specify `mg` or `sfml` as appropriate for the SadConsole back-end you want to use (see above), and replace `<NameOfGame>` with your desired name.
    - Note: Visual Studio 2022, as well as Rider, have GUI support for `dotnet new` templates.  As such, once you have run the `dotnet new --install` command from step 1, you should also be able to select these templates from the "New Project" dialog in those IDEs if you prefer.

A new subdirectory will be created in the current directory of your CLI called `<NameOfGame>`.  This directory will contain a `.csproj` for your project, as well as some documented starter code that demonstrates some basic integration library features.

If you are using Visual Studio or some similar IDE, also note that, as is typically the case with projects created via the `dotnet new` command, the template has a `.csproj` (project) but _not_ a `.sln` (solution) file.  Visual Studio will create a default solution file for you the first time you open the `.csproj`, and if you select `File -> Save All` as soon as the project opens, it will save the created `.sln` file to disk.  You can also create a `.sln` manually via the appropriate `dotnet new` command if you wish.  Either way, you will need a .sln file in order for Visual Studio to properly handle build configurations (Debug vs Release).

For reference, the created project will have the following NuGet packages installed (some explicitly, the rest automatically because they are dependencies):
- `TheSadRogue.Primitives` - Library of shared primitive types between GoRogue and SadConsole.
- `Newtonsoft.JSON` - JSON library used by SadConsole for serialization.
- `SadConsole` - The core SadConsole package, with no back-end renderer implementation.
- `SadConsole.Host.[MonoGame | SFML]` - The package implementing the back-end renderer for the framework you decided on.
- `Troschuetz.Random` - RNG library on which GoRogue relies.
- `OptimizedPriorityQueue` - Priority queue library on which GoRogue relies.
- `GoRogue` - The GoRogue package.
- `TheSadRogue.Integration` - Library containing APIs to integrate GoRogue's GameFramework with SadConsole.

## Project Without Integration Library
1. Follow the directions [here](https://sadconsole.com/v9/articles/getting-started-cli.html); this will get you a project with SadConsole installed and a small bit of code to start out.
2. Install the [GoRogue NuGet package](https://www.nuget.org/packages/GoRogue/3.0.0-alpha04).

Note that GoRogue v3 is currently in alpha, so you _must_ "include prereleases" when viewing packages to see it.  Additionally, each version of the package will have a version with "-debug" appended onto the end.  This version is a version compiled in Debug mode, and can be useful for debugging issues when you need to step into GoRogue functions themslves.  However, it is highly recommended that you use the regular release version unless you need to debug.  Note that the "-debug" version will show up as the most "recent" GoRogue release due to the way NuGet resolves package versions.  This will be fixed when GoRogue v3 receives a full (non-alpha) release.

For reference, this will leave you with a project that has the following NuGet packages installed (some explicitly, the rest automatically because they are dependencies):
- `TheSadRogue.Primitives` - Library of shared primitive types between GoRogue and SadConsole.
- `Newtonsoft.JSON` - JSON library used by SadConsole for serialization.
- `SadConsole` - The core SadConsole package, with no back-end renderer implementation.
- `SadConsole.Host.[MonoGame | SFML]` - The package implementing the back-end renderer for the framework you decided on.
- `Troschuetz.Random` - RNG library on which GoRogue relies.
- `OptimizedPriorityQueue` - Priority queue library on which GoRogue relies.
- `GoRogue` - The GoRogue package.
