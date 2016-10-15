# Sharing Role Behavior with Modules

Sometimes we need to share common behavior across classes that don't share an abstraction/specialization relationship. For example, FastFeet could need a recumbent mountain bike.

There are times when inheritance makes sense, and time when it does not. You must understand when to use it and when not to use it. Use of classical inheritance is always optional; every problem that it solves can be solved in other way.

No design is free, we must make design decisions by doing tradeoffs between relative costs and likely benefits of the alternatives.

## Understanding roles

Sometimes we need to share behavior among unrelated classes that don't respond to the sperclass/subclass requirement of classical inheritance, this relation between classes is a role that the class plays.

Unrelated objects that play a role enter into a relationship with the objects they play the role for. These kind of relationships are not as visible as those created by classical inheritance, and of course, this relationship introduces costs that must be take into account when deciding among design options.

### Finding roles

The *Preparer* duck-type from Chapter 5 is a role, played by objects that implement that interface. *Mechanic*, *TripCoordinator* and *Driver* are *preparers* because they implement the *prepare_trip* public method, objects can interact with them and treat them like a preparer without concerns for their class.

Roles usually come in pairs, so the existence of a *Preparer* role suggest the existence of *Preparable*, another role, in Chapter 5, *Trip* acted as a *Preparable* whose interface implements all the messages that any *Preparer* might expect to send to a *Preparable*.

*Preparable* is a simple role, to play it an objects simply needs to implement the *prepare_trip* method. *Preparable* objects only share the method signature but no other code.

There are more complex roles where not only a method signature is shared but also specific behavior via code. We need to organize our code so we define the behavior in one place but makie it usable by any object that acts as a *Preparable*.

In Ruby we have *modules*, a mechanism provided by the language in order to define a named group of methods that can be mixed into an object. This mechanism exists in many OOP languages, and it's called mixins.

Methods can be defined in one module and then this module can be added to any object, this allows objects of different classes to play a common role using shared code.

**When an object includes a module, the methods defined therein become available via automatic delegation**, it is a similar mechanism to the one used by classical inheritance, when an object receives a message it does not understand it gets automatically routed somewhere else.

By adding modules to our codebase, and including those modules in our objects we are entering a new realm of design complexity. The total set of messages to which an object can respond to includes:

+ Those it implements
+ Those implemented in all objects above it in the hierarchy
+ Those implemented i any module that has been added to it
+ Those implemented in all modules added to any objects above it in the hierarchy

### Organizing Responsibilities
