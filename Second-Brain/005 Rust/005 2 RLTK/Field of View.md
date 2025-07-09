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
- To act as a bridge between our map implementation and our RLTK, we have a few traits that will help us
- `map.rs`
```rust
impl Algorithm2D for Map {
	fn dimensions(&self) -> Point {
		Point::new(self.width, self.height)
	}
}
```
- This helps RLTK figure out a lot of other traits from the `dimensions` function: point indexing, bounds checking etc
- We also need to support `BaseMap` 
- `map.rs`
```rust
impl BaseMap for Map {
	fn is_opaque(&self, idx:usize) -> bool {
		self.tiles[idx as usize] == TileType::Wall
	}
}
```
- returns true if the tile is a wall, else false
- Needs to be expanded once there are more types of tiles

# Asking RLTK for  a viewshed: The system
- Modifying the `visibility_system.rs`:
```rust
use specs::prelude::*;
use super::{Viewshed, Position, Map};
use rltk::{field_of_view, Point};

pub struct VisibilitySystem {}

impl<'a> System<'a> for VisibilitySystem {
    type SystemData = ( ReadExpect<'a, Map>,
                        WriteStorage<'a, Viewshed>, 
                        WriteStorage<'a, Position>);

    fn run(&mut self, data : Self::SystemData) {
        let (map, mut viewshed, pos) = data;

        for (viewshed,pos) in (&mut viewshed, &pos).join() {
            viewshed.visible_tiles.clear();
            viewshed.visible_tiles = field_of_view(Point::new(pos.x, pos.y), viewshed.range, &*map);
            viewshed.visible_tiles.retain(|p| p.x >= 0 && p.x < map.width && p.y >= 0 && p.y < map.height );
        }
    }
}
```
- We've added a `ReadExpect<'a, Map>` - meaning that the system should be passed our `Map` for use. We used `ReadExpect`, because not having a map is a failure.
- In the loop, we first clear the list of visible tiles.
- Then we call RLTK's `field_of_view` function, providing the starting point (the location of the entity, from `pos`), the range (from the viewshed), and a slightly convoluted "dereference, then get a reference" to unwrap `Map` from the ECS.
- Finally we use the vector's `retain` method to delete any entries that _don't_ meet the criteria we specify. This is a _lambda_ or _closure_ - it iterates over the vector, passing `p` as a parameter. If p is inside the map boundaries, we keep it. This prevents other functions from trying to access a tile outside of the working map area.

This will now run every frame (which is overkill, more on that later) - and store a list of visible tiles.

# Rendering Visibility - Badly
- change our `draw_map` function to retrieve the map and the viewshed:
```rust
use super::{Player, Rect, Viewshed, World};
use rltk::{Algorithm2D, BaseMap, Point, RandomNumberGenerator, Rltk, RGB};
use specs::{Join, WorldExt};

