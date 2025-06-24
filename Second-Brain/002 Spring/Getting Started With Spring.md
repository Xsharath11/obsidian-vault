[Ref](https://start.spring.io/) -> Spring Initializer, quick starter

We can choose:
1. Build Tools: Maven, Gradle
2. Language: Java, Kotlin, Groovy
3. Spring Boot Version
4. Project Metadata: Group, artifacts, Java version etc
5. **Dependencies**

Follows a maven project structure
`src/main/Java` -> Contains the main Java code. Created from the metadata provided initially in spring initializer and the main Application Class

`src/main/resources` -> Contains static files and templates + application.properties

`src/test` -> Package mirroring of `main` 

`.mvn`, `mvnw`, `mvnw.cmd` -> maven wrappers. Essentially no need to have maven installed to run the application

`pom.xml` -> All of the dependencies are declared (Picked in the init). Also called spring-boot starters

![[Pasted image 20250608171504.png]]
^ There is a maven tab on the right side which shows all of the commands we can use for each plugin

##### Why are java packages usually named like `dev.xyz.name` or to generalize it further: `com.organisation.mypackage` or `com.org.region.package`?
Short answer: 
	To avoid namespace conflicts during imports. It uses a reverse domain name naming convention

Long Answer:
	It's that way because package names are meant to be globally unique. That way, when you import a package, there are no conflicts. See, for example, the languages which came just before it and inspired it: C and C++.
	---
	There, if you do something like `#include xyz.h`, you either use the `<>` syntax or `""` syntax, but in either case, it's hard to know exactly which header you picked up without actually trying to compile (things like environment variables and the compiler-installer's idiosyncracies will factor in). Plus, there's nothing stopping two different vendors from making the same file called `xyz.h`.
	---
	So, in Java, they thought about this problem, and decided package names should be given the best possible chance to be globally unique. So, a reverse-domain name naming convention was adopted, because now two vendors are unlikely to collide, and within an organization, you can generally work it out (or have your naming dispute refereed by managers or tech leads or ping pong death matches).
	---
	It's a reverse DNS name convention, chosen so that packages from different vendors can be unique, and thus avoiding the whole: "Wait...which `AbstractFactoryInstanceProviderSingletonInterface` class is this one? Oh...It's the one from `com.reddit.`"

#### Setting up a logger:
![[Pasted image 20250608174753.png]]
Initializing the logger automatically provides a pop up to add the necessary imports

**We will be packaging our code by features**

#### What is a Record Java class?
Record vs Class: 
- In a class, we need to setup explicit getters and setters for each of the class attributes but it is not required in a Record (?).
- A record is immutable.
