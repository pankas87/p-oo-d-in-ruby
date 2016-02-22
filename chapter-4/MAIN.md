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
