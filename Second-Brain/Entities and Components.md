Game developers have moved away from an OOP based design mostly because it starts to get quite confusing once we start moving past the initial scope of the game and we may end up recreating whole classes with the same logic which have been defined in other classes:

For example:
Let us take a class `MeleeMob` and a class `ArcherMob`.
If we need to create an NPC which can use ranged as well as melee attacks, we will need a create a whole new class with logic combined from both the previous classes.

Next, suppose, down the line, the NPC needs to turn friendly, that poses new problems

Here comes in the Idea of an Entity Component based design

#### Entity Component Based Design
- An **Entity** is a thing. Anything. It is basically an identification number for the type of mob it is at a very high level
- To each entity, we can add **Components**. 
- Components are just data grouped by whatever properties we want our entities to have.
Example:
We can build the same set of mobs listed above with components for:
- Position
- Renderable
- Hostile
- MeleeAI