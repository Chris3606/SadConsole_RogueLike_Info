# Integration Library Overview
This page is designed to give you a very broad overview of the GoRogue-SadConsole integration library architecture, so that you understand general concepts core to the library.

# Design Goals
The integration library, in general, is designed with the following set of goals in mind:
1. Provide a performant, safe, and easy interface for a user to use.
2. Focus code primarily on two areas:
    - Integrating areas where GoRogue and SadConsole provide structures for the same concepts (for example, the component systems, highlighted in below sections).
    - Writing helpful features that can only be written with GoRogue-SadConsole integration as a given (for example, FOV visibility handling).
3. Leave SadConsole features to SadConsole and GoRogue features to GoRogue wherever possible.
    - This is to say, the library does _not_ try to replace/conceal features already in SadConsole or GoRogue; it instead simply tries to minimize points of friction between them.  If you're familiar with SadConsole, all of your SadConsole knowledge should translate to using the integration library; and similarly GoRogue-specific knowledge should work as well.  If the integration library itself doesn't provide a method of doing something, so long as you have an understanding of how the integration library features merge features from the two libraries, you should be able to successfully default to use your standalone knowledge of GoRogue or SadConsole to accomplish your objective.

# General Design Decisions
The following outlines some core design choices of the library and their implications.  Understanding these decisions is very useful in understanding the purpose and limitations of the library.

