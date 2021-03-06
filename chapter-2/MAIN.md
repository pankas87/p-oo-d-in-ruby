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

+ When exposing variables as methods, we must choose if said data will be hidden or exposed to other object in the application. Hiding it is as simple as declaring the accessors as private methods.

+ Distinction between data and regular object begins to disappear. While it's expedient to think of parts of your application as behavior-less data, most things are better thought of as plain old objects.

You should hide data from yourself, doing so protects the code from being affected by unexpected changes. Data very often has behavior that you don't yet know about. Send messages to access variables even if you think of them as data.

### Hide data structures

If being attached to an instance variable is bad, depending on a complicated data structure is worse. Consider the following *ObscuringReference* class.

````(ruby)
class ObscuringReference
  attr_reader :data

  def initialize(data)
    @data = data
  end

  def diameters
    # 0 is rim, 1 is diameter
    data.collect { |cell| cell[0] + (cell[1] * 2) }
  end  
end
````

````(ruby)
# Rim and tire sizes in a 2d array
@data = [[622,20], [622,25], [559,50], [559,40]]
````

Since @data contains a complicated data structure, just wrapping it with an accessor method is not enough, the data method returns the array, senders of data must have complete knowledge of what piece of data is at which index in the array.

The *diameters* method must know where to find rim and tires in the array, it depends heavily on the structure of the array, if that structure changes then this code must change.

References are leaky, escape encapsulation, and the knowledge of rims and tires location are repeated troughtout the code, they're no dry.

Each change represents an opportunity to create a bug so stealthy that your attempts to find it will make you cry.

Direct references into complicated structures are confusing, because they obscure what the data really is, and they're a maintenance nightmare, every reference will need to be changed when the structure of the array changes.

The *RevelaingReferences* class has a similar interface, it takes a two-dimensional array as an initialization argument and it implements the *diameters* method, despite this the internal implementation is very different.

````(ruby)
class RevealingReferences
  attr_reader :wheels

  def initialize(data)
    @data = wheelify(data)
  end

  def diameters
    wheels.collect { |wheel| wheel.rim + (wheel.tire * 2) }
  end
  # Now everyone can send rim/tire to wheel  
end

Wheel = Struct.new(:rim, :tire)

def wheelify(data)
  data.collect { |cell| Wheel.new(cell[0], cell[1]) }
end
````

All knowledge of the array structure is insolated in the *wheelify* method, that converts the array of arrays into an array of structs.

The *wheelify* method contains the only bit of code that understands the structure of the incoming array, if the input changes the code will change in just this one place.

### Enforce single responsibility everywhere

#### Extract extra responsibilities from methods

The idea of single responsibility can be usefully employed in many other parts of your code.

Methods, like classes, should have a single responsibility and for the same reasons. The same design techinques work:

+ Ask them questions about what they do
+ Try to describe their responsibilities in a single sentence

Look at the *diameters* method in *RevealingReferences*

````(ruby)
def diameters
  wheels.collect { |wheel| wheel.rim + (wheel.tire * 2) }
end
````

This method clearly has two responsibilities, iterating over wheels AND calculating the diameter of each wheel.

Simplify the code by separating it into two methods, each with one responsibility.

````(ruby)
def diameters
  wheels.collect { |wheel| diameter(wheel) }
end

def diameter(wheel)
  wheel.rim + (wheel.tire * 2)
end
````

Do these refactoring even when yo don't know the ultimate design. Theyre needed not because the design is clear but because it isn't. Yo do not have to know where you're going to use good design practices to get there. Godd practices reveal design.

The impact of a single refactoring like this is small but the cumulative effect of this coding style is huge. Methods that have a single responsibility confer the following benefits:

+ *Expose previously hidden qualities*: Having each method serve a single purpose makes the set of things the class does more obvious.

+ *Avoid the need for comments*: If a bit of code in a method needs a comment, extract that bit into a separate method whose new name will serve the same purpose as did the old comment.

