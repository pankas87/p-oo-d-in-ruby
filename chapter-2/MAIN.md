# Chapter 2. Designing Classes with a Single Responsibility

The foundation of an object-oriented system is the *message*, the most visible organizational structure is the *class*.

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

A class should do the smallest possible thing: single responsibility.

### An example appliction: Bycicles and gears

````(ruby)
# number of teeth
chainring = 52
cog = 11
ratio = chainring / cog.t_f
puts ratio # 4.7272

chainring = 30
cog = 27
ratio = chainring / cog.t_f
puts ratio # 1.11
````

We'll write a application to calculate gearing ratio, made of up Ruby classes, each representing some part of the domain.

Nouns that represent objects of the domain, like *bycicle* and *gear* are candidates to be classes.

Nothing in the description shows any behavior for the *bycicle* class so it does not qualify. On the other hand a *gear* has both data and behavior: chainrings, cogs and ratios. With the behavior from the previous script we can model a simple Gear class.


````(ruby)
class Gear
  attr_reader :chainring, :cog

  def initialize(chainring, cog)
    @chainring  = chainring
    @cog        = cog
  end

  def ratio
    chainring / cog.to_f
  end
end

puts Gear.new(52,11).ratio # 4.7272
puts Gear.new(30,27).ratio # 1.11
````

Now we calculate gear inches, defined as follows:

gear inches = wheel diameter * gear ratio

where

wheel diameter = rim diameter + twice tire diameter

````(ruby)
class Gear
  attr_reader :chainring, :cog, :rim, :tire

  def initialize(chainring, cog, rim, tire)
    @chainring  = chainring
    @cog        = cog
    @rim        = rim
    @tire       = tire
  end

  def ratio
    chainring / cog.to_f
  end

  def gear_inches
    ratio * (rim * (tire * 2))
  end
end

puts Gear.new(52, 11, 26, 1.5).gear_inches #137.09
puts Gear.new(52, 11, 24, 1.25).gear_inches #137.09
````

The application meets the specification but has one bug

````(ruby)
# Didn't this used to work
puts Gear.new(52,11).ratio # 4.7272

# ArgumentError: wrong number of arguments (2 for 4)
````

Altering the number of arguments that a method requires breaks all callers of a method. This is a terrible problem on large applications.

With a rudimentary *Gear* class it's time to ask yourself: Is this the best way to organize the code.

As usual, it depends, if the application will remaain in its current form *Gear* is good enough. However, for a more complex calculator *Gear* is the first of many classes of an application that will evolve. *To efficiently evolve, code must be easy to change*.

### Why single responsibility matters

Applications that are easy to use consist of classes that are easy to reuse, pluggable units of well defined behavior with few entanglements.

An application that is easy to use is like a box of building blocks; you can select just the pieces you need and assemble them in unanticipated ways.

Class with many responsibilities are hard to reuse, the various responisbilities are entangled and you can just get the pieces you need. You end up either duplicating the code, which leads to increased bugs and maintenance costs; or add conditional execution flow which creates a class that is confused about what it does, any change to this class has a big possibility of breaking things around the application with any change.

You increase your application's chance of braking unexpectedly if you depend on classes that do too much.

### Determining if a class has a single responsibility

A good way to determine if a class has behavior that belongs somewhere else is to pretend that it is sentient and interrogate. Rephrase every one of its methods as a question.

For example, asking the Gear class *"What is your ratio?"* seems perfectly reasonable, *"Please Mr Gear, what are your gear_inches?"* is on shaky ground and *"Please Mr Gear, what is your tire (size)?"* is just downright ridiculous.

Another way to check for single responsibility is to describe the class using one sentence. If the simplest description uses the word "and", the class has more than one responsibility. If you use the word "or", it has two responsibilities and they're probably not very related.

OO designers use the word *cohesion* to describe this concept. When everything in a class is related to its central purpose, the class is said to be highly cohersive.

### Determining when to make design decisions

There is always a tension between *improve it now* and *improve it later*. Applications are never perfectly designed. Every choice has a price. A good designer understands this tension and minimizes cost by making informed tradeoffs between the needs of the present and the possibilities of the future.

## Writing code that embraces change

Classes can be arranged so that they'll be easy to change even if you don't know what changes will come.

Change is inevitable, hence coding in a changeable style has big future payoffs and as an additional bonus, coding in these styles improve your code today, at no extra cost.

### Depend on behavior, not data

Behavior is captures in methods and invoked by sending messages. In classes with a single responsibility behavior lives in one and only one method (DRY principle, Don't Repeat Yourself).

DRY tolerates change because any change can be made by changing code in only one place.

### Hide instance variables

Wrap instance variables with accessor methods instead of referring them directly, like the ratio method does below.

````(ruby)
class Gear
  def initilize(chainring, cog)
    @chainring = chainring
    @cog = cog
  end

  def ratio
    # road to ruin
    @chainring / @cog.to_f
  end
end
````

Hide the variables even from the class that defines them. The initial getter methods can be implemented using the attr_reader keyword provided by Ruby.

````(ruby)
class Gear
  def initialize(chainring, cog)
  end

  def ratio
    chainring / cog.to_f
  end

  def chainring
    @chainring
  end

  def cog
    @cog
  end
end
````

This cog method is now the only place in the code that understands what cog means. Cog becomes the result of message sent. This implementation changes cog from data referenced everywhere to behavior that is defined just once.

If the @cog instance variable is referred to ten times and needs to be adjusted, it has to be done in only one place.

````(ruby)
# A simple implementation
def cog
  @cog * unanticipated_adjustment_factor
end
````

````(ruby)
# A more complex implementation
def cog
  @cog * (foo? ? bar_adjustment : baz_adjustment)
end
````

The second adjustment is a simple behaviour change when done in a method but a code destroying mess when applied to a bunch of instance variable references.

Dealing with data as objects that understand messages creates two issues:

+ When exposing variables as methods, we must choose if said data will be hidden or exposed to other object in the application. Hiding it is as simple as declaring the accesors as private methods.

+ Distinction between data and regular object begins to disappear. While it's expedient to think of parts of your application as behavior-less data, most things are better thought of as plain old objects.

You should hide data from yourself, doing so protects the code from being affected by unexpected changes. Data very often has behavior that you don't yet know about. Send messages to access variables even if you think of them as data.

### Hide data structures
