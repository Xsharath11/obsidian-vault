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
A `World` is an ECS, provided by `Specs`. We need to extend our `State` struct to have a place to store the world

```
struct State {
	ecs: World
}
```

And now in `main`, when we create the world - we'll put an ECS into it:

```
let mut gs = State {
	ecs: World::new()
};
```

- Notice that `World::new()` is another constructor.
- It is a method inside the `World` type but without a reference to `self`
- Due to this, It does not work on existing worlds. It only creates new ones

The next thing to do is to is tell ECS about the components we have created. It is done as follows:
```
gs.ecs.register::<Position>();
gs.ecs.register::<Renderable>();
```
What this does:
- Tells our `World` to take a look at the types we are giving it, and do some internal magic to create storage systems for each of them
- `Specs` has made this easy. so long as it implements `Component`, You can put anything you like as a Component!
#### Creating entities
- Now we have a `World` that knows how to store `Position` and `Renderable` components
- In order to use them, they need to be attached to something in the game
- In the *ECS World*, that is called an *entity*
- They are little more than an identification number
- These identification numbers tell the ECS that an entity exists
- They can have any combination of components attached to them
Creating an Entity with a `Renderable` and a `Position`:
```
gs.ecs
	.create_entity()
	.with(Position { x: 40, y: 25 })
	.with(Renderable {
		glyph: rltk::to_cp437('@'),
		fg: RGB::named(rltk::YELLOW),
		bg: RGB::named(rltk::BLACK),
	})
	.build();
```
- This is known as the builder pattern in RUST
- This works because each function returns a copy of itself

Let us add a few more entities:
```
for i in 0..10 {
	gs.ecs
	.create_entity()
	.with(Position { x: i * 7, y: 20 })
	.with(Renderable {
		glyph: rltk::to_cp437('F'),
		fg: RGB::named(rltk::RED),
		bg: RGB::named(rltk::BLACK),
	})
	.build();
}
```
**Note**: `to_cp437` is a helper RLTK provides to let us use Unicode and get the equivalent member of the old DOS/CP437 character set

#### Iterating entities - a generic render system
Now, we need to actually render all of the entities we have created above

We need to replace the call to render "Hello Rust" with:
```
let positions = self.ecs.read_storage::<Position>();
let renderables = self.ecs.read_storage::<Renderables>();

for (pos, render) in (&positions, &renderables).join() {
	ctx.set(pos.x, pos.y, render.fg, render.bg, render.glyph)
}
```
- Here we are asking for read only access to the structure used to store the components of each type
- The join used here is very similar to a database join. It only returns entities that have both.

At this stage, we will be able to see static entities on the screen

**Additional Notes:**
1. If you change the bg for a particular entity, only the immediate background surrounding the entity is affected.
2. Notice how the `build` function for the `simple80x50` function is called as `build?()`
   We can achieve the same result by removing the `?` operator from there and applying it to the context variable in
   `rltk::main_loop(context?, gs)` *What does the `?` operator do?*^Further-Explorations-1
3. 

