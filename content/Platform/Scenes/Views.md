___
# Overview

A view is a managed Godot scene that is meant to represent a unique portion of the game a player is presented. Examples include the settings, level select, and start screens typically seen across games.

Note views are a subset of Godot scenes. We make this distinction to split the interfaces for working with scenes in the context of views v.s. other use cases. Other use cases of Godot scenes fall under what is referred to as **Templates** in this project.

- This separation mimics a pattern seen in some other game engines, notable Unity which scene and prefab concepts align closely with the view and template concepts described here. Godot doesn’t have this same separation natively.

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

- **Scene tree size**: A naive approach to structuring a game’s content is to simply have it all within one scene. For very small games this is fine, but as the game’s content scales not having some mechanism to load and unload portions of the game as needed can result in performance and maintainability impacts.

- **View data transferring**: One hurdle that arises when using separated views is how to share data between them. Separated views cannot have serialized references to each other, this results in your scenes either needing to be self-contained in that no outside, previous context is needed to determine behavior for a view, or using some pattern to share data from the existing scene(s) to others.
	- The later approach can be achieved simply with autoloading view references or a static class that can be written to and read from any view.
	- A potentially better approach than the ones above is to allow views to implement an interface for an "init" function that callers of the ViewManager can pass a data transfer object (DTO) when requesting to load a view.

<p align="center">
  <img width="5000" src="view-manager.drawio.svg">
</p>