+ *Encourage reuse*: Other programmers will reuse the methods because they just do one decoupled thing and nothing more, instead of duplicating the code. They will follow the pattern you have established and create small, reusable methods in turn. This coding style *should* propagate itself.

+ *Are easy to move to other classes*: When you get more design information and decide to make changes, small methods are easy to move. You can rearrange behavior without doing a lot of method extraction and refactoring. Small methods lower the barrier to improving your design.

#### Isolate extra responsibilities in classes

Once every method has a single responsibility the scope of your class will be more apparent.

The *Gear* class has some wheel-like behavior. Does this application ned a *Wheel* class?

It may seem impossible for *Gear* to have a single responsibility unless you remove its wheel-like behavior; the extra behavior is either in *Gear* or it's not but casting the design choice in either/or terms is shortsighted. Your goal is to preserve single responsibility in *Gear* while making the fewest design commitments possible.
It may seem impossible for *Gear* to have a single responsibility unless you remove its wheel-like behavior; the extra behavior is either in *Gear* or it's not but casting the design choice in either/or terms is shortsighted. Your goal is to preserve single responsibility in *Gear* while making the fewest design commitments possible.

Changing changeable code means postponing decisions until you are absolutely forced to make them. Any decision in advance of a requirement is just a guess.

**Don't decide; preserve your ability to make a decision later**

In the next example the *Wheel* struct is extended with a block that adds a *diameter* method.

````(ruby)
class Gear
  attr_reader :chainring, :cog, :wheel

  def initialize(chainring, cog, rim, tire)
    @chainring = chainring
    @cog = cog
    @wheel = Wheel.new(rim, tire)
  end

  def ratio
    chainring / cog.to_f
  end

  def gear_inches
    ratio * wheel.diameter
  end

  Wheel = Struct.new(:rim, :tire) do
    def diameter
      rim + (tire * 2)
    end
  end
end
````

Now you have a *Wheel* that can calculate its own diameter. Embedding the *Wheel* in *Gear* is not a long term design decision, it's just an experiment in code organization that cleans up *Gear* but defers the decision about *Wheel*

If you have a muddled class with too many responsibilities, separate those responsibilities into different classes. If there are extra responsibilities that cannot yet be removed, isolate them.

Do not allow extraneous responsibilities to leak into your class.

### Finally, the real wheel

Finally the future arrives, you show your calculator to a cyclist friend that tells you that she would like to calculate "bicycle wheel circumference". She has a computer on her bike that claculates speed and must be configured with the wheel's diameter.

This information allows us to make the next design decision, now our application has an explicit need for a *Wheel* class. It's time to set *Wheel* free to be a separate class of its own.

Because *Wheel* behavior is carefully isolated inside of *Gear* the change is painless, just convert the *Wheel* struct to an independent class.

````(ruby)
class Gear
  attr_reader :chainring, :cog, :wheel

  def initialize(chainring, cog, wheel = nill)
    @chainring = chainring
    @cog = cog
    @wheel = wheel
  end

  def ratio
    chainring / cog.to_f
  end

  def gear_inches
    ratio * wheel.diameter
  end
end

class Wheel
  attr_reader :rim, :tire

  def initialize(rim, tire)
    @rim = riim
    @tire = tire
  end

  def diameter
    rim + (tire + 2)
  end

  def circumference
    diameter * Math::PI
  end
end

@wheel = Wheel.new(26, 12.5)

puts @wheel.circumference
# -> 91.106186186954104

puts Gear.new(52, 11, wheel).gear_inches
# -> 137.090909

puts Gear.new(52, 11).ration
# -> 4.727272
````

Both classses have a single responsibility. The code is not perfect, but in some ways it achieves a higher standard: it is *good enough*

## Summary

The path to changeable and maintainable object-oriented software begins with classes that have a single responsibility. Classes that do one thing isolate that thing from the rest of your application. This isolation allows change without consequence and reuse without duplication.
