spoUp until now, we can see the whole map at once which beats the point of exploration. To tackle this issue, we need to add a Field of View (FoV)

## Map Refactor:
- We will keep map related functions and data together

# Adding the field of view component:
- Not just the player but we also need to consider what the monsters can see as well
- We will add field of view as a component to the entities
- In `components.rs` we add:
```rust
#[derive(Component)]
pub struct Viewshed {
	pub visible_tiles: Vec<rltk::Point>,
	pub range: i32
}
```
- In `main.rs` we need to add this component to ecs"
```rust
gs.ecs.register::<Viewshed>();
```
- Next we need to give the player the Viewshed Component
```rust
gs.ecs
    .create_entity()
    .with(Position { x: player_x, y: player_y })
    .with(Renderable {
        glyph: rltk::to_cp437('@'),
        fg: RGB::named(rltk::YELLOW),
        bg: RGB::named(rltk::BLACK),
    })
    .with(Player{})
    .with(Viewshed{ visible_tiles : Vec::new(), range : 8 })
    .build();
```

# A new system: Generic Viewsheds
- we will start by defining a new system to take care of this for us
- We want this to be generic since we want it to work for anything that can benefit from knowing what it can see
- We create a new file `visibility_system.rs`
```rust
use specs::prelude::*;
use super::{Viewshed, Position};

pub struct VisibilitySystem {}

impl <'a> System<'a> for VisibilitySystem {
	type Systemdata = (WriteStorage<'a, Viewshed>,
					   WriteStorage<'a. Position>);
	fn run(&mut self, (mut viewshed, pos): Self::SystemData) {
		for (viewshed, pos) in (&mut viewshed, &pos).join() {
		}
	}
}
```
- Now we need to adjust `run_systems` in `main.rs` to actually call the system
```rust
impl State {
	fn run_systems(&mut self) {
		let mut vis = VisibilitySystem {};
		vis.run_now(&self.ecs);
		self.ecs.maintain();
	}
}
```
- We also need to tell `main.rs` to use the new module:
```rust
mod visibility_system;
use visibility_system::VisibilitySystem;
```

# Asking RLTK for  a viewshed: Trait Impl
