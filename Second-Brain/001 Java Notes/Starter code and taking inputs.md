Starter Code:
```
public class Main {
	public static void main(String[] args){
		...
	}
}
```

Taking Inputs:
```
import java.util.Scanner

public class main {
	public static void main(String[] args){
		Scanner scanner = new Scanner(System.in);
		String name = scanner.nextLine();
		int age = scanner.nextInt();
		System.out.println(name + "," + age);
		Scanner.close();
	}
}
```
scanner.next() -> one word i.e no spaces
scanner.nextLine() -> One line
scanner.Int
scanner.Boolean
scanner.Double

Common Issues:
```
import java.util.Scanner;  
  
public class Main {  
  
    public static void main(String[] args){  
        Scanner scanner = new Scanner(System.in);  
          
        System.out.print("Enter your age: ");  
        int age = scanner.nextInt();  
          
        System.out.print("Enter your favourite colour: ");  
        String color = scanner.nextLine();  
          
        System.out.println("You are " + age + " years old");  
        System.out.println("You like the color" + color);  
        scanner.close();  
  
    }  
}
```

Output: 
```
Enter your age: 222
Enter your favourite colour: You are 222 years old
You like the color

```

Why:
When we type in a number i.e 222, within the input buffer, there is still a \n. So the nextLine picks up the previous string and prints it out. Therefore, use:
`scanner.nextLine();` without being assigned to anything after accepting an int or a double