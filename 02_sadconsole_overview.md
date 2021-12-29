# SadConsole Overview
SadConsole is a large library with many features.  Further, SadConsole provides its own documentation as described in the [readme](README.md#documentation).  It is highly recommended that you give this documentation a read if you're just learning SadConsole.  This page is not official SadConsole documentation, and is designed only to give you a very broad overview of only the portions of the SadConsole architecture directly related to GoRogue integration.  Its goal is nothing more than to enable you understand general SadConsole concepts core to integrating with GoRogue and the integration library.  There are many useful concepts and features of SadConsole that aren't discussed here.

# Concepts to Be Aware Of
A few concepts that will help you understand SadConsole architecture:

## Interfaces with Corresponding Classes
Most of SadConsole's rendering-specific primitive data types are provided as interfaces.  For each interface, there is a class provided that implements the interface in a default way.  For example, the interface `IScreenObject` is implemented by the class `ScreenObject`, and similarly `ICellSurface` is implemented by `CellSurface`.  I will henceforth call the class in these situations a "corresponding class" of an interface, for brevity.

Generally, you will NOT want to implement SadConsole interfaces yourself; the class implementing the interface for you is usually sufficient.  The interfaces are provided so that in complex or intricate scenarios, they _could_ be implemented in a custom way by a user; but generally this is not necessary.  In short, prefer using or inheriting from the corresponding class before implementing the interfaces directly. 

## Partial Classes
The source code for some of SadConsole's classes is quite large; and as a result the author elected to use [partial classes](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/partial-classes-and-methods) to split the implementation of these large classes up over multiple files.  Keep this in mind; if you're looking for the implementation of a SadConsole method and not finding it, it may be in another file that is also part of the class's implementation.  In these situations, the .cs files will have related names; ie. `ScreenSurface.cs` and `ScreenSurface.Input.cs`.

# Basic Object Heirarchy
SadConsole presents a core inheritance heirarchy of interfaces/objects that represent rendering concepts.  Understanding the differences between them will help you understand how to properly utilize SadConsole.  This does NOT provide an overview of every class and interface in the library; just the ones core to its rendering structure.

Many of the interfaces listed here have "corresponding classes" as explained above; in these cases, I will note it as `IMyInterface` | `TheCorrespondingClass`.

- `ColoredGlyph` - A foreground, background, and glyph, along with any associated decorators.  Represents the rendering properties of a single grid position on a surface/console.  Note that the class itself does NOT record a position.
- `ICellSurface` | `CellSurface` - A fancied up array of `ColoredGlyph` objects used to represent the visual aspect of a 2D surface.  To that end, it includes all sorts of editing methods and state.  It does not do any actual rendering, however it does have a viewport which renderers are expected to respect.  As a result, if you need to have two different viewports of the same cells, this is implemented by sharing the same `ColoredGlyph` objects between two different cell surfaces.
- `IScreenObject` | `ScreenObject` - A generic object processed by SadConsole. Provides concepts representing absolute and relative positions, parent/child relationships, the ability to attach components, as well as a framework for input processing and interfacing with the Update loop.  Note that the basic `ScreenObject` class does not render anything on its own, although it provides methods that can be overriden to do so.  Also note that its children may render things even though the base object does not.
    - `IScreenSurface` | `ScreenSurface` - A SadConsole object (which implements `IScreenObject`) that renders an `ICellSurface` to the screen, given a font, etc.  Also provides some relevant mouse input APIs (eg. information about when mouse enters, exits the surface, clicks in it, etc.).
        - `AnimatedScreenSurface` - A screen surface that takes a collection of frames (appearances of a set of cells) and animates them.
        - `Console` - A screen surface, with the addition of a positionable (and optionally visible) cursor that can be used to write text.
    - `Entity` - A positionable, single-cell screen object that can be associated with a screen surface.  Useful for small, independent things that move around (player, monsters, items, etc).  Note that it DOES NOT render itself; for rendering, you must associate an `Entity` with an `Entities.Renderer`.  A `Renderer` is attached as a component to a `ScreenSurface`; and the `Renderer` renders all entities added to it using the screen surface's font, if and only if they are within the viewport of the cell surface being rendered.

# Rendering Architecture
It is important to remember that although SadConsole is designed to emulate the style/look/feel of old-school consoles, it does not functionally render as traditional consoles do.  It intentionally deviates from the traditional path so that it can provide features outside of what a normal console could.  These features include efficiency improvements in rendering itself, fonts with more than 256 characters, multiple fonts on screen at once, support for integrating graphical elements which are not strictly grid based, and more.  These deviations are important to understand, however, as they tend to permeate the design decisions you will find throughout the library.

## Textures and Caching
In order to efficiently render consoles/surfaces, SadConsole effectively takes the surface's list of `ColoredGlyphs` and renders it onto a single texture (when and only when the appearance of some cell changes).  The visible portion of this texture can then be rendered to the screen each time through the render loop, in a single draw call.  `Entities.Renderer` works in a similar way; it renders all of the (visible) entities that have been added to it down to a single texture (when and only when one of them changes), and that texture can then be drawn to the screen each time through the render loop.

This is important to understand, as it has a number of implications/effects:
- Objects generally have an `IsDirty` flag, which is used by the rendering systems to know when items need to be re-drawn to a texture.  Generally, SadConsole functions will set this flag as appropriate for you, however there are cases where you may need to know it exists.  Namely, if it isn't set properly, visual changes will not appear because the change will not actually be drawn to the backing texture.
- Visual changes such as updating the appearance of an object should happen in the `Update()` function, NOT in `Render()`.  Functions named `Render()` generally are intended for drawing the existing visual states to a texture, and as such changes to the data from which the texture is derived (eg. ColoredGlyphs, themes, etc) should happen before this point.

# Differences Between Surfaces and Entities
As stated above, surfaces render all of their `ColoredGlyphs` down into a single texture.  The `Entities.Renderer` similarly takes all of the entities added to it that are within a cell surface's viewport, and renders them down into a single texture.  Nevertheless, there are some key differences between `ColoredGlyphs` in a surface and `Entities` that affect how they should be used.  Being familiar with these differences is extremely useful, as they are key to understanding when to use entities and when not to.

The core difference, is that a _screen surface_ (eg. an object that renders an array of `ColoredGlyphs`) is a `ScreenObject`; the individual `ColoredGlyphs` it is rendering are _not_ screen objects.  In contrast, an `Entity` itself is a `ScreenObject`.

This means that an individual Entity can handle input, handle `Update`, have its own position, etc.  An individual `ColoredGlyph` in a screen surface can do none of these things.

## Takeaways TLDR;
The differences will be expanded upon below.  In short, however, the takeaway should be, DO NOT default to using `Entity` for everything.  The actual rendering of an entity is relatively cheap performance-wise; but many of the other associated operations (such as tracking which entities are within viewports) are not.

For example, given a typical roguelike map, each individual terrain tile should NOT be an entity; adding the overhead of a `ScreenObject` for each tile is performance intensive.  Instead, the terrain's appearance should be recorded in a surface.  By contrast, it would be OK to have your player, monsters, items, etc. be `Entities`; and probably preferred, because it allows you to do useful things like have them handle `Update` or input events, have components attached/detached, and be easy to reposition.

## Entities are Positionable
`ColoredGlyphs` themselves do not have a position associated with them.  Neverless, a cell surface is more or less an array of ColoredGlyphs, where each glyph represents the appearance of a single position.  **In this case, the mapping of ColoredGlyph instances to positions is based on the index in the cell surface/array at which it is located.**

Entities, on the other hand, do have a position associated with them which has nothing to do with their position in any list or array.  This makes certain operations more expensive, like finding all entities within a viewport.  For a surface, that is easy because you simply iterate over each position and pull the `ColoredGlyph` out of the corresponding index.  For a group of entities, you must iterate over all the entities and check their `Position` field.

As a result of this, note that rendering/managing a number of entities `n` is less efficient than rendering/managing `n` `ColoredGlyphs` in a cell surface. 

## Entities Can Handle Update and Can Have Components
An individual `ColoredGlyph` itself has no facility to perform actions in the update loop; there is no overridable `Update` method and no components can be attached to them.  Similar logic applies to handling keyboard or mouse input.  However, because `Entity` is a `ScreenObject`, it _does_ have an `Update` method, and as such can perform actions in the `Update` loop, as well as handle input via the typical `ProcessKeyboard` or `ProcessMouse` functions.  This of course entails some overhead, because the entity renderer (as an entity's "owning" object) must ensure `Update` is called on each `Entity` when applicable (although this behavior is optional).

Note that an entity renderer _only_ forwards `Update()` calls through to its entities; it is _not_ responsible for forwarding `ProcessKeyboard` and `ProcessMouse` calls through.  These functions respect the traditional "focus" system of the SadConsole input framework.  In short, if you wish an entity to process input, you must either ensure it is the focused `ScreenObject`, or manually ensure that the focused `ScreenObject` will forward calls through to the entity.

