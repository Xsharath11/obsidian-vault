```
import java.util.Scanner;  
  
public class Main {  
  
    public static void main(String[] args){  
        Scanner scanner = new Scanner(System.in);  
  
        String day = scanner.nextLine();  
  
        switch(day){  
            case "Monday", "Tuesday", "Wednesday", "Thursday", "Friday" -> System.out.println("It is a weekday :sad:");  
            case "Saturday", "Sunday" -> System.out.println("It is a weekend");  
            default -> System.out.println("Is not a day");  
        }  
  
        scanner.close();  
  
  
    }  
}
```
