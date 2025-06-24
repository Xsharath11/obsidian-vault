#### Spring MVC:
- Model: The data
- View: How are we going to represent the models?
- Controller: Takes in a request and figures out what we will do with it and return a response

Spring is an inversion of control framework.
What this means:
- We will almost never need to use the `new` keyword to initialize new instances of classes since spring will already be maintaining an instance by itself.
- we need to use dependency injection to use the instance being maintained by spring itself
- For this we can use the wrappers `@repository` 

![[Pasted image 20250609135614.png]]
^ Wrong way to initialize the controller which has an instance of the `RunRepository`

What is the `RunRepository`?
- It is just a class that contains all of the Runs.
- It is wrong to maintain the data of all of the runs within the controller itself. Violates Dependency Injection (?)
- It also contains methods pertaining to the runs
- A controller is very dumb, It just takes in a request and figures out what to do with it. i.e just routing. The actual logic is offloaded to other classes. A `RunRepository` in this case 

![[Pasted image 20250609140132.png]]
^ `RunRepository`

Basically, doing it this way would initialize a `RunRepository` instance every time the controller is hit which is bad

#### Right way to do it:
- Wrap the `RunRepository` class with the `@Repository` wrapper
- Pass the `RunRepository` instance as a dependency in the constructor to the controller class 
- ![[Pasted image 20250609141047.png]]

#### Dynamically fetching ids from the url to return runs from the repository:

![[Pasted image 20250609172038.png]]
- We use `@PathVariable`  to fetch the id from the endpoint
`RunRepository.java`:
```
Optional<Run> findById(Integer id) {  
    return runs.stream()  
            .filter(run -> run.id() == id)  
            .findFirst();  
}
```
^ Method in `RunRepository` class which fetches runs based on the ID, if the ID exists. Else, returns `null`

![[Pasted image 20250609172756.png]]
^ Handling cases where `null` is returned from the `RunRepository` and raising a `404 Not Found Error`

