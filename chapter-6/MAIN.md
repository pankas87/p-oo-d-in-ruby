# Chapter 6. Acquiring Behavior Through Inheritance

Classical inheritance is a mechanism built into the syntax of most object oriented languages to share code between classes. This chapters teaches you to write codes that properly uses inheritance, its goal is to teach you how to build a technically sound inheritance hierarchy.

Once you understand how to use classical inheritance the concept is easily transferred to other mechanisms for inheritance.

## Understanding Classical Inheritance

There is a simple abstraction for understanding inheritance, it is just a mechanism to automatically delegate messages not understood by an object. Instead of explicitly coding a path to delegate the messages you just declare an inheritance relationship between two objects and the delegation happens automatically.

The term classical makes reference to a mechanism of inheritance based on classes, not to an archaic or old technique. In classical inheritance you make a subclass inherit from a superclass. There are other mechanism for declaring inheritance. In Javascript there is prototypical inheritance (Which is a piece of crap), and you have modules in Ruby, as an additional mechanism for sharing code that we will study in the next chapter.

We will come to understand the uses and misuses of classical inheritance via an example where we start with a single class and goes through several refactorings to reach a satisfactory set of subclasses.

## Recognizing Where to Use Inheritance

The first challenge is recognizing where inheritance would be useful. Let's continue with the FastFeet bike touring company example.

We will start with road bikes, lightweight, with a curved handlebar, skinny tires used for paved roads.

The mechanics of FastFeet must keep bicycles running and take several bike parts in every trip. The spares needed change depending on the bicycle type.

### Starting with a Concrete Class

In our FastFeet application we already have a Bicycle class. Every road bike is represented by an instance of this class.

All bikes have an overall size, a handlebar tape color and a chain type; all the necessary spares must be taken on every trip.

````(ruby)
class Bicycle
  attr_reader :size, :tape_color

  def initialize(args)
    @size       = args[:size]
    @tape_color = args[:tape_color]
  end

  # Every bike has the same defaults for tire and chain size
  def spares
    {
      chain:      '10-speed',
      tire_size:  '23',
      tape_color: tape_color
    }
  end
end

bike = Bicycle.new(
          size:       'M',
          tape_color: 'red'
        )

bike.size # -> 'M'
bike.spares
# { chain:      '10-speed',
#   tire_size:  '23',
#   tape_color: 'red' }
````

Our bicycle responds to the size and tape_color messages and a `Mechanic` can figure out what parts to take on a trip by calling the `spares` method; this method commits the sin of embedding defaults into itself but for our starting point is fairly reasonable.

The class works perfect until something changes, for example, when FastFeet starts offering mountain bike trips.

Mountain bikes and road bike clearly have some similarities but there are also clear differences between them. Your design task is to add support for mountain bikes to the FastFeet application.

Much of the behavior already exists, mountain bikes are, after all, bicycles; they have an overall size and chain and tire sizes, the difference is that road bikes need handlebar tape and mountain bikes have suspension.

### Embedding Multiple Types

In this case where a preexisting class contains most of the behavior, it's tempting to add code to make it fit our needs, adding the necessary methods for handling the suspension (`front_shock`, `rear_shock`) and including conditionals to respond properly to messages, like the next example, which makes the `spares` method work for both road and mountain bikes.

````(ruby)
class Bicycle
  attr_reader :style, :size, :tape_color, :front_shock, :rear_shock

  def initialize(args)
    @style        = args[:style]
    @size         = args[:size]
    @tape_color   = args[:tape_color]
    @front_shock  = args[:front_shock]
    @rear_shock   = args[:rear_shock]
  end

  # Checking "style" starts down a slippery slope
  def spares
    if style == :road
      {
        chain:      '10-speed',
        tire_size:  '25',
        tape_color: tape_color
      }
    else
      {
        chain:      '10-speed',
        tire_size:  '2.1',
        rear_shock: rear_shock
      }
    end
  end
end

bike = Bicycle.new(
          style:        :mountain,
          size:         'S',
          front_shock: 'Manitou',
          rear_shock:   'Fox'
        )

