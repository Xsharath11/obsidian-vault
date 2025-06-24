```
String[] foods = {"pizza", "tacos", "hamburger"};
String[] foods = {}; // Empty array but wrong way to initialise empty array
String[] foods = new String[3]; // Right way to init empty array

// Now we can do:
foods[0] = "pizza";
foods[1] = "tacos";
foods[2] = "hamburger";
```

### To declare array size dynamically:
```
size = scanner.nextInt();
scanner.nextLine(); // After accepting an int or a double, clear the input buffer since they do not accept the next line character
foods = new String[size];
```

### Declaring a dynamic resizeable list:
```
private List<Run> runs = new ArrayList<>();
```
![[Pasted image 20250609133426.png]]
