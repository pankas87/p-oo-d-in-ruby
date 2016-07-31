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

In our new design `Bicycle` will be the superclass for `MountainBike` and `RoadBike`, `Bicycle` will contain the common behaviour and the subclasses will add specializations. This new version of bicycle will not define a complete bike, just the bits that all bicycles share, and because of that we should not directly use it to create instances, this is known in object oriented design as an `abstract class`. Some OOP languages have a syntax that allows you to explicitly declare a class abstract and the compiler or interpreter prevents you from creating instances of said abstract class. Ruby doesn't have any mechanism to define abstract classes, in line with its trusting nature, only good sense prevents other programmers from instantiating classes that should be considered as abstracts.

Abstract classes exist to be subclassed, that's their sole purpose.
