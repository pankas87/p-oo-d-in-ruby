# Chapter 3. Managing Dependencies

Objects reflect qualities of a real-world problem and the interactions between those objects provide solutions. If we follow the single responsibility principle, an object cannot know everything, inevitably it will have to talk to another object.

All of the behavior in an application, invoked via messages, is dispersed among objects. Therefore, for any desired behavior, an object either knows it personally, inherits it, or knows another object who knows it.

The very nature of objects, requires that they collaborate to accomplish complex tasks. This collaboration is powerful and perilous; objects must know something about others, this creates a dependency that could strangle your application if not managed carefully.

## Understanding dependencies

An object depends on another object if, when one object changes, the other might be forced to change in turn.

````(ruby)
class Gear
  attr_reader :chainring, :cog, :rim, :tire

  def initialize(chainring, cog, rim, tire)
    @chainring = chainring
    @cog = cog
    @rim = rim
    @tire = tire
  end

  def gear_inches
    ratio * Wheel.new(rim, tired).diameter
  end

  def ratio
    chainring / cog.to_f
  end
end

class Wheel
  attr_reader :rim, :tire

  def initialize(rim, tire)
    @rim = rim
    @tire = tire
  end

  def diameter
    rim + (tire * 2)
  end
end

Gear.new(52, 11, 26, 1.5).gear_inches
````

*Gear* has at least four dependencies in *Wheel*

### Recognizing dependencies

+ The name of another class. *Gear* expects a class named *Wheel* to exist.
+ The name of a message that intends to send to someone other than *self*. *Gear* expects a *Wheel* instance to respond to *diameter*.
+ The argument that a message requires. *Gear* knows that *Wheel.new* requires a *rim* and a *tire*.
+ The order of those arguments.

Each dependency creates a chance that *Gear* will be forced to change because of a change in *Wheel*. Some degree of dependency is inevitable, after all, these classes must collaborate, but most are unnecesary.

Dependencies make code less reasonable, they turn minor tweaks into major undertakings where small changes cascade through the application.

The challenge is to manage dependencies so that each class has the fewest possible. A class should know just enough to do its job and not one thing more.

### Coupling Between Objects (CBO)

These dependencies couple *Gear* to *Wheel*.

Each *coupling* creates a *dependency*. The more one object knows about the other, the more tightly coupled they are. The more tightly coupled they are, the more they behave like a single entity.

Objects that behave like a single entity are harder to use independently, to reuse and to test.

When two (or three or more) objects are so tightly coupled that they behave as a unit, it's impossible to reuse just one. Changes to one object force changes to all. Left unchecked, unmanaged dependencies cause an entire application to become an entangled mess.

A day will come when it's easier to rewrite everything than to change anything.

#### Other Dependencies

The four dependencies named before will discussed in the remainder of this chapter but it is worth mentioning a few other common dependency related issues.

One specially destructive kind of dependency is when one object knows another, who knows another, who knows something. Many messages are chained together to reach behavior that lives in a distant object. Message chaining creates a dependency between the original object and every object and message along the way.

This coupling increases the change that the original object has to change because of a change in any of the intermediate objects.

Test dependencies on code can also exist and are very costly. Developers that are new to testing write test code that is too tightly coupled to the code they're testing. This coupling leads to frustration, test break every time the code is refactored, even when the fundamental behavior of the code does not change.

### Writing Loosely Coupled Code

#### Inject Dependencies

Referring directly to a class's name and/or instantiating it inside another class creates a hard-coded reference that makes our class dependent on, and only able to work with objects of the referenced class.

In the *Gear* example, the class is only willing to collaborate with an object of the *Wheel* class. If the application expands to include objects such as disks or cylinders and you need to know the gear inches of gears which use them, *you cannot*.

Code should not be attached to static types without justification, it is not the class of the object that's important, it's the *message* you plan to send to it (Duck typing, interfacing in statically typed languages).

*Gear* needs access to an object that responds to the *diameter* method, regardless of its class. It doesn't need to know that a *Wheel* needs a *rim* a *tire* to be initialized; it just needs an object that knows *diameter*.

