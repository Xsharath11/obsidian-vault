Having `///` before a function definition is Rust's way of handling docstrings

#### Making Rectangular rooms
```
pub fn new_map_rooms_and_corridors() -> Vec<TileType> {
	let mut map = vec![TileType::Wall, 80*50];
	map
}
```
- *We are creating a map with all walls*
- Changing the map is as simple as changing
  `gs.ecs.insert(...)` with the new function


a