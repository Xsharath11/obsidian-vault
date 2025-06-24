```
public class main {
	public static void main(string[] args) {
		String name = "Sharath";

		int lenght = name.length();
		char letter = name.charAt(0);
		int index = name.indexOf("h");
	}
}
```
1. length -> returns length of string
2. charAt -> returns character at an index
3. indexOf -> returns first occurrence of a character
4. lastIndexOf -> returns last index of a character
5. toUpperCase -> 
6. toLowerCase
7. trim -> trims whitespaces around the text
8. replace("o", "a") -> replaces o with a
9. isEmpty -> returns a boolean depending on whether string is empty or not
10. contains("a") -> returns boolean depending on arg
11. equals("a string") -> checks if a string is equal to another
12. equalsIgnoreCase -> same as above but ignores case

## Intermediate methods
1. .substring() -> method used to extract a portion of a string
		Usage: 
			```String email = "sharath@gmail.com"
			String username = email.substring(0,email.indexOf("@"));