bike.spares
#   {
#     chain:      '10-speed',
#     tire_size:  '2.1',
#     rear_shock: rear_shock
#   }
````

This code makes decisions about the spares part based on the value of `style`. Several problems can occur adding a new style will force us to modify the `if` statement, careless coding where the last option is the default can have unexpected consequences with an unexpected `style`. Also, some of the embedded defaults are now duplicated in each part of the `if` statement.

The public interface of `Bicycle` has some unreliable behaviour. The `size` method still works, `spares` generally works but it can fail with unexpected inputs and the parts methods cannot be trusted, we cannot predict when a specific part has been initialized and objects that hold an instance of `Bicycle` could feel the temptation to check for the `style` of a bike before invoking one of its parts methods.

The code wasn't great in the beginning and it is much worse now. The new flaws have broader consequences, it violates the single responsibility principle, contains stuff that can change unrealiably for different reasons and cannot be reused.

This pattern of coding illustrates vividly what we call an antipattern that suggests a better design after we notice it.

The conditional to check for the `style`, which is nothing more than an attribute holding the category of an object, should bring back memories of a pattern discussed in the previous chapter about duck typing, where we used and `if` state to check the `class` of an object to determine what message to send to that object.

We should think of the class of an object as an specific case of an attribute that holds the category of an object; this makes both the pattern in the last chapter and the pattern exemplified here become the same pattern: I know `who` you are and because of that I know `what` you do.

In the last chapter this pattern allowed us to recognize the presence of a hidden duck type; here the pattern indicates a missing subtype (subclass).

### Finding the Embedded Types

An `if` statement switching on variables named `style`, `type` or `category` gives us a clue about the underlying antipattern, after all, what is a class but a category or a type?

In our case the `style` variable clearly marks a difference between objects that share some common behavior but have some unique properties and methods that apply only to one of the styles and not the other. This is the kind of problem that inheritance solves, highly related types that share common behavior but differ along some dimension.

### Choosing Inheritance

Inheritance is very simple to understand if you look at it from the right perspective.

Objects receive messages, no matter how complicated the code, an object handles a received message in one of two ways, either respond directly or it passes the message on to some other object for a response. Inheritance allows us to define a relationship between two objects such that when the first receives a message it does not understand it automatically delegates it to the second. It's as simple as that.

In most object-oriented languages objects can inherit from only one superclass, this is because having multiple super classes creates complex challenges like deciding to which super class delegate a message received by an object that does not understand said message, or in the case of having several super classes implementing the same method, deciding on the priority of the implementations.

The designers of many object oriented languages sidestep this complexity by restricting inheritance to only one superclass.

We use inheritance all the time even if we have not created a class hierarchy ourselves. When we define a class and don't specify its superclass Ruby automatically sets it to Object. Every class you create is by definition a subclass of something. A nice example of automatic delegation of messages is the implementation of the `nil?` method.

````
class Object
  def nil?
    false
  end
end

class NilClass < Object
  def nil?
    false
  end
end

class String < Object
end

class MyString < String
end
````

The original `nil?` method is defined in the `Object` class, any class that has `Object` in its class hierarchy (Literally any Ruby class) will have a default implementation of the `nil?` method to fall back to, one that responds with the value `false`; and `NilClass` implements its own version of `nil?` that responds with the value `true`. This way we can be sure that any object that does not implement the `nil?` method is by default `not nil`.

The fact that unknown messages get delegated up the superclass hierarchy implies that subclasses are everything their classes are, plus more. An instance of `String` ``is`` a `String` but is also an `Object`. Subclasses are specializations of their super classes and must respond appropriately to any message defined in the public interface of said super classes.

Going back to the bicycle example, we should revert to the original version of `Bicycle`, perhaps a mountain bike it's just a specification of `Bicycle` and we could solve our design problem by applying inheritance.

## Misapplying Inheritance

The following is a first attempt at a MountainBike class, displaying some difficulties encountered by novices in implementing inheritance. This subclass inherits from Bicycle and overrides two methods, `initialize` and `spare`.

