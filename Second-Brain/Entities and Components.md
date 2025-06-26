Game developers have moved away from an OOP based design mostly because it starts to get quite confusing once we start moving past the initial scope of the game and we may end up recreating whole classes with the same logic which have been defined in other classes:

For example:
Let us take a class `MeleeMob` and a class `ArcherMob`.
If we need to create an NPC which can use ranged as well as melee attacks, we will need a create a whole new class with logic combined from both the previous classes.

Next, suppose, down the line, the NPC needs to turn friendly, that poses new problems

Here comes in the Idea of an Entity Component based design

#### Entity Component Based Design
- An **Entity** is a thing. Anything. It is basically an identification number for the type of mob it is at a very high level
- To each entity, we can add **Components**. 
- Components are just data grouped by whatever properties we want our entities to have.
Example:
We can build the same set of mobs listed above with components for:
- `Position`
- `Renderable`
- `Hostile`
- `MeleeAI`
- Some `CombatStats` etc
At any point, for any mob we can just change the components and change the type of mob

Basically, components are just like the inheritance tree but rather than *inheriting* traits, we *compose* them by adding *components* until it does whatever we want. This is called **Composition**

---
#### Including specs in the project
```
cargo.toml
[dependencies]
rltk = { version = "0.8.7" }
specs = "0.20.0"
specs-derive = "0.4.1"
```

Also added some lines of code analogous to imports
```
src/main.rs
use rltk::{GameState, Rltk, RGB, VirtualKeyCode};
use specs::prelude::*;
use std::cmp::{max, min};
use specs_derive::Component;
```

#### Defining a Position Component
```
struct Position {
	x: i32,
	y: i32,
}
```
This is a simple *Position* component which has an X and Y coordinate as *32 bit integers*.
- This is what is known as a `POD` or Plain old Data
- Only data, no logic
- Core part of pure ECS
- Logic is implemented elsewhere
- There are two reasons to do this:
	- It keeps all the code that *does something* elsewhere i.e in "Systems"
	- Performance: It is very fast to keep all of the positions next to each other in memory with no redirects

To use this *Position* Struct, we need to tell specs that it is a *Component*. It can be done as follows:
```
struct Position {
	x: i32,
	y: i32,
}

impl Component for Position {
	type Storage = VecStorage<Self>;
}
```
Lot of repeated typing. `specs-derive` helps us here;
Instead, we can just do:
```
#[derive(Component)]
struct Position {
	x: i32,
	y: i32,
}
```

`#[derive(x)]` is a macro that says "*from my basic data, derive the boilerplate required for **x***"

#### Defining a renderable component
We need to create a new component - `Renderable`
- It will contain a foreground, a background and a glyph (such as @)
```
#[derive(Component)]
struct Renderable {
	glyph: Rltk::FontCharType,
	fg: RGB,
	bg: RGB,
}
```

- RGB Comes from RLTK and represents a colour
- If we didn't do `use rltk::{... RGB}`, we would be typing `rltk::RGB` everytime
#### Worlds and Registration
The next component which we need to setup is the `World`
