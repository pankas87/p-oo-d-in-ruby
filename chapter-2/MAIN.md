# Chapter 2. Designing Classes with a Single Responsibility

The foundation of an object-oriented systema is the *message*, the most visible organizational structure is the *class*.

Some overwhelming questions:

+ What are your classes?
+ How many should you have?
+ What behavior will they implement?
+ How much do they know about other classes?
+ How much of themselves should they expose?

At the initial stage your obligation is to take a deep breath and *insist that it be simple*. Your goal is to model your application, using classes, such that it does what is supposed to do *right now* and it's easy to change *later*.

Your appliction needs to work right now just once; it must be easy to change forever.

## Deciding What Belongs in a Class

When we have an application in our mind we usually have a clear idea of the implementation; we do not have a technical problem but rather an organizational problem; we know how to write the code but we don't know where to put it.

### Grouping Methods into Classes

The classes you create will affect how you think about your application forever. The classes are a virtual world that sets constraints for our imagination; they create a box that may be difficult to think outside of.

You will never know less than you know right now. The ability to successfully make changes is determined by the application's design.

Design is more about preserving changeability than about perfection.

#### Organizing Code to Allow for Easy Changes

*Easy to change* can be defined as:

+ Changes have no unexpected side effects
+ Small changes in requirements require correspondingly small changes in code
+ Existing code is easy to reuse
+ The easiest way to make a change is to add code that in itself is easy to change

Therefore, code should be:

+ **Transparent**: The consequences of change should be obvious in the code that is changing and in distant code that relies upon it

+ **Reasonable**: The cost of any change should be proportional to the benefits the change achieves

+ **Usable**: Existing code should be usable in new and unexpected contexts

+ **Exemplary**: The code itself should encourage those who change it to perpetuate these qualities

*T*ransparent, *R*easonable, *U*sable, *E*xemplary (TRUE) code meets today's needs and can be changed to meet the needs of tomorrow.

To ensure that the code we write is *TRUE* we must first ensure that each class has a single well-defined responsibility.

## Creating Classes that Have a Single Responsibility
