varargs: allow a method to accept a varying number of args making methods more flexible. no need for overloading. Java will pack args into an array

```
static int add(int... numbers){
	int sum = 0;
	for(int number : numbers){
		sum += number;
	}
	return sum
}
```
