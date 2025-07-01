#### Defining map tiles
- We need to start by allowing tile types:
	- Walls
	- Floors
- We can use an `enum` to represent them
```
#[derive(PartialEq, Copy, Clone)]
enum TileType {
	Wall, Floor
}
```
- Clone adds a `.clone()` method to the type. Allowing a copy to be made programatically.
- Copy changes the default from moving the object on assignment to making a copy. basically, `tile1 = tile2` leaves both values valid and not in a "Moved from" State.
	- *Somewhat like a deepcopy*
- `PartialEq` allows us to use `==` to see if two tile types match
- If we didn't *derive* these features, `tile_type == TileType::Wall` would fail to compile

#### Building a Simple Map
- We'll make a function that returns a vector of tiles (`vec`).   
- We'll use a vector sized to the whole map
- For this, we need a way to figure out which array index is at given x/y pos
```
pub fn xy_idx(x: i32, y: i32) -> usize {
	(y as usize * 80) + x as usize
}
```
- Multiplies y position by map width i.e 80 and adds the x position
- Effectively maps the the location in memory for left to right reading
- Notice that the function body doesn't end with a semi colon
- In Rust, any function that ends with a statement that lacks a `;` at the end is considered the return statement for that function

#### Constructor function to make  map:
```
fn new_map() -> Vec<TileType> {
	let mut map = Vec![TileType::Floor; 80*50];

	// Make Boundary Walls
	for x in 0..80 {
		map[xy_idx(x, 0)] = TileType::Wall;
		map [xy_idx(x, 49)] = TileType::Wall;
	}
	for y in 0..50 {
		map[xy_idx(0, y)] = TileType::Wall;
		map[xy_idx(79, y)] = TileType::Wall;
	}
	// Next, RNG walls
	let mut rng = rltk::RandomNumberGenerator::new();

	for _i in 0..400 {
		let x = rng.roll_dice(1, 79);
		let y = rng.roll_dice(1, 49);
		let idx = xy_idx(x, y);
		if idx != xy_idx(40, 25) {
			map[idx] = TileType::Wall;
		}
	}
	map
}
```
- `Vec` is a Rust Vector.
	- Like an array but doesn't have a size limit
	- You can `push` and `remove` as you go
	- [Ref1](https://doc.rust-lang.org/rust-by-example/primitives/array.html) [Ref2](https://doc.rust-lang.org/rust-by-example/std/vec.html)
- `let mut map = Vec![TileType::Floor; 80*50]`:
	- `Vec!` is a Rust macro
		- `!` is a way of telling rust that this is a procedural macro as opposed to a derive macro
		- Procedural macros run like a function. They define a procedure
		- Takes it's params in square brackets
		- First param is the value for each element in the new vector i.e `Floor`
		- Second param is how many tiles we need to create i.e 80 x 50
		- *instead of using `vec!`, we can define map as a vector and push inside a for loop. Which is exactly what the macro is doing for us*
	- RLTK does not have an exclusive range which is why roll_dice is given the said parameters

#### Making the map visible to the World
- Specs includes a concept of "resources" - Shared data which the whole ECS can use, So in our main function we need to add the randomly generated map to use
```
gs.ecs.insert(new_map());
```

The map is now available from anywhere the ECS can see! Now inside your code, you can access the map with the rather unwieldy `let map = self.ecs.get_mut::<Vec<TileType>>();`; it's available to systems in an easier fashion. There's actually _several_ ways to get the value of map, including `ecs.get`, `ecs.fetch`. `get_mut` obtains a "mutable" (you can change it) reference to the map - wrapped in an optional (in case the map isn't there). `fetch` skips the `Option` type and gives you a map directly. You can learn more about this [in the Specs Book](https://specs.amethyst.rs/docs/tutorials/04_resources.html).

#### Draw the map
- Now that we have a map available, we need to render it on the screen
```
fn draw_map(map: &[TileType], ctx: &mut Rltk) {
	let mut y = 0;
	let mut x = 0;
	for tile in map.iter() {
		// Render depending on the file type
		match tile {
			TileType::Floor => {
				ctx.set(x, y, RGB::from_f32(0.5, 0.5, 0.5), RGB::from_f32(0., 0., 0.), rltk.to_cp437('.'));
			}
			TileType::Wall => {
				ctx.set(x, y, RGB::from_f32(0.0, 1.0, 0.0), RGB::from_f32(0., 0., 0.), rltk.to_cp437('#'));
			}
		}
		// Move the coords
		x += 1;
		if x > 79 {
			x = 0;
			y += 1;
		}
	}
}
```
- In the declaration we pass in `&[TileType]` rather than `&Vec<TileType>`
- This allows us to pass in slices of the map if we so choose
- Not useful yet but maybe in the future

We also need to call this function in the tick function
```
let map = self.ecs.fetch::<Vec<TileType>>();
draw_map(&map, ctx);
```
Must be placed below `player_input` and `run_systems` as mutable borrow of self occurs there whereas immutable borrow of self occurs before it if the above lines are placed above ^42e5c1
#### Making walls solid
At this point, we will have a map where we can move around but the walls are not solid and we can move through them

To achieve this, we just need to make small changes to the `try_move_player` function:
```
fn try_move_player(delta_x: i32, delta_y: i32, ecs: &mut World) {
    let mut positions = ecs.write_storage::<Position>();
    let mut players = ecs.write_storage::<Player>();
    let map = ecs.fetch::<Vec<TileType>>();

    for (_player, pos) in (&mut players, &mut positions).join() {
        //pos.x = min(79, max(0, pos.x + delta_x));
        //pos.y = min(49, max(0, pos.y + delta_y));
        let destination_idx = xy_idx(pos.x + delta_x, pos.y + delta_y);
        if map[destination_idx] != TileType::Wall {
            pos.x = (pos.x + delta_x).clamp(0, 79);
            pos.y = (pos.y + delta_y).clamp(0, 49);
        }
    }
}

```
- Fetch the map using ecs
- Before moving, make sure that the TileType is not a wall