````(ruby)
class MountainBike < Bicycle
  attr_reader :front_shock, :rear_shock

  def initialize(args)
    @front_shock  = args[:front_shock]
    @rear_shock   = args[:rear_shock]
    super(args)    
  end

  def spares
    super.merge(rear_shock: rear_shock)
  end
end
````

When you send `super` it sends the message up the super class chain until it finds any implementation in the hierarchy.

Jamming the new MountainBike class directly under the Bicycle class was blindly optimistic and running the code exposes several flaws.

````
mountain_bike = MountainBike.new(
                  size: 'S',
                  front_shock: 'Manitou',
                  rear_shock: 'Fox'
                )

mountain_bike.size # -> 'S'

mountain_bike.spares
# ->  {
#       :tire_size  => '23',       
#       :chain      => '10-speed',
#       :tape_color => nil,
#       :front_shock => 'Manitou',
#       :rear_shock => 'Fox',
#     }
````

Checking out the previous code, `size` responds correctly but `spares` clearly gives us incorrect results, showing a skinny tire size and and uninitialized `tape_color` which is not working properly and is not necessary for a mountain bike that doesn't use handlebar tape.

We could easily expect `MountainBike` to have a strange mix of road and mountain bike behavior. `Bicycle` is a concrete class that was not meant to be subclassed, combining behavior general to all bicycles and specific to road bicycles, and `MountainBike` inherits both the general and the specific.

Because design is evolutionary, this situation arises all the time. The problem here started with the names of these classes.

## Finding the Abstraction

In the first design we chose a generic name, `Bicycle` for an object that was a little more specific, the current `Bicycle` class doesn't represent any kind of bike, it represents a very specific road bike.

In the original version the generic name was good enough, and having an unnecessary overly specific name like `RoadBike` would have been a bad choice.

Now that we have a `MountainBike` the `Bicycle` name is misleading because it implies inheritance between these two classes but `Bicycle` actually represents a road bike. Subclasses are specializations of their super classes; a `MountainBike` should be everything a `Bicycle` is, plus more.

There are two rules when using inheritance:

+ The objects that you are modeling must truly have a generalization-specialization relationship.
+ You must use the correct coding techniques.

Now it's time to separate the general from the specific in the `Bicycle` class, and move the specific to a new, separate `RoadBike` class.

### Creating an Abstract Superclass

In our new design `Bicycle` will be the superclass for `MountainBike` and `RoadBike`, `Bicycle` will contain the common behavior and the subclasses will add specializations. This new version of bicycle will not define a complete bike, just the bits that all bicycles share, and because of that we should not directly use it to create instances, this is known in object oriented design as an `abstract class`. Some OOP languages have a syntax that allows you to explicitly declare a class abstract and the compiler or interpreter prevents you from creating instances of said abstract class. Ruby doesn't have any mechanism to define abstract classes, in line with its trusting nature, only good sense prevents other programmers from instantiating classes that should be considered as abstracts.

Abstract classes exist to be subclassed, that's their sole purpose. They supply a repository of common behavior shared among subclasses.

It is recommended to create a hierarchy of abstract and concrete classes only when you have a specific requirement that forces you to deal with specializations of a general class. In our Bicycle example, the original `Bicycle` class, despite having a generic name, was used to model a specific type of bike, a road bike, this was good enough for the original situation where we had only one class to deal with. Creating a hierarchy has costs, the best way to minimize these costs is to wait until you have enough information to get the abstraction right, usually we should wait until we have at least three specific classes sharing behavior. Waiting until having more information available will increase the odds of finding the right abstraction.

For now, let's assume we have a good reason to create a `Bicycle` hierarchy. The first step is to outline the general structure of our classes.

````(ruby)
class Bicycle
  # This class is now empty
  # All code has been moved to RoadBike
end

class RoadBike < Bicycle
  # Now a subclass of Bicycle
  # Contains all code from the old Bicycle class
end

class MountainBike < Bicycle
  # Still a subclass of bicycle (Which is now empty)
  # Code has not changed
  # Code that it depends on has been moved from the parent to a peer
end
````