## Based on GameFramework
Recall that GoRogue provides [two main categories](http://www.roguelib.com/articles/intro.html#library-design-concepts) of features; "core" features, and "game framework" features.  The differences between the two are notable, and can be reviewed at the linked page.  Note that the integration library is based on the GoRogue's "game framework" features.

This makes the most sense for general use, as it avoids pointlessly replicating structures for maps and such.  However, game framework by definition is less generic than the rest of the library, and although it fits many use cases, there are times where it may not fit a particular use case.  Some notable limitations of game framework include (but are not necessarily limited to):

- No out-of-box multi-z-level open-world map support
- No infinite map/chunkloading support
- No out-of-box support for objects taking up multiple squares

Although these limitations can be worked around in some cases, there can be cases when it is not desireable to use the integration library.  In these cases, you may not be able to use the integration library as-is.  However, it is open-source, meaning that in these cases you can still fork and modify it for your needs, or just use its architecture as a model/reference point for building your own from scratch.  Given that some elements of integration are non-trivial to write properly, this can still be of significant value.

## Rendering Strategy
Recall from the [SadConsole overview](02_sadconsole_overview.md#Differences-between-surfaces-and-entities) that there are some important distinctions between SadConsole entities, and a cell surface being rendered via a screen surface.  Notably, as a result of those distinctions, it would be too performance intensive to have every piece of terrain on a map be visually represented by a SadConsole `Entity`.

Also recall that GoRogue's "game framework" defines all map objects in terms of a single interface called `IGameObject`, which has a position and is used to represent both terrain and non-terrain objects.  There are some special restrictions put on game objects that reside in the terrain layer; primarily that while they are added to a `Map` they can't be moved (attempting to do so will produce an error).  This allows the game framework to employ certain optimizations to efficiently deal with terrain, including storing terrain in a grid view instead of a spatial map.  This distinction is useful to know, because it matches up fairly well with the distinction between SadConsole's cell surfaces and its entities.

It is obviously beneficial to have non-terrain map objects be represented by SadConsole `Entity`.  Since rendering terrain using `Entity` instances is not viable, however, a different approach is required here.  Broadly speaking, there are two basic ways to go about rendering game framework terrain in SadConsole:

- Have each piece of terrain record its appearance.  Copy the appearances for terrain on the map to a corresponding `ColoredGlyph` within a cell surface, and ensure to update that surface when any terrain's appearance changes.
    - This is generally the simplest approach to implement, and it is viable for a fair number of use cases.  However, it is not the most efficient method, and can be error-prone if written generically.  Therefore, the integration library takes a different approach.
- Have each piece of terrain record its appearance.  Create a custom `ICellSurface` which retrieves its `ColoredGlyphs` by accessing the terrain layer of the `Map` and retrieving its appearance field, instead of retrieving them from a completely separate array of `ColoredGlyphs`.
    - This is the core approach the integration library takes.  It is not as simple to implement as the first approach, but the burden of the additional work falls to the library writers to implement.  It also enables more straightforward integration of other corresponding systems of the two libraries, such as their component systesms.  Ultimately, from the perspective of a user, this method results in a more efficient, less error prone, and easier to use product.

Understanding that the integration library takes the second approach will help you to better comprehend its core architecture.

## Component Integration
Recall that SadConsole's `IScreenObject` interface provides a facility for attaching and detaching components to and from objects.  These components may handle input and participate in the `Update`/`Render` loops.  Also note that GoRogue's `IGameObject` interface provides a separate facility for attaching components to implementing objects, and GoRogue's `Map` also provides a similar facility.

In all of these cases, each library provides access to its components via a separate field on the object the components are being attached to.  For example, components attached to a SadConsole `IScreenObject` are accessible via the `IScreenObject.SadComponents` field, and GoRogue components are similarly accessible via a field called `GoRogueComponents`.

Considering that each of these component systems serve different purposes and provide differing functionality, this can obviously cause some confusion when the libraries are integrated.  If an object implements `IGameObject` and also inherits from `Entity`, for example, it would have both a SadConsole component system (from `Entity`, because it derives from `ScreenObject`), and a GoRogue component system.  A SadConsole component _could_ be added to the GoRogue component system, but none of the SadConsole-specific functionality would work if it wasn't also added to SadConsole's.  Similarly, a component implementing GoRogue's `IParentAwareComponent` interface _could_ be added to SadConsole's component system (provided it also implements SadConsole's `IComponent`); but in this case none of the `IParentAwareComponent` functionality would work properly.

The integration library provides a solution to this by integrating the component systems into one property, in places where they are both present.  You will find an `AllComponents` field on all objects which have both SadConsole and GoRogue component functionality.  Any component added via this field will automatically be added to both SadConsole's and GoRogue's component systems as needed.  For example, an object added to this field could simultaneously implement SadConsole's `IComponent` and use that interface to hook into the `Update` loop, and also implement GoRogue's `IParentAwareComponent` and benefit from that functionality.  The integration library also provides convenient base classes for implementing components as such, which are discussed in the integration library's documentation.

Note that there are some disparities in where each component system is offered; notably, `RogueLikeCell` (eg. terrain) objects support GoRogue components, but _not_ SadConsole components; and as such, `RogueLikeCell` does not have an `AllComponents` field.  You may still add components to the `GoRogueComponents` field of terrain; however they cannot take advantage of any functionality defined by SadConsole's component system (such as participation in the `Update` or `Render` loops).  Each terrain object having components which could participate in the `Update`/`Render` loop directly would be far too performance intensive to maintain.

# Basic Object Hierarchy
The basic object hierarchy for the core SadConsole structures is described [here](https://github.com/thesadrogue/TheSadRogue.Integration/tree/main/TheSadRogue.Integration#thesadrogueintegration).  Understanding the functionality provided by these structures is crucial to understanding how to properly utilize the library.

## Basic Use Cases
The following is a quick-reference describing some basic use cases and the classes to use.  This is NOT intended to cover every possible use case for the library, nor to be a complete reference for each of the classes mentioned.  It is simply designed to be a quick-start guide for determining where you may want to look in order to implement a certain type of functionality.

| Use Case                                             | Class/Config                 |
| ---------------------------------------------------- | ---------------------------- |
| Terrain-Layer Objects                                | `RogueLikeCell`              |
| Non-Terrain Layer Objects (player, monsters, etc)    | `RogueLikeEntity`            |
| Map Rendered at a Single Point on Screen             | `RogueLikeMap` (specify DefaultRenderer parameters to constructor, or specify null and assign to the DefaultRenderer field of the Map later) |
| Map Rendered at Different Points on Screen           | `RogueLikeMap` (specify null default renderer parameters to constructor); call `CreateRenderer` as needed then add BOTH the Map AND any renderers to the SadConsole screen hierarchy) |
| Creating Player Controls                             | `PlayerKeybindingsComponent` |
| Controlling Visibility Based on FOV                  | `FieldOfView` namespace      |
| Creating Components to Attach to Non-Terrain Objects | `RogueLikeComponentBase`/`RogueLikeComponentBase<T>` |
| Creating Components to Attach to Terrain Object      | Any class (do NOT use `IComponent` from SadConsole; its functionality is not supported in terrain components) |
| Creating Components to Attach to Maps                | `RogueLikeComponentBase`/`RogueLikeComponentBase<T>` |

# Example Code
In addition to this document, the forms of documentation currently available for the integration library are listed in the [readme](README.md#documentation).  In particular, you are encouraged to look through the [ExampleGame](https://github.com/thesadrogue/TheSadRogue.Integration/tree/main/ExampleGame) project and the code given to you in the new project template, as those sources demonstrate some of the basic features of the library.