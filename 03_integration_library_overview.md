# Integration Library Overview
This page is designed to give you a very broad overview of the GoRogue-SadConsole integration library architecture, so that you understand general concepts core to the library.

# General Design Decisions
The following outlines some core design choices of the library and their implications.  Understanding these decisions is very useful in understanding the purpose and limitations of the library.

## Based on GameFramework
Recall that GoRogue provides [two main categories](http://www.roguelib.com/articles/intro.html#library-design-concepts) of features; "core" features, and "game framework" features.  The differences between the two are notable, and can be reviewed at the linked page.  Note that the integration library is based on the GoRogue's "game framework" features.

This makes the most sense for general use, as it avoids pointlessly replicating structures for maps and such.  However, game framework by definition is less generic than the rest of the library, and although it fits many use cases, there are times where it may not fit a particular use case.  Some notable limitations of game framework include (but are not necessarily limited to):

- No out-of-box multi-level map support
- No infinite map/chunkloading support

There may be cases, therefore, when it is not desireable to use the integration library.  In these cases, you may not be able to use the integration library as-is.  However, it is open-source, meaning that in these cases you can still fork and modify it for your needs, or just use its architecture as a model/reference point for building your own from scratch.  Given that some elements of integration are non-trivial to write generically, this can still be of significant value.

## Rendering Strategy
Recall from the [SadConsole overview](02_sadconsole_overview.md#Differences-between-surfaces-and-entities) that there are some important distinctions between SadConsole entities, and a cell surface being rendered via a screen surface.  Notably, as a result of those distinctions, it would be too performance intensive to have every piece of terrain on a map be visually represented by a SadConsole `Entity`.

Also recall that GoRogue's "game framework" defines map objects in terms of a single interface called `IGameObject`, which has a position and is used to represent both terrain and non-terrain objects.  There are some special restrictions put on game objects that reside in the terrain layer; namely that while they are added to a `Map` they can't be moved (attempting to do so will produce an error).  This allows the game framework to employ certain optimizations to efficiently deal with terrain, including storing terrain in a grid view instead of a spatial map.  This distinction is useful to know, because it matches up fairly well with the distinction between SadConsole's cell surfaces and its entities.

It is obviously beneficial to have non-terrain map objects be represented by SadConsole `Entity`.  Since rendering terrain using `Entity` instances is not viable, however, a different approach is required here.  Broadly speaking, there are two basic ways to go about rendering game framework terrain in SadConsole:

- Have each piece of terrain record its appearance.  Copy the appearances for terrain on the map to the `ColoredGlyph` within a cell surface, and ensure to update that surface when any terrain appearance changes.
    - This is generally the simplest approach to implement, and it is viable for a fair number of use cases.  However, it is not the most efficient method, and can be error-prone if written generically.  Therefore, the integration library takes a different approach.
- Have each piece of terrain record its appearance.  Create a custom `ICellSurface` which pulls its `ColoredGlyphs` by accessing the terrain layer of the `Map` and retrieving its appearance field when a `ColoredGlyph` is requested, instead of retrieving them from an array of `ColoredGlyphs`.
    - This is the core approach the integration library takes.  It is not as simple to implement as the first approach, but the burden of the additional work falls to the library writers to implement.  It also enables more straightforward integration of other corresponding systems of the two libraries, such as their component systesms.  Ultimately, from the perspective of a user, this method results in a more efficient, less error prone, and easier to use product.

Understanding that the integration library takes the second approach will help you to better comprehend its core architecture.

## Component Integration
Recall that SadConsole's `IScreenObject` interface provides a facility for attaching and detaching components to objects.  These components may handle input and participate in the `Update`/`Render` loops.  Also note that GoRogue's `IGameObject` provides a separate facility for attaching components to implementing objects, and GoRogue's `Map` also provides a similar facility.

In all of these cases, each library provides acess to its components via a separate field on the object the components are being attached to; for example, components attached to a SadConsole `IScreenObject` are accessible via the `IScreenObject.SadComponents` field, and GoRogue components are similarly accessible via a field called `GoRogueComponents`.

Considering that each of these component systems serve different purposes and provide differing functionality, this could obviously cause some confusion when the libraries are integrated.  If an object implements `IGameObject` and also inherits from `Entity`, for example, it would have both a SadConsole component system (from `Entity`, because it derives from `ScreenObject`), and a GoRogue component system.

The integration library provides a solution to this by integrating the component systems into one in cases where they are both present.  Generally, you will find an `AllComponents` field on objects which have both SadConsole and GoRogue component functionality.  Any component added to this structure will automatically be added to both SadConsole's and GoRogue's component systems as needed.  For example, an object added to this field could simultaneously implement SadConsole's `IComponent` and use that interface to hook into the `Update` loop, and also implement GoRogue's `IParentAwareComponent` and benefit from that functionality.  The integration library also provides convenient base classes for implementing components as such, which will be discussed later.

# Basic Object Hierarchy
The basic object hierarchy for the core SadConsole structures is described [here](https://github.com/thesadrogue/TheSadRogue.Integration/tree/main/TheSadRogue.Integration#thesadrogueintegration).  Understanding the functionality provided by these structures is crucial to understanding how to properly utilize the library.

## Basic Use Cases
The following is a quick-reference describing some basic use cases and the classes to use.  This is NOT intended to cover every possible use case for the library, nor to be a complete reference for each of the classes mentioned.  It is simply designed to be a quick-start guide for determining where you may want to look in order to implement a certain type of functionality

| Use Case                                             | Class/Config                 |
| ---------------------------------------------------- | ---------------------------- |
| Terrain-Layer Objects                                | `RogueLikeCell`              |
| Non-Terrain Layer Objects (player, monsters, etc)    | `RogueLikeEntity`            |
| Map Rendered at a Single Point on Screen             | `RogueLikeMap` (specify DefaultRenderer parameters to constructor) |
| Map Rendered at Different Points on Screen           | `RogueLikeMap` (null default renderer parameters to constructor); add BOTH the Map AND any renderers to the SadConsole screen hierarchy |
| Creating Player Controls                             | `PlayerKeybindingsComponent` |
| Controlling Visibility Based on FOV                  | `FieldOfView` namespace      |
| Creating Components to Attach to Non-Terrain Objects | `RogueLikeComponentBase`/`RogueLikeComponentBase<T>` |
| Creating Components to Attach to Terrain Object      | Any class (do NOT use `IComponent` from SadConsole); its functionality is not supported in terrain components |
| Creating Components to Attach to Maps                | Can inherit from SadConsole's `IComponent` and/or use any GoRogue component functionality |

# Example Code
In addition to this document, the forms of documentation currently available for the integration library are listed in the [readme](README.md#documentation).  In particular, you are encouranged to look through the [ExampleGame](https://github.com/thesadrogue/TheSadRogue.Integration/tree/main/ExampleGame) project, as it demonstrates the basic features of the library.