pub fn draw_map(ecs: &World, ctx: &mut Rltk) {
    //let map = ecs.fetch::<Map>();
    let mut viewsheds = ecs.write_storage::<Viewshed>();
    let mut players = ecs.write_storage::<Player>();
    let map = ecs.fetch::<Map>();

    for (_player, viewshed) in (&mut players, &mut viewsheds).join() {
        let mut x = 0;
        let mut y = 0;

        for tile in map.tiles.iter() {
            let mut y = 0;
            let mut x = 0;
            for tile in map.tiles.iter() {
                let pt = Point::new(x, y);
                if viewshed.visible_tiles.contains(&pt) {
                    match tile {
                        TileType::Floor => {
                            ctx.set(
                                x,
                                y,
                                RGB::from_f32(0.5, 0.5, 0.5),
                                RGB::from_f32(0., 0., 0.),
                                rltk::to_cp437('.'),
                            );
                        }
                        TileType::Wall => {
                            ctx.set(
                                x,
                                y,
                                RGB::from_f32(0.0, 1.0, 0.0),
                                RGB::from_f32(0., 0., 0.),
                                rltk::to_cp437('#'),
                            );
                        }
                    }
                }
                x += 1;
                if x > 79 {
                    x = 0;
  
```
- The above lets us see what the player can see alright
- Performance is bad
- We are basically just checking for every point if it falls within the visible_tiles for the player in the current state
- There is no memory

# Expanding map to include revealed tiles
- To simulate map memory:
	- Extend `Map` class to include a `revealed_tiles` struct
	- It is just a `bool` for each tile in the map
	- If true: we know what's there
```rust
#[derive(Default)]
pub struct Map {
	pub tiles: Vec<TileType>,
	pub rooms: Vec<Rect>,
	pub width: i32,
	pub height: i32,
	pub revealed_tiles: Vec<bool>
}
```
- Due to this,  we also need to fill in the new type in the function that fills in the map: `new_rooms_and_corridors`:
```rust
let mut map =  Map {
	tiles: vec![TileType::Wall; 80*50],
	rooms: Vec::new(),
	width: 80,
	height: 50,
	revealed_tiles: vec![false; 80*50]
};
```
- change `draw_map` to look at this value rather than iterating the component each time
```rust
pub fn draw_map(ecs: &World, ctx: &mut Rltk) {
    //let map = ecs.fetch::<Map>();
    let map = ecs.fetch::<Map>();

    let mut y = 0;
    let mut x = 0;
    for (idx, tile) in map.tiles.iter().enumerate() {
        if map.revealed_tiles[idx] {
            match tile {
                TileType::Floor => {
                    ctx.set(
                        x,
                        y,
                        RGB::from_f32(0.5, 0.5, 0.5),
                        RGB::from_f32(0., 0., 0.),
                        rltk::to_cp437('.'),
                    );
                }
                TileType::Wall => {
                    ctx.set(
                        x,
                        y,
                        RGB::from_f32(0.0, 1.0, 0.0),
                        RGB::from_f32(0., 0., 0.),
                        rltk::to_cp437('#'),
                    );
                }
            }
        }
        x += 1;
        if x > 79 {
            x = 0;
            y += 1;
        }
    }  
}
```
- This renders a black screen since we are never updating the revealed_tiles
- Extend the `VisibilitySystem`to update the tiles
- Check to see if an entity is the player. If it is, update the maps revealed status
```rust
use super::{Map, Player, Position, Viewshed};
use rltk::{field_of_view, Point};
use specs::prelude::*;

pub struct VisibilitySystem {}

impl<'a> System<'a> for VisibilitySystem {
    type SystemData = (
        WriteExpect<'a, Map>,
        Entities<'a>,
        WriteStorage<'a, Viewshed>,
        WriteStorage<'a, Position>,
        ReadStorage<'a, Player>,
    );
    fn run(&mut self, data: Self::SystemData) {
        let (mut map, entities, mut viewshed, pos, player) = data;
        for (ent, viewshed, pos) in (&entities, &mut viewshed, &pos).join() {
            viewshed.visible_tiles.clear();
            viewshed.visible_tiles = field_of_view(Point::new(pos.x, pos.y), viewshed.range, &*map);
            viewshed
                .visible_tiles
                .retain(|p| p.x >= 0 && p.x < map.width && p.y < map.height && p.y >= 0);
            // If Player, reveal tiles
            let p: Option<&Player> = player.get(ent);
            if let Some(p) = p {
                for vis in viewshed.visible_tiles.iter() {
                    let idx = map.xy_idx(vis.x, vis.y);
                    map.revealed_tiles[idx] = true
                }
            }
        }
    }
}
```
- Getting entities list along with components
- obtaining read only access to Players Storage
- Iterate over them
- `let p : Option<&Player> = player.get(ent);` to see if this is the player. The rather cryptic `if let Some(p) = p` runs only if there is a `Player` component. Then we calculate the index, and mark it revealed 

At this stage, the game should be much quicker and it also remembers where the player has been
# Speeding up even more - recalc visibility when needed
- let us add a `dirty` flag to our `Viewshed` component
```rust
#[derive(Component)]
pub struct Viewshed {
	pub visibile_tiles: Vec<rltk::Point>,
	pub range: i32,
	pub dirty: bool
}
```
- We need to init in `main.rs` to say that the viewshed is dirty
```rust
.with(Viewshed{ visible_tiles : Vec::new(), range: 8, dirty: true })
```
- The system can now be extended to check if the `dirty` flag is true and only recalc if it is true and set dirty flag to false after
- in `try_move_player`:
```rust
pub fn try_move_player(delta_x: i32, delta_y: i32, ecs: &mut World) {
    let mut positions = ecs.write_storage::<Position>();
    let mut players = ecs.write_storage::<Player>();
    let mut viewsheds = ecs.write_storage::<Viewshed>();
    let map = ecs.fetch::<Map>();

    for (_player, pos, viewshed) in (&mut players, &mut positions, &mut viewsheds).join() {
        //pos.x = min(79, max(0, pos.x + delta_x));
        //pos.y = min(49, max(0, pos.y + delta_y));
        let destination_idx = map.xy_idx(pos.x + delta_x, pos.y + delta_y);
        if map.tiles[destination_idx] != TileType::Wall {
            pos.x = (pos.x + delta_x).clamp(0, 79);
            pos.y = (pos.y + delta_y).clamp(0, 49);

            viewshed.dirty = true;
        }
    }
}
```
- This should be pretty familiar by now: we've added `viewsheds` to get write storage, and included it in the list of component types we are iterating. Then one call sets the flag to `true` after a move.

The game now runs _very_ fast once more, if you type `cargo run`.

# Greying out what we remember but cannot see:
