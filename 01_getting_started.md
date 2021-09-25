# Getting Started
This page will serve as a general guide for setting up a project for using SadConsole and GoRogue.  It is not a step-by-step tutorial, per-se, and assumes you know how to do basic things like install NuGet packages.  Note that each library involved provides its own documentation, as linked in the [readme](README.md#documentation); and this set of write-ups is NOT intended to be a substitute therein.

# Prequisites
Before creating a project, it is recommended that you consider the following items:

## Install Appropriate .NET Version
SadConsole and GoRogue all support .NET Standard 2.1 or greater, .NET Core 3.1 or greater, or .NET 5 or greater; however the most recent .NET release is generally recommended for new projects.  At the time of writing, that is [.NET 5](https://dotnet.microsoft.com/download/dotnet/5.0) (SDK).

Note that .NET Framework is NOT supported.

## Decide on a Rendering Back-End
As outlined in the [readme](README.md#back_end_flexibility), SadConsole v9 offers support for multiple back-end renderers.  It is possible as a library user to implement support for a custom back-end; however doing so is beyond the scope of this document.  Currently, the SadConsole author provides back-ends for MonoGame and SFML:
- [SadConsole.Host.MonoGame](https://www.nuget.org/packages/SadConsole.Host.MonoGame/)
    - All other things equal, this is generally the recommended back-end for starting out.  Simply put, it has the most users, and comes with the easiest setup and fewest caveats cross-platform.
- [SadConsole.Host.SFML](https://www.nuget.org/packages/SadConsole.Host.SFML/)
    - Note that, at the time of writing, the current C# package for SFML distributed by those developers has some caveats regarding its usage in Linux, which are beyond the control of SadConsole.  If you intend to develop on Linux or distribute Linux packages, be prepared to test against and deal with these caveats to ensure your product will work as expected.

# Create Project
From here, you can create your project.  All packages involved use NuGet, so you can rely on NuGet to do appropriate dependency resolving; however, the path you follow here will differ slightly depending on whether or not you intend to use the GoRogue-SadConsole integration library.

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

## Project with Integration Library
In the future, the integration library will have an official NuGet release and `dotnet` template associated with it, so setting up a project will become very easy.  For now, however, please note that there are some manual processes involved.

In order to ensure dependencies are handled properly, it is highly recommended that you set up a local folder to act as a NuGet feed and place a manually compiled NuGet package in that feed, which allows you to use NuGet to install the integration library; this is the process that is outlined here.  Creating manual references is also technically possible, however if future changes break your dependency chain, you have been warned.

1. Set up a [Local NuGet Feed](https://docs.microsoft.com/en-us/nuget/hosting-packages/local-feeds).  The simplest way to accomplish this is to run: `nuget sources Add -Name "My Local NuGet Feed" -source <Path_To_Folder_Where_You_Will_Place_NuGet_Packages>`.
2. Clone the [integration library source code](https://github.com/thesadrogue/TheSadRogue.Integration).
3. Compile the source code in `Release` mode.  This will not only compile the source code, but also create a NuGet package (ie. a `.nupkg` file).  This file will be placed in the `<repository_root>/nuget` folder.
4. Copy the .nupkg files from above (EXCLUDE the .snupkg files) to the directory specified when you created your NuGet feed.
5. Create a new .NET project (Console is fine, the type can be switched later).
6. Install `TheSadRogue.Integration` from your local NuGet feed.  Note that some UIs allow you to select only specific NuGet feeds to show packages from when searching; ensure that you have your local feed selected in this case, or you will not be able to find the package.  This will install the integration library and core dependencies.  Also ensure that you select "include prereleases".
7. Install the appropriate host package for the rendering back-end you decided on: eg. `SadConsole.Host.MonoGame` or `SadConsole.Host.SFML`.
8. Copy-paste the sample SadConsole code included at the bottom of this section to test your project.

Note that, if in the future, an update is released to the integration library and you wish to compile the update and use it, it may be prudent to purge all NuGet caches before attempting the upgrade.  In general the version number for the NuGet package is not incremented when it is updated since the package does not yet have an official NuGet release, and NuGet is not designed to handle a different package with the same package ID and version number properly.

For reference, this will leave you with a project that has the following NuGet packages installed (some explicitly, the rest automatically because they are dependencies):
- `TheSadRogue.Primitives` - Library of shared primitive types between GoRogue and SadConsole.
- `SadConsole` - The core SadConsole package, with no back-end renderer implementation.
- `SadConsole.Host.[MonoGame | SFML]` - The package implementing the back-end renderer for the framework you decided on.
- `GoRogue` - The GoRogue package.
- `TheSadRogue.Integration` - Library containing APIs to integrate GoRogue's GameFramework with SadConsole.

### SadConsole Starter Code
```CS
using SadConsole;
using SadRogue.Primitives;

namespace <Insert_Namespace_Here>
{
    class Program
    {
        private static void Main(string[] args)
        {
            var SCREEN_WIDTH = 80;
            var SCREEN_HEIGHT = 25;

            SadConsole.Settings.WindowTitle = "SadConsole Game";
            SadConsole.Settings.UseDefaultExtendedFont = true;

            SadConsole.Game.Create(SCREEN_WIDTH, SCREEN_HEIGHT);
            SadConsole.Game.Instance.OnStart = Init;
            SadConsole.Game.Instance.Run();
            SadConsole.Game.Instance.Dispose();
        }

        private static void Init()
        {
            // This code uses the default console created for you at start
            var startingConsole = Game.Instance.StartingConsole;

            startingConsole.FillWithRandomGarbage(SadConsole.Game.Instance.StartingConsole.Font);
            startingConsole.Fill(new Rectangle(3, 3, 23, 3), Color.Violet, Color.Black, 0, Mirror.None);
            startingConsole.Print(4, 4, "Hello from SadConsole");

            // --------------------------------------------------------------
            // This code replaces the default starting console with your own.
            // If you use this code, delete the code above.
            // --------------------------------------------------------------
            /*
            var console = new Console(Game.Instance.ScreenCellsX, SadConsole.Game.Instance.ScreenCellsY);
            console.FillWithRandomGarbage(console.Font);
            console.Fill(new Rectangle(3, 3, 23, 3), Color.Violet, Color.Black, 0, 0);
            console.Print(4, 4, "Hello from SadConsole");

            Game.Instance.Screen = console;

            // This is needed because we replaced the initial screen object with our own.
            Game.Instance.DestroyDefaultStartingConsole();
            */
        }
    }
}
```