Unnecessary dependencies reduce a class *reusability* and increases the susceptibility of being forced to change unnecessarily. It becomes less useful when it knows too much about *other objects*; by knowing less, classes can do more.

For example, this next version of wheel expects to be initialized with an object that can respond to diameter.

````(ruby)
class Gear
  attr_reader :chainring, :cog, :wheel

  def initialize(chainring, cog, wheel)
    @chainring = chainring
    @cog = cog
    @wheel = wheel    
  end

  def gear_inches
    ratio * wheel.diameter
  end

  def ratio
    chainring / cog.to_f
  end
end

# Gear expects a 'Duck' that knows diameter
Gear.new(52,11,Wheel.new(26,1.5)).gear_inches
````

This change is so small it is almost invisible, but coding in this style has huge benefits. Moving the creation of the new *Wheel* outside of the *Gear* class decouples the two classes. *Gear* can now collaborate with any class that implements *diameter*. The benefit was free, not a single line of code was added, code was only rearranged.

This technique is known as *dependency injection*.

#### Isolate dependencies

Sometimes dependencies cannot be removed completely, specially when working on existing applications with severe constraints on what can be changed. In those cases where perfection cannot be achieved the goal should be to improve the overall situation by leaving the code better than what we found it.

If we cannot remove unnecessary dependencies we should try and isolate them.

Dependencies are alien bacterium trying to infect our class, therefore we should provide our class with a strong immune system; quarantining each dependency.

##### Isolate instance creation

In the *Gear* example, if constraints prevent us from injecting a *Wheel* into a *Gear* at least we should isolate the *Wheel* instantiation so it happens once in only one isolated place.

The next two examples illustrate the idea.

````(ruby)
class Gear
  attr_reader :chainring, :cog, :rim, :tire

  def initialize(chainring, cog, rim, tire)
    @chainring = chainring
    @cog = cog
    @wheel = Wheel.new(rim, tire)
  end

  def gear_inches
    ratio * wheel.diameter
  end

  def ratio
    chainring / cog.to_f
  end
end
````

````(ruby)
class Gear
  attr_reader :chainring, :cog, :rim, :tire

  def initialize(chainring, cog, rim, tire)
    @chainring = chainring
    @cog = cog
    @rim = rim
    @tire = tire    
  end

  def gear_inches
    ratio * wheel.diameter
  end

  def ratio
    chainring / cog.to_f
  end

  def wheel
    @wheel ||= Wheel.new(rim, tire)
  end
end
````

The *Gear* class still knows too much, it still takes rim and tire as arguments and still creates its own new instance of wheel.

*Gear* is still stuck to *Wheel*, it cannot calculate the gear_inches of any other object, but an improvement has been made. This coding style reduces the number of dependencies in *gear_inches* while publicly exposing *Gear*'s dependency on wheel.

The changes reveal dependencies instead of concealing them, making the code easier to changes when circumstances allow it.

#### Isolate vulnerable external messages

External messages, sent to someone other than self, that are susceptible to change should be wrapped (encapsulated) inside methods, by doing this we remove the dependency to the method name and arguments.

Let's take a look at the refactored *gear_inches* method below. In the original version *gear_inches* had to know that there was a *wheel* object that responded to a *diameter* message, and could even have some parameters. Now we have decoupled *gear_inches* from the calling of *wheel.diameter*, this way if any of the object's name, method name or method arguments change, we will only need to change the *diameter* wrapper method.

````(ruby)
class Gear
  attr_reader :chainring, :cog, :rim, :tire

  def initialize(chainring, cog, rim, tire)
    @chainring = chainring
    @cog = cog
    @rim = rim
    @tire = tire    
  end

  def gear_inches
    ratio * diameter
  end

  def ratio
    chainring / cog.to_f
  end

  def wheel
    @wheel ||= Wheel.new(rim, tire)
  end

  def wheel_diameter
    wheel.diameter
  end
end
````

This technique becomes necessary when a class contains embedded references to a *message* that is likely to change. Isolating the reference provides some insurance against being affected by that change. Although not every external method is a candidate for this preemptive isolation, it's worth examining your code, looking for and wrapping the most vulnerable dependencies.

