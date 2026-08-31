A view is a managed Godot scene that is meant to represent a unique portion of the game a player is presented, think the different "screens" a player sees like the start, level select, and settings screens.
# Design

The view structure is designed around the concept of the game booting with at first only a **Bootstrap** scene that contains **Managers** needed across multiple **views** and loads the first view to be presented to the player.

<p align="center">
  <img width="400" src="scene-arch.drawio.svg">
</p>

## Bootstrap

As mentioned previously, the bootstrap scene is meant to hold managers that are needed across most, if not all views. As seen from the diagram, this includes the managers for SFX, Music, Steam, etc.

Managers are singleton nodes that by living in the Bootstrap scene and are guaranteed to exist when being referenced by any dependent view. This is because the Bootstrap scene is never unloaded and is "Ready" before the first view.

### ViewManager
The view manager is in charge of loading, discarding, and transferring data between views. This manager is built to address the following.

- <u>**Scene tree size**</u>: A naive approach to structuring a game’s content is to simply have it all within one scene. For very small games this is fine, but as the game’s content scales not having some mechanism to load and unload portions of the game as needed can result in performance and maintainability impacts.

- <u>**View data transferring**</u>: One hurdle that arises when using separated views is how to share data between them. Separated views cannot have serialized references to each other, this results in your scenes either needing to be self-contained in that no outside, previous context is needed to determine behavior for a view, or using some pattern to share data from the existing scene(s) to others.

	- The later approach can be achieved simply with autoloading view references or a static class that can be written to and read from any view.

	- A potentially better approach than the ones above is to allow views to implement an interface for an "init" function which would accept a data transfer object (DTO) parameter. DTOs would be passed to the ViewManager then passed again to the new view after being loaded.
<h4>Flow Diagram</h4>
<p align="center">
  <img width="1000" src="view-manager.drawio.svg">
</p>

Basic idea is that Godot scenes intended to be views will have the root node implement the `IView` interface which will implement the abstract static property `viewName` which will return a `ViewName` enum value.

- This setup allows `ViewManager` to get the name of the view (used in resolving the scene path) given the provided generic type enforced to implement `IView`.

- `ViewName` enum is not required here, but is more flexible in reusability than a loose string.

Once the new view has been loaded and the `ViewStrategy` behavior has been handled, The passed in `IViewDto` object is passed through to the `initView` function on the `IView` root node.

- `ViewStrategy` as of right now is very simple in that it only handles adding a scene or replacing all of them, this is likely to be expanded to have finer control over unloading behavior.

- Similar to the `viewName` setup, the `initView` function accepting of a specific type of `IViewDto` is enforced with an interface. If I wanted my view to accept a `FoobarViewDto`, I would have the view implement `IView<FoobarViewDto` which would make the implementation of `initView(FoobarViewDto ...)` required.