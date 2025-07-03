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
```rust
cargo.toml
[dependencies]
rltk = { version = "0.8.7" }
specs = "0.20.0"
specs-derive = "0.4.1"
```

Also added some lines of code analogous to imports
```rust
src/main.rs
use rltk::{GameState, Rltk, RGB, VirtualKeyCode};
use specs::prelude::*;
use std::cmp::{max, min};
use specs_derive::Component;
```

#### Defining a Position Component
```rust
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
```rust
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
```rust
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
```rust
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

```rust
struct State {
	ecs: World
}
```

And now in `main`, when we create the world - we'll put an ECS into it:

```rust
let mut gs = State {
	ecs: World::new()
};
```

- Notice that `World::new()` is another constructor.
- It is a method inside the `World` type but without a reference to `self`
- Due to this, It does not work on existing worlds. It only creates new ones

The next thing to do is to is tell ECS about the components we have created. It is done as follows:
```rust
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
```rust
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
```rust
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
```rust
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

#### Example System - Random Movement
**Steps:**
1. New Component to handle the movement. - `LeftMover`
2. Tell ECS to use the component. i.e `gs.ecs.register::<LeftMover>();`
3. Add the Component to the entity builder logic `.with(LeftMover{})`
4. Actually Make them move

Steps 1 through 3 are straight forward

For step 4, we will need to define a new system:
*Systems are a way of keeping entity/component logic together, and have them run independently*

```rust
struct LeftWalker {}

impl<'a> System<'a> for LeftWalker {
    type SystemData = (ReadStorage<'a, LeftMover>, 
                        WriteStorage<'a, Position>);

    fn run(&mut self, (lefty, mut pos) : Self::SystemData) {
        for (_lefty,pos) in (&lefty, &mut pos).join() {
            pos.x -= 1;
            if pos.x < 0 { pos.x = 79; }
        }
    }
}
```
- Line 1 just defines an empty structure - somewhere to attach the logic
- `impl<'a> System<'a> for LeftWalker` - We are implementing Specs' System Trait for the `LeftWalker` struct
- The `'a` are *lifetime specifiers*. [Ref](https://doc.rust-lang.org/book/ch10-00-generics.html)
- `type SystemData` - defining a type to tell Specs what the system requires
- In this case, it requires read access to `LeftMover` Components and Write access to `Position` Components since the position will be changing
- `fn run` is the actual trait implementation which is required by the `impl System`
- For loop here is very similar to the one defined before. It will run once for each entity that has a `LeftMover` and a `Position` Component
- The underscore before the `LeftMover` variable is to tell rust that we are not using this variable and it is not a bug [[General Rust notes#^533349]]

Notice how what we have written is very similar to our rendering code. But instead of calling *in* to the ECS, *the ECS system is calling into* our function/system. It can be a tough judgement call on which to use
- If your system just needs data from the ECS, then a system is a right place to put it
- If it also needs access to other parts of your program, it is better implemented on the outside - calling in^0a680e

 **Now that we have written our state, we need to be able to use it.**
 To do this, we need to add a `run_systems` function to our state:
```rust
impl State {
	fn run_systems(&mut self) {
		let mut lw = LeftWalker{};
		lw.run_now(&self.ecs);
		self.ecs.maintain();
	}
}
```
1. `impl State` - We would like to implement functionality for `State`
2. `fn run_systems(&mut self)` - we are defining a function, mutable, access to self. i.e It can access data in its instance of `State` using the `self.` keyword
3. `let mut lw = LeftWalker{};` - Creates a new mutable instance of the `LeftWalker` System
4. `lw.run_now(&self.ecs)` - tells the system to run and tells it how to find the ECS
5. `self.ecs.maintain()` - tells specs that if any changes were queued up by the syatems, they should be applied to the world now
To finally run the system, we need to add the following in the `tick` function:
```rust
self.run_systems();
```

#### Moving the Player
We need to know which entity is the Player.
For this,  we will make a new tag component
```rust
#[derive(Component, Debug)]
struct Player {}
```

We need to add it to the registration:
```rust
gs.ecs.register::<Player>();
```

We need to then add it to the Players entity:
```rust
gs.ecs
	.create_entity()
	.with(Position {x: 40, y: 25})
	.with(Renderable {
		glyph: rltk::to_cp437('@'),
		fg: RGB::named(rltk::Yellow),
		bg: RGB::named(rltk::BLACK),
	})
	.with(Player{})
	.build();
```

Next, we need to implement a new function: `try_move_player`
```rust
fn try_move_player(delta_x: i32, delta_y: i32, ecs: &mut World) {
	let mut positions = ecs.write_storage::<Position>();
	let mut players = ecs.write_storage::<Players>();

	for (_player, pos) in (&mut players, &mut positions).join() {
		pos.x = min(79, max(0, pos.x + delta_x));
		pos.y = min(49, max(0, pos.y + delta_y));
	}
} 
```
- As we can make out by now, the function gains write access to `Position` and `Player`
- Joins the two
- it will only work on entities that have both the player and Position component
- It just adds the delta value to x and y

#### We need to define another function to read keyboard input
```rust
fn player_input(gs: &mut State, ctx: &mut Rltk) {
	match ctx.key {
		None => {} // Nothing happend
		Some(key) => match key {
			VirtualKeyCode::Left => try_move_player(-1, 0, &mut gs.ecs),
			VirtualKeyCode::Right => try_move_player(1, 0, &mut gs.ecs),
			VirtualKeyCode::Up => try_move_player(0, -1, &mut gs.ecs),
			VirtualKeyCode::Down => try_move_player(0, 1, &mut gs.ecs),
			_ => {}
		}
	}
}
```

Finally we need to add the function into `tick`:
```rust
player_input(self, ctx);
```

By this point we will have a moveable player on the screen, with some other moving characters

---
**Note**:
```rust
fn try_move_player(delta_x: i32, delta_y: i32, ecs: &mut World) {
    let mut positions = ecs.write_storage::<Position>();
    let mut players = ecs.write_storage::<Player>();

    for (_player, pos) in (&mut players, &mut positions).join() {
        //pos.x = min(79, max(0, pos.x + delta_x));
        //pos.y = min(49, max(0, pos.y + delta_y));
        pos.x = (pos.x + delta_x).clamp(0, 79);
        pos.y = (pos.y + delta_y).clamp(0, 49);
    }
}

```
We can use the clamp function to achieve the same functionality

[[Walking a Map]]