#### Remove Argument-Order dependencies

You can't avoid knowing the types, name and meaning of the arguments passed to messages and initializations, but a second, more subtle dependency can be avoided. That is, the fixed order of the arguments.

In the *Gear* example, *initialize* method takes three arguments that must be passed in a fixed order and provides no defaults, each argument must be passed.

If the order and number of the parameters change, all senders of the *new* message will need to change.

It's quite common to tinker with initialization arguments, especially early on, when the design is not quite nailed down, you may go through several cycles of adding and removing arguments. If you use fixed-order arguments each of these cycles may force changes to many dependents, even worse, you may find yourself avoiding making changes the arguments even when your design calls for them because you can't bear to change all the dependents yet again.

##### Use hashes for Initialization Arguments

In the following example, the *initialize* method only takes one hash of initialization arguments.

````(ruby)
class Gear
  attr_reader :chainring, :cog, :wheel

  def initialize(args)
    @chainring = args[:chainring]
    @cog = args[:cog]
    @wheel = args[:wheel]
  end
end
````

This technique add verbosity, but in this case, verbosity has a value; it balances the needs of the present and the uncertainty of the future.

There are case where we can work with fixed-order and hash arguments at the same time, when we have a list of stable parameters and a lengthy list of parameters that are likely to change.

##### Explicitly define defaults

We can add simple non-boolean defaults using Ruby's || method.

````(ruby)
class Gear
  def initialize(args)
    @chainring = args[:chainring] ||  40
    @cog = args[:cog] ||  18
    @wheel = args[:wheel]
  end
end
````

This technique must be used with caution, it cannot handle booleans properly. Since the *||* method acts as an or condition, it first evaluates the left-hand condition and if the expression returns *false* or *nil* it proceeds to evaluate and return the value of the second condition. It works for any other value because the *[]* method os *Hash* returns *nil* for missing keys but it keeps up from explicitly defining a *false* value for an argument.

For handling boolean values and distinguishing correctly between *nil* and *false* we can use the *fetch* method which expects the fetched key to exist in the array and provide mechanisms for explicitly handling missing keys.


````(ruby)
class Gear
  def initialize(args)
    @chainring = args.fetch(:chainring, 40)
    @cog = args.fetch(:cog, 18)
    @wheel = args[:wheel]
  end
end
````

The default can be removed from the constructor, encapsulating them in their own method. This is especially useful when the defaults are more complex than just strings, numbers and booleans.

In this case, the defaults methods defines a hash that is merged with the arguments hash received by the constructor.


````(ruby)
class Gear
  def initialize(args)
    args = defaults.merge(args)

    @chainring = args[:chainring]
    @cog = args[:cog]
    @wheel = args[:wheel]
  end

  def defaults
    {chainring: 40, cog: 18}
  end
end
````

##### Isolate multiparameter initialization

When we don't have the luxury of controlling the signature of the method, we can wrap the sending of messages inside our own methods.

Imagine that *Gear* is part of a framework and that its initialization requires fixed-order arguments. Also imagine that our code has many places where it creates a new instance of *Gear*. We can DRY the initialization of *Gear* by wrapping it with a method.

**The classes in your application should depend on code that you own; use a wrapping method to isolate external dependencies**

````(ruby)
module SomeFramework
  class Gear
    attr_reader :chainring, :cog, :wheel

    def initialize(chainring, cog, wheel)
      @chainring = chainring
      @cog = cog
      @wheel = wheel
    end
    # ...
  end
end

module GearWrapper
  def self.gear(args)
    SomeFramework::Gear.new(args[:chanring], args[:cog], args[:wheel])
  end
end
````

Using a module conveys the idea that we're not supposed to create instances of *GearWrapper*.

The other interesting thing about *GearWrapper* is that its sole purpose is to create instances of some other class. Object oriented designers call this kind of objects *factories*.

The above techinque for substituting an options hash for a list of fixed-order arguments is perfect for cases where you are forced to depend on external interfaces that you cannot change. Do not allow these kinds of external dependencies to permeate your code; protect yourself by wrapping each in a method that is owned by your own application.
