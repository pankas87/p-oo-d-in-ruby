# Chapter 4.

An object oritented application is more than just its classes.  It is made up of classes but defined by messages. Classes control what's in your source code repository and message reflect the living, animated application.

Design deals with:

+ What objects know (Their *responsibilities*)
+ Who they know (Their *dependencies*)
+ How they talk to one another (Their *messages*)

The conversation between objects takes place using their *interfaces*.

## Understanding Interfaces

Imagine two applications consisting of objects and messages passing between them.

![Communication patterns](4-1-communication-patterns.jpg)

In the first application we cannot identify no apparent pattern. Every object may send a message to any other object. The trails left by messages look like a mat.

In the second application we can spot a clear pattern. The objects communicate in specific and well defined ways. The trails left by the messages look like islands occasionally connected by bridged between them.

Applications are characterized by the pattern of their messages.

Objects in the first application are hard to reuse, they expose too much of themselves and know too much about their neighbors. No object stands alone, to reuse any you need all, to change one thing you must change everything.

The second application is made up of pluggable component-like objects. Each reveals as little of itself and knows as little about others as possible.

We can have classes with a single responsibility that use dependency injection and still end up with a mess like the one in the first application. Those two techniques alone are not enough to prevent the construction of a poorly designed application. The root of this new problems lies not in what the classes do but in what they reveal.

All methods in a class are not the same, some are more general or more likely to change than others.

In a well designed application, like the second one, the message patterns are visibly constrained, there are limits to what messages can be passed between objects.

The exposed methods comprise the class's *public interface*.

By interface we are referring to the kind of interface that is within a class: Classes implement methods, some of those methods are meant to be used by others and these methods make up its public interface.

There is another type of interface, one that spans across classes. In this sense, the word *interface* represents a set of messages where the messages themselves define the interface. It's almost as if the interface defines a *virtual* class; that is, any class that implements the required methods can act like the *interface* kind of thing.

## Defining Interfaces

Imagine a restaurant kitchen, customers order food off a menu, the orders come into the kitchen through a little window and the food comes out.

Customers don't need to know how the food is made or to decide what food can be cooked based on the current inventory of the kitchen. They interact with the kitchen using the menu, the kitchen's *public* interface, and inside the kitchen a lot of messages get passed between the workers, these messages are the *private* interface of the kitchen, invisible to the customer.

The customers can ask for *what* they want without knowing *how* it's made.

Our classes are like a kitchen, they fulfill a single responsibility but implement many methods. Methods vary in scale and granularity, some are broad general methods exposing the main responsibility of the class, other ones are tiny utility methods not meant to be used externally.

### Public Interfaces

Methods that make up the public interface of the class, the face it shows to the world:

+ Reveal its primary responsibility
+ Are expected to be invoked by others
+ Will not change on a whim
+ Are safe for others to depend on
+ Are thoroughly documented in the tests

### Private Interfaces

All other methods in the class are part of its private interface. They:

+ Handle implementation details
+ Are not expected to be sent by other objects
+ Can change for any reason whatsoever
+ Are unsafe for others to depend on
+ May not even be referenced on the tests

### Responsibilities, Dependencies and Interfaces

There is a correspondence between the statements you might make about these more specific responsibilities and the classes' public methods.Public methods should read like a description of the responsibilities of the class; a public interface is a contract that articulates the responsibilities of your class.

The public parts of a class are the stable parts; the private parts are the changeable parts. Marking a method as public or private tells users of the class upon which method they may safely depend.

Public methods shold be stable.

### Finding the Public Interface

Finding and defining public interfaces is an art. It is a design challenge that doesn't have a cookie-cutter solution. There are many ways to create a *good enough* interface and the costs of a *not  good enough* interface are not obvious at first so it may be difficult to learn from mistakes.

The design goal will always be to retain the maximum possible flexibility in the future while doing just enough effort to meet today's requirements.

Good public interfaces reduce the cost of unexpected change, bad public interfaces raise it.

#### An Example Application: Bicycle touring company

FastFeet Inc is a bicycle touring company that offers road and mountain bike trips. It runs its business using a paper system with no automation at all.

Each trip follows a specific route and may occur several times during the year. Each has limitations on the number of customers who may go and requires a specific number of guides who double as mechanics.

Each route is rated according to its aerobic difficulty, mountain bikes have an additional rating regarding technical difficulty.

Customers have an aerobic fitness level and a mountain bike technical skill level to determine if a trip is right for them.

Customers may rent bicycles or they may bring their own. FastFeet has some bicycles available and shares a pool of bicycles with local bike shops. Rental bikes come in various sizes and are suitable for either road or mountain trips.

Consider the following simple requirements, which will be referred to later as as a *use case*: A customer in order to choose a trip, would like to see a list of available trips by appropriate difficulty, on a specific date, where rental bicycles are available.

##### Constructing an Intention

