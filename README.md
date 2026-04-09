# GraphicsEngine3D

GraphicsEngine3D is a from-scratch 3D graphics engine written in C#. The project is structured around a component-based object model, a custom scene/rendering pipeline, and a set of reusable geometry and utility systems.

## Project Overview

The engine is built around a small set of core ideas:

- `Gameobject` is the base object in the scene
- behavior is added through attachable components
- rendering is handled through mesh and camera components
- geometry is generated through factory classes
- scenes manage objects, cameras, updates, and rendering
- transforms handle spatial logic such as position, rotation, and local/world conversion

The design is component-based rather than inheritance-heavy. Instead of creating many specialized object subclasses, objects are built by attaching the components they need.

## Controls
- `W / A / S / D` for movement
- `E/Q` for up/down
- mouse for camera rotation
- click camera button for switching cameras

## Architectural Style

The engine follows a component-based object model.

This is not a full ECS (Entity Component System). `Gameobject` is still a real class that owns data and behavior, rather than being only an ID. However, the structure is ECS-like in the sense that functionality is split into modular components instead of being placed into deep inheritance trees.

The main architectural pattern is:

- `Gameobject` owns a `Transform` and a list of components
- components define behavior such as rendering, input, scripting, camera logic, or collision
- `Scene` coordinates updating and rendering
- the renderer uses the active camera to project mesh data into screen space

## Core Structure
### `Window`
`Window` acts as the application host. It is responsible for creating the scene, maintaining runtime state, and starting the engine loop.

This is the highest-level entry point of the project.

## `Scene`

`Scene` manages the state of the world.

Its responsibilities include:

- storing game objects
- storing UI elements
- selecting the active camera
- updating objects
- determining render order
- invoking rendering

`Scene` is the main orchestration layer of the engine.

### `Gameobject`
`Gameobject` is the base scene object.

A `Gameobject` contains:

- a `Transform`
- a list of attached components

Its main role is to act as a container for behavior. Objects are not specialized through inheritance; instead, they become useful by attaching components.

Examples:

- a visible object = `Gameobject` + `MeshComponent`
- a controllable camera = `Gameobject` + `ControllerComponent` + `CameraComponent`
- a scripted object = `Gameobject` + `ScriptableComponent`

### `Transform`
`Transform` stores an object's spatial data and orientation logic.

It is responsible for:

- position
- rotation
- directional vectors such as forward, up, and right
- converting between local and world space

This is one of the most important utility systems in the engine because it connects movement, rendering, and camera projection.

### Component System
Components are the main extension mechanism in the engine.

Each component is attached to a `Gameobject`, initialized with that object, and updated through the object's update cycle.

This allows behavior to be composed rather than inherited.

#### `MeshComponent`
`MeshComponent` is responsible for renderable geometry.

This component stores mesh data and handles:

- face input
- triangulation
- vertex reuse
- edge reuse
- rendering through the active camera
- wireframe or filled drawing modes

One of the major structural changes in the project was moving mesh logic into `MeshComponent` instead of treating mesh objects as specialized `Gameobject` subclasses.

Internally, `MeshComponent` contains its own geometry representation:

- `Face`
- `Tri`
- `Edge`
- `Vertex`

Only `Face` is exposed as public input geometry. The lower-level geometry types are implementation details of the mesh system.

This keeps the public API simpler and prevents the rest of the engine from depending on internal mesh topology details.

#### `CameraComponent`
`CameraComponent` defines how a scene is viewed and projected.

The camera system stores projection-related data such as:

- field of view
- aspect ratio
- near clip
- far clip
- focal length
- priority

The camera is implemented as a component rather than as a special hardcoded object type. This makes the camera part of the same object model as everything else in the engine.

#### `PerspectiveCameraComponent`
This component implements perspective projection.

Objects farther from the camera appear smaller, which makes the rendered view behave more like a normal 3D camera.

#### `OrthographicCameraComponent`
This component implements orthographic projection.

Object size does not change with distance, making it useful for debugging, technical views, or non-perspective rendering.

#### `ControllerComponent`
`ControllerComponent` handles object movement through input.

It is typically attached to a camera object, allowing the camera's transform to be moved or rotated through user input.

This keeps input logic separate from camera math and separate from scene logic.

#### `ScriptableComponent`
`ScriptableComponent` allows object-specific logic to be attached using delegates.

