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