Now instead of containing too much behavior, `Bicycle` contains none, the common behavior needed by `MountainBike` is not accessible to that class. We arrange the code this way because it's easier, less expensive and less risky to promote common (abstract) behavior rather than to demote specific behavior.

`RoadBike` still works because it contains everything it needs, `MountainBike` on the other hand is relying on behavior inherited from its parent class which is now empty. For example, here' what happen when you create instances of each subclass and ask them for size.

````(ruby)
road_bike = RoadBike.new(
              size: 'M',
              tape_color: 'red' )

road_bike.size # => "M"

mountain_bike = MountainBike.new(
                  size: 'S',
                  front_shock: 'Man',
                  rear_shock: 'Fox' )

mountain_bike.size # NoMethodError: undefined method 'size'

````

### Promoting Abstract Behavior

It's obvious why this fails, the `size` method is not defined anywhere along the hierarchy of MountainBike, neither is the `spares` method which is also common to both subclasses. Let's start by promoting size, which requires three changes: promoting the attribute reader and the initialization up to `Bicycle` from `RoadBike`, and sending `super` inside the `initialize` of `RoadBike`.

````(ruby)
class Bicycle
  attr_reader :size # <- promoted from RoadBike

  def initialize(args = {})
    @size = args[:size] # <- promoted from RoadBike
  end
end

class RoadBike < Bicycle
  attr_reader :tape_color

  def initialize(args)
    @tape_color = args[:tape_color]
    super(args) # <- RoadBike now MUST send super
  end
end
````

Now both `RoadBike` and `MountainBike` delegate the `size` message up their class hierarchy until it is found in the `Bicycle` class. This common behavior is now defined where it should be, in the abstract parent class.

Some might note that the code for handling a bicycle's size was moved twice. Once from `Bicycle` to `RoadBike` and then back up from `RoadBike` to `Bicycle`. You might feel tempted to just skip this double moving and leave it in `Bicycle` but this a risky move. The push-everything-down-and-then-pull-some-things-up strategy is an important part of the refactoring. Correctly separating concrete from abstract behavior will prevent difficulties while using inheritance.

If you start the refactoring by leaving everything inside the `Bicycle` class and then push the concrete down to `RoadBike` it is very likely that you will end up leaving inside `Bicycle` dangerous concrete behavior that belongs in `RoadBike` and it will go unnoticed for a long time. On the other hand, if you start promoting abstract behavior up to `Bicycle`, you will realize soon enough when some abstract behavior is missing, as long as any of the subclasses starts to fail because of some missing behavior it depends on; this situation will force you to either duplicate code or promote the common behavior to the abstract class, since even the most junior of developers avoids code duplication, the problem will be noticed and fixed by anyone working on the application.

When deciding between refactoring strategies, indeed, when deciding between design strategies in general, it's useful to ask the question: "What will happen if I'm wrong". Promotion failures have low consequences, demotion failures on the other hand could go unnoticed for a long time, and inexperienced developers might start using conditional statements to avoid triggering this concrete behavior, creating quirky subclasses that will be constantly checking for type; this type checking could even leak to other parts of the application, increasing dependencies and creating an anti pattern very similar to the one we saw in the duck typing chapter.

The general rule for refactoring into a new inheritance hierarchy is to arrange code so that you can promote abstractions rather than demote concretions. Every decision you make includes two costs: one to implement it and another to change it when you discover that you were wrong.

### Separating Abstract from Concrete

Both subclasses implement a version of the `spares` method. `Roaadbike`'s version is the original self-contained version, therefore, still works.

````(ruby)
class RoadBike < Bicycle
  # Rest of the code

  def spares
    { chain:      '10-speed',
      tire_size:  '23',
      tape_color: tape_color }
  end
end
````

The `spares` method defined in `MountainBike` is a leftover from the first sub classing attempt and relies on the parent's `spares` method that no longer exists up the hierarchy chain. Sending the `spares` message to an instance of `MountainBike` results in a `NoMethodError` exception.

````(ruby)
mountain_bike.spares # NoMethodError: super: no superclass method 'spares'
````

The obvious solution is adding a `spares` method in our `Bicycle` class but it's not as simple as just copying the `spares` method in `RoadBike`.