Getting started with the first bit of code of a brand new application is intimidating, you must make decisions that will determine the patterns of this application forever. The design that gets extended later is the one that you are establishing now.

You should not dive in and write code. Some experts advocate writing the tests first, but it can be difficult for novice developers imagining the first test.

Test-first gurus can easily start writing tests because they have so much design experience. At the first stage they have already constructed a mental map of possibilities for objects and interactions. They're not attached to any specific idea and use the tests to discover alternatives, but they know so much about design that they have already formed an intention about the application. It is this intention that allows them to specify the first test.

The description of Fast Feet has probably give you some idea about possible classes like Customer, Trip, Route, Bike, Mechanic...

These classes spring to mind because they represent *nouns* in the application of things that have both *behavior* and *data*. They are the domain objects of our application and probably will end up with a representation in the database.

These domain objects are a trap, because if we fixate on them we will coerce unnatural behavior into them. Design experts *notice* domain objects without concentrating on them; they focus not on these objects but on the messages that pass between them. These messages are guides that lead you to discover other objects, ones that are just as necessary but far less obvious.

First than anything you should form an intention about the objects *and* the messages needed to satisfy this use case. One simple and inexpensive way of doing this without writing code is using UML sequence diagrams.

##### Using Sequence Diagrams

UML sequence diagrams enable us to experiment with objects and messages in a low-cost manner.

Sequence diagrams are quite handy. They provide a simple way to experiment with different object arrangements and message passing schemes. They bring clarity to your thoughts and provide a vehicle to collaborate and communicate with others. They're a lightweight to acquire an intention about an interaction. Draw them on a blackboard, alter them as needed and erase them when they've served their purpose.

Figure 4.3 shows a simple sequence diagram representing the implementation of the use case mentioned before. Moe a *Customer* sends the *suitable_trips* message to the *Trip* class an gets back a response.

![Sequence diagram](figure-4-3.png)

This sequence diagram can be read as follows: *Customer* Moe sends the *suitable_trips* message to the *Trip* class.

The diagram is almost an exact literal translation of the use case. The nouns in the use case became objects in the sequence diagram and the action of the use case turned into a message. The message requires three parameters: *on_date*, *of_difficulty*, and *need_bike*.

Moe expects the *Trip* class to find him a suitable trip. It seems reasonable that *Trip* would be responsible for finding trips on a date and of a difficulty, but Moe may also need a bicycle and he clearly expects *Trip* to handle that too.

Drawing the diagram exposes the message passing between classes and prompts you to ask the question: "Should *Trip* be responsible  for figuring out if an appropriate bicycle is available for each suitable trip?" or more generally, "Should this receiver be responsible for responding to this message?"

Sequence diagrams explicitly specify the messages that pass between objects, and because objects should only communicate with public messages, they are a great tool for exposing, experimenting and defining public interfaces.

Suddenly the conversation has changed, rather tan talk about classes we're talking about messages. Instead of deciding on a class and figuring out its responsibilities, we're now deciding on a message and figuring out where to send it.

This transition from class-based design to message-based design is a turning point in your design career. The message-based perspective yields more flexible application than does the class based perspective. Changing the fundamental design question from "I know I need this class, what should it do?" to "I need to send this messagem who should respond to it?" is the first step in that direction.

** You don't send messages because you have objects, you have objects because you send messages **

It's perfectly reasonable for a *Customer* to send the suitable_trips message but *Trip* should not receive it.

Now that you hace the *suitable_trips* message in mind but no place to send it, you must construct some alternatives.

Sequence diagrams make it easy to explore the possibilities.

Perhaps there should be a *Bicycle* that should figure out if bicycles are available. *Trip* should be responsible for *suitable_trips* and *Bicycle* for *suitable_bicycle*. Moe can get the answer he needs if he talks to both of them, like in figure 4.4

![Sequence diagram](figure-4-4.png)

Figure 4.4 is an improvement in some areas but a failure in others. This design removes extraneous responsibilities from *Trip* but transfers them to *Customer*.

Moe must not only know *what* he wants it but now he must know *how* other objects should collaborate to provide it.

Going back to the kitchen metaphor, Moe isn't ordering from a menu, he's going into the kitchen and cooking. *Customer* is taking responsibilities that belong somewhere else and is binding itself to an implementation that might change.

##### Asking for what instead of how

There is a subtle, yet important, distinction between a message that that asks for what the sender wants and a message that tells the receiver how to behave. Understanding this difference is a key part of creating reusable class with well-defined public interfaces.

To understand the difference between *what* vs *how* let's put the customer and trip design problem to rest for a while.

![Sequence diagram](figure-4-5.png)

In figure 4.5 a trip is about to depart and it needs to make sure all the bicycles scheduled to be used are in good shape. The use case is: A trip, in order to start, needs to ensure that all its bicycles are mechanically sound. *Trip* could know exactly how to make a bike ready for a trip and could ask a *Mechanic* to do each of those things.
