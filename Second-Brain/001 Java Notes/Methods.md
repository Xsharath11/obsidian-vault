Method: A block of code that can be executed when called

If a method needs to be called within a static method, that method must also be static. Why?

```
public class main {
	public static void main(string[] args) {
		...
		years = scanner.nextInt();
		happyBirthday(years);
	}
	static void happyBirthday(int years){
		System.out.println("Happpy Birthday!")
		System.out.println("You are " + years + "old!")
	}
}
```

Overloaded methods: methods that share the same name but  different parameters
signature = name + parameters

## Variable Scope
1. Class -> defined at a class level
2. Local -> defined within a method
In case of same name between local and class, local is given precedence since Java looks for local scope first