`RoadBike`'s version of the method knows too much, it's too coupled to the default hard coded values of a road bike and has the `tape_color` attribute which is not common to all bicycles. We must untangle the mess, separate the abstract from the concrete. The abstract parts will be promoted to `Bicycle` while the remaining concrete parts will stay in `RoadBike`.

The first step is to add getter and setter methods for the common attributes, `chain` and `tire_size`, rather than using hard code values.

````(ruby)
class `Bicycle`
  attr_reader :size, :chain, :tire_size

  def initialize( args={} )
    @size      = args[:size]
    @chain     = args[:chain]
    @tire_size = args[:tire_size]
  end
end
````

### Using the Template Method Pattern

We will change the `initialize` method to send messages to get defaults. We will add the `default_chain` and `default_tire_size` to our methods.

Wrapping defaults inside methods is generally a good practice but these new message serve another purpose as well, which is giving the subclasses an opportunity to contribute specializations by overriding the default methods. This technique is known as the ``template method pattern``.

````(ruby)
class Bicycle
  attr_reader :size, :chain, :tire_size

  def initialize( args = {} )
    @size      = args[:size]
    @chain     = args[:chain] || default_chain
    @tire_size = args[:tire_size] || default_tire_size
  end

  def default_chain
    '10-speed'
  end
end

class RoadBike < Bicycle
  # ...

  def default_tire_size
    '23'
  end
end

class MountainBike < Bicycle
  # ...

  def default_tire_size
    '2.1'
  end
end
````

`Bicycle` now provides structure, a common algorithm for its subclasses, allowing its them to influence the algorithm via overriding the methods.

There's still a booby trap we need to defuse.

### Implementing Every Template Method

The `initialize` method in the parent class `Bicycle` sends the `default_tire_size` message but doesn't implement it, this responsibility is delegated to the subclass. This could be omitted by an innocent programmer who, for example, creates a new bicycle type, the recumbent bike an forgets to implement the `default_tire_size` method.

````(ruby)
class RecumbentBike < Bicycle
  def default_chain
    '9-speed'
  end
end

bent = RecumbentBike.new # <- NameError: undefined local variable or method 'default_tire_size'
````

The original designer of the class rarely encounters this problem since he knows what are the requirements for subclasses. This error will be encountered by other programmers who could be changing the application to meet new requirements. The root cause of the problem is that `Bicycle` impose a requirement on subclasses that is not obvious by a quick glance at the parent class. The solution is easy, every class that uses the template method pattern should implement the method itself, even it simply throws an exception. This is a cost-effective way to document the requirements that subclasses must meet.

````(ruby)
class  Bicycle
  # ...

  def default_tire_size
    raise NotImplementedError, "This #{self.class} cannot respond to:"
  end
end

bent = RecumbentBike.new

# NotImplementedError:
# This RecumbentBike cannot respond to: 'default_tire_size'
````

It doesn't matter when the error is detected, it will be easy to correct and unambigous.

Code that fails with reasonable error messages is easy to do in the present and will save us a lot of time and trouble in the future. Each error message is a small thing, but small things accumulate to produce big effects.Always document template method requirements by implementing matching methods and raising useful errors.

## Managing Coupling Between Superclass and Subclasses

`Bicycle` now contains most of the abstract behavior, now we need to add the `spares` method. There are many ways to implement it, with different levels of coupling between superclass and subclasses. We should always strive to reduce coupling to a minimum.

### Understanding Coupling

The simplest implementation produces a lot of coupling. Taking a look at `MountainBike` we see that its `spares` implementation sends `super`, and expects it to return a hash that will merge with its own spares hash.

Then we should implement the `spares` method in our `Bicycle` class and rather than using hard coded values like the old `spares` implementation in `RoadBike`.

Finally, it's just a matter of replicating in `RoadBike`'s `spare` a method a similar pattern to the one used in `MountainBike`: Sending super, expecting to receive a hash and merging that hash with `RoadBike`'s concrete spares.

