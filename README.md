# SadConsole-GoRogue Mini-Tutorial
This repository contains a series of markdown-based introductions to concepts for those looking to use SadConsole and GoRogue for roguelike game development or similar such endeavors.  It assumes some familiarity with GoRogue, and focuses mostly on SadConsole and how the two interact (and why you might want to use them together).

These docs will move elsewhere at some point in the future; this repository will be archived at that time.

# Why Pair GoRogue with SadConsole?
GoRogue aims to be a renderer independent library, and as such does not tie itself to any particlar library in that regard.  That much said, there are a number of reasons SadConsole can make a very good pairing with GoRogue:

## Shared Primitive Types
GoRogue v3 and SadConsole v9 are both major refactors of their predecessors.  One of the things that happened during these refactors, is many GoRogue v2 and SadConsole v8 types that pertained exclusively to 2d grids or visual representations of them were pulled out into a new library: [TheSadRogue.Primitives](https://github.com/thesadrogue/TheSadRogue.Primitives).  This DOES NOT introduce any sort of dependency of each library on the other; eg. GoRogue still functions without SadConsole, and vice versa.  However, it does make them a convenient pairing because you generally do not have to worry about type conversions; SadConsole's `Point` and GoRogue's `Point` are represented by the same data structure, and so on for all of the functionality moved to the primitives library.

For reference, functionality now in the primitives library includes all of the basic grid primitives from GoRogue v2 (Area, Point, Rectangle, Distance, AdjacencyRule, Radius, etc), the map view system from GoRogue v2 (now called Grid Views), and all the basic color representations from SadConsole (Color, Palette, Gradient).  It also includes some new functionality such as representations for polar coordinates, etc.

## Back-End Flexibility
SadConsole v9 is no longer based on MonoGame; it instead creates an abstraction independent of any particular back-end to suit its needs, and then provides numerous implementations of that abstraction for common frameworks such as MonoGame and SFML.  It also allows a user to implement their own back-end if they so desire.

This pairs well with GoRogue in that it provides some level of portability, yet does not forgo the possibility of interfacing directly with a back-end to do things outside the scope of console emulation (render non-grid based images, for example).

## Cooperative Community
GoRogue and SadConsole's communities interact frequently.  There are numerous developers on SadConsole's discord that are using GoRogue with SadConsole, and the developers of GoRogue and SadConsole respectively cooperate on various projects (including the shared primitives library and the integration library).

## First-Class Integration Support
The developers of GoRogue and SadConsole, along with tremendous support from some members of the communities, have created an "integration library" that provides a set of APIs that integrate GoRogue and SadConsole seamlessly: [TheSadRogue.Integration](https://github.com/thesadrogue/TheSadRogue.Integration).  It's similar in spirit to the one created for SadConsole v8 and GoRogue v2, though it is implemented much differently.

The integration library uses `GoRogue.GameFramework` as its base structure for dealing with maps, etc; so in general, the implementation as-is is subject to all of the same limitations inherent to `GameFramework`.  Even in cases where this is not viable for out-of-box use, however, it may be useful to fork and modify, or simply to use as a reference point for how to build your own structure.  The library implements things very efficiently and the code is generally well documented.

# Current State Snapshot
Since GoRogue v3 is not a finished product, there are a few things to be aware of.  The following summarizes the release status of each component of this toolchain at the time of writing:

| Library                 | State of Release  | Nuget? | API Stability |
| ----------------------- | ----------------- | ------ | ------------- |
| TheSadRogue.Primitives  | Full              | Yes    | Stable        |
| GoRogue                 | Beta              | Yes    | Stable; API is documented (though examples are lacking in some areas).  No major known bugs. |
| SadConsole              | Full              | Yes    | Stable        |
| SadConsole.Extended     | Full              | Yes    | Stable        |
| TheSadRogue.Integration | Beta              | Yes    | Mostly stable; no major known bugs, and no major refactors planned.  Serialization not implemented. |

# Documentation
Each element in this toolchain has its own documentation, which includes API documentation and in some cases articles which constitute tutorials on how to use various features.  Each library's documentation is linked and described (as it exists at the time of writing) below:
- [TheSadRogue.Primitives](https://thesadrogue.github.io/TheSadRogue.Primitives/) - Includes only API documentation.
- [GoRogue](http://www.roguelib.com) - Includes API documentation, upgrade guides, and some how-to's for various features.
- [SadConsole/SadConsole.Extended](https://www.sadconsole.com/v9) - Includes API documentation, getting started guides, and some how-to's/tutorials.
- [TheSadRogue.Integration](https://github.com/thesadrogue/TheSadRogue.Integration) - No web-hosted API documentation (though all classes are documented such that an IDE will retrieve documentation).  Each project in the GitHub repository contains its own README which details its architecture and basic functionality.  There is an `ExampleGame` project which details basic usage, and the basic `dotnet` template provides some well-documented code as well.  There is also a more involved example project [here](https://github.com/Chris3606/SadRogueExample).
