```
import java.util.Random

public class Main {
	public static void main(String[] args) {
		Random random = new Random();
		int num = random.nextInt(1, 6);
		double number = random.nextDouble(); // Random number between 0 and 1
		boolean isTrue = random.nextBoolean();
		
	}
}
```

nextDouble only returns a number between 0 and 1
nextInt takes a lower bound (inclusive), and an upper bound (exclusive) 