````(ruby)
class Bicycle
  attr_reader :size, :chain, :tire_size

  def initialize( args = {} )
    @size      = args[:size]
    @chain     = args[:chain] || default_chain
    @tire_size = args[:tire_size] || default_tire_size
  end

  def spares
    {   tire_size:  tire_size,
        chain:      chain }
  end

  def default_chain
    '10-speed'
  end

  def default_tire_size
    raise NotImplementedError, "This #{self.class} cannot respond to:"
  end
end

class RoadBike < Bicycle
  attr_reader :tape_color

  def initialize(args)
    @tape_color = args[:tape_color]
    super(args)
  end

  def spares
    super.merge({ tape_color: tape_color })
  end

  def default_tire_size
    '23'
  end
end

class MountainBike < Bicycle
  attr_reader :front_shock, :rear_shock

  def initialize(args)
    @front_shock = args[:front_shock]
    @rear_shock = args[:rear_shock]
    super(args)
  end

  def spares
    super.merge({ front_shock: front_shock, rear_shock: rear_shock })
  end

  def default_tire_size
    '2.1'
  end
end
````

This class works as it is but there is still another booby-trap that needs to be defused.

Both subclasses know things about themselves, like their `spares` implementation, but also know things about their parent, for example, its `spares` implementation that returns a hash. Knowing things about other classes creates dependencies, which makes change harder in the future.

The trap that this coupling hide is illustrated in this example, showing a new subclass of `Bicycle` that doesn't send super in `initialize`

````(ruby)
class RecumbentBike < Bicycle
  attr_reader :flag

  def initialize(args)
    @flag = args[:flag]
    # Forgot to send super because ID10-T
  end

  def spare
    super.merge({flag: flag})
  end

  def default_chain
    '9-speed'
  end

  def default_tire_size
    '28'
  end
end

bent = RecumbentBike.new(flag: 'tall and organge')

bent.spares
=begin
{ :tire_size => nil,  <- didn't get initialized
  :chain     => nil,  <- didn't get initialized
  :flag      => 'tall and orange' }
=end

````

Failing to send super in `RecumbentBike`'s `initialize` the common initialization provided by `Bicycle` is missed and common attributes don't get their correct values. A similar thing happens if the `spares` method forgets to send super, it will not fail but the `spares` will not have the parts common to all bicycles (tires, chain). This pattern requires that subclasses know what they do but also how to interact with their superclass. It makes sense that subclasses know the specializations they contribute with, but having them know how to interact with their superclasses causes a lot of trouble. It pushes knowledge of the algorithm down to subclasses and causes duplication of code. Additionally, it increases the odds that future programmers that don't know the full details of the abstraction will create errors when writing new subclasses.

### Decoupling Subclasses Using Hook Messages

We'll do one last refactoring to prevent these problems. We will send hook messages in our superclasses, this will allow our subclasses to contribute specializations by implementing matching methods.

In the following example, subclasses contribute to specialization via the `post_initialize` sent by the superclass' `initialize` method.

````(ruby)
class Bicycle
  attr_reader :size, :chain, :tire_size

  def initialize( args = {} )
    @size      = args[:size]
    @chain     = args[:chain] || default_chain
    @tire_size = args[:tire_size] || default_tire_size

    # The superclass both sends
    post_initialize(args)
  end

  # and implements this method
  def post_initialize(args)
    nil
  end

  # ...
end

class RoadBike < Bicycle
  def post_initialize(args)
    @tape_color = args[:tape_color]
  end
end
````

Since we reove the `initialize` method from the subclass, now initialization is completely controlled by the superclass. Subclasses no longer need to worry about initialization routines, their only responsibility is to add specialization to their superclass. Coupling is reduced, subclasses need to know less about how and when initialization occurs, they only handle their specifics initializations.

Putting control of the timing in the superclass means the algorithm can change  without forcing changes on the superclasses.

The same technique can be used to remove the sent of super in the subclasses implementation of `spares`.

````(ruby)
class Bicycle
  # ...

  def spares
    { tire_size:  tire_size,
      chain:      chain }.merge(local_spares)
  end
end

class RoadBike < Bicycle
  # ...

  def local_spares
    { tape_color: tape_color }
  end
end
````
