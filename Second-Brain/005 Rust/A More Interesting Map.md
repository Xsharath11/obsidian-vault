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

In `rect.rs`, We add the following to create rectangular rooms:
```
pub struct Rect {
    pub x1 : i32,
    pub x2 : i32,
    pub y1 : i32,
    pub y2 : i32
}

impl Rect {
    pub fn new(x:i32, y: i32, w:i32, h:i32) -> Rect {
        Rect{x1:x, y1:y, x2:x+w, y2:y+h}
    }

    // Returns true if this overlaps with other
    pub fn intersect(&self, other:&Rect) -> bool {
        self.x1 <= other.x2 && self.x2 >= other.x1 && self.y1 <= other.y2 && self.y2 >= other.y1
    }

    pub fn center(&self) -> (i32, i32) {
        ((self.x1 + self.x2)/2, (self.y1 + self.y2)/2)
    }
}
```
- There is nothing new here with all of the functions doing as their name implies

Next, we need a function in *map.rs* which actually uses the struct and functions defined in *rect.rs* and creates the rooms on the given map:
```
fn apply_room_to_map(room : &Rect, map: &mut [TileType]) {
    for y in room.y1 +1 ..= room.y2 {
        for x in room.x1 + 1 ..= room.x2 {
            map[xy_idx(x, y)] = TileType::Floor;
        }
    }
}
```

Now, we actually need a function to create the map:
```
pub fn new_map_rooms_and_corridors() -> Vec<TileType> {
    let mut map = vec![TileType::Wall; 80*50];

    let room1 = Rect::new(20, 15, 10, 15);
    let room2 = Rect::new(35, 15, 10, 15);

    apply_room_to_map(&room1, &mut map);
    apply_room_to_map(&room2, &mut map);

    map
}
```

#### Making a Corridor
- We need two functions. 
- One for horizontal and one for vertical tunnels
```
fn apply_horizontal_tunnel(map: &mut [TileType], x1; i32, x2:i32, y: i32) {
	for x in min(x1, x2) ..= max(x1, x2) {
		let idx = xy_idx(x, y);
		if idx > 0 && idx < 80*50 {
			map[idx as usize] = TileType::Floor
		}
	}
}
```
- Similarly for vertical
#### Making a simple dungeon
```
pub fn new_map_rooms_and_corridors() -> Vec<TileType> {
	let mut map = vec![TileType::Wall; 80*50];
	// New Definitions 
	let mut rooms: Vec<Rect> = Vec::new();
	const MAX_ROOMS: i32 = 30;
	const MIN_SIZE: i32 = 6;
	const MAX_SIZE: i32 = 10;
	// RNG
	let mut rng = RandomNumberGenerator::new();
	for _ in 0..MAX_ROOMS {
		let w = rng.range(MIN_SIZE, MAX_SIZE);
		let h = rng.range(MIN_SIZE, MAX_SIZE);
		let x = rng.roll_dice(1, 80 - w - 1) - 1;
		let y = rng.roll_dice(1, 50 - h - 1) - 1;
		let new_room = Rect::new(x, y, w, h);
		let mut ok = true;
		for other_room in rooms.iter() {
			if new_room.intersect(other_room) {ok = false}
		}
		if ok {
			apply_room_to_map(&new_room, &mut map);
			// Joining the rooms together
			 if !rooms.is_empty() {
                let (new_x, new_y) = new_room.center();
                let (prev_x, prev_y) = rooms[rooms.len() - 1].center();
                if rng.range(0, 2) == 1 {
                    apply_horizontal_tunnel(&mut map, prev_x, new_x, prev_y);
                    apply_vertical_tunnel(&mut map, prev_y, new_y, new_x);
                } else {
                    apply_vertical_tunnel(&mut map, prev_y, new_y, prev_x);
                    apply_horizontal_tunnel(&mut map, prev_x, new_x, new_y);
                }
            }

			//
			rooms.push(new_room)
		}
	}

	map
}
```