Its purpose is to provide a lightweight scripting mechanism without creating dedicated subclasses for every custom behavior.

This is useful for rapid testing, prototype logic, and object-level behaviors.

#### `CollisionComponent`
`CollisionComponent` handles collision-related behavior.

Based on the current project direction, this subsystem is used for checking object overlap and supporting collision-aware behavior. It appears to be part of a broader move toward modular physics systems.

### Mesh and Geometry Logic
The mesh system is one of the central parts of the engine.

Geometry is provided as one or more `Face` objects. A face is defined by an array of points. These faces are then triangulated into triangles so they can be rendered.

#### Face Triangulation
Faces are triangulated using a fan-style method:

- the first point is treated as the anchor
- additional triangles are formed using adjacent points

This works correctly for convex polygons.

It does not fully support:

- concave polygons
- polygons with holes

That limitation is already implied by the implementation and is important if the mesh system is extended later.

#### Vertex and Edge Reuse
The mesh system uses dictionaries to avoid duplicate geometry data where possible.

It tracks:

- vertices by position
- edges by endpoint pair
- triangles by point combination

This means identical geometry elements are reused instead of recreated repeatedly.

This gives the mesh system a clearer internal topology and makes it more structured than a simple "draw raw triangles" pipeline.

### Rendering Logic
Rendering is driven by the relationship between:

- `Scene`
- `CameraComponent`
- `MeshComponent`
- `Transform`

At a high level, the flow is:

1. the scene selects the active camera
2. game objects are filtered for renderable mesh components
3. render order is determined
4. mesh vertices are transformed into world/view space
5. points are projected into 2D screen space
6. polygons are drawn using the graphics context

The engine currently behaves like a custom software-style renderer built on top of standard C# drawing APIs rather than a hardware-accelerated engine framework.

### Geometry Factories
Primitive shape creation is separated into factory classes.

These include:

- `Cuboid`
- `Plane`
- `Pyramid`
- `Line`

These factory classes are responsible for generating reusable mesh data for common shapes.

This separation keeps shape construction logic out of scene logic, rendering logic, and object logic.

### Scene Construction Pattern
A typical object is created in two steps:

1. create a `Gameobject`
2. attach one or more components

Example structure:

- create object
- assign position/rotation through `Transform`
- attach a `MeshComponent`
- optionally attach controller, scripting, or collision components

This pattern is used throughout the project and is the intended way to build new engine objects.

### Design Decisions
Several design decisions define the structure of the project.

#### Component-Based Composition

The engine favors composition over inheritance.

Instead of creating many special object subclasses, behavior is attached through components.

This makes the engine easier to extend and keeps systems more modular.

#### Encapsulation of Mesh Internals

Mesh topology types are hidden inside `MeshComponent`.

This prevents the rest of the project from becoming tightly coupled to the internal geometry representation.

#### Separate Camera Abstractions

Perspective and orthographic cameras share a common base concept but are implemented as distinct camera component types.

This keeps projection logic organized and extensible.

#### Factory-Based Geometry Creation

Primitive geometry generation is isolated in factory classes rather than spread across the codebase.

This improves maintainability and keeps responsibilities clear.

### Current State of the Project

The project has evolved significantly from its original starting point.

Despite the earlier repository naming and remnants of the older idea, the codebase is better described as a general from-scratch 3D engine rather than a terrain or heightmap visualizer.

The engine currently contains:

- a component-based object model
- transform and spatial math systems
- scene management
- mesh rendering
- multiple camera types
- input/controller logic
- scripting support
- collision-oriented systems
- primitive mesh generation

### Areas to Revisit Later

If development resumes, the most likely systems to review first are:

- rendering order and visibility logic
- camera projection code
- mesh triangulation limitations
- collision/physics module structure
- scene construction workflow
- remaining naming cleanup from earlier versions of the project

## Summary

GraphicsEngine3D is a component-based 3D engine built from scratch in C#.

Its structure is centered around:

- `Gameobject`
- `Transform`
- attachable components
- custom mesh handling
- camera abstractions
- scene-managed rendering

The project is organized to support experimentation with rendering, object composition, spatial math, and engine architecture while keeping the main systems modular and relatively easy to extend.

## Author
Created by [UnbrokenHunter](https://github.com/UnbrokenHunter)
