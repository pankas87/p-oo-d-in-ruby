# Chapter 5. Reducing Costs with Duck Typing

Duck typing is one technique that helps us to reduce the cost of change, which is the main goal of object oriented design.

Knowing that messages are at the design center of our application and being committed to construct rigorously defined public interfaces, we can mix these two ideas into duck typing.

Duck types are public interfaces that are not tied to a specific class, they add enormous flexibility to our application, replacing costly dependencies on classes with more forgiving dependencies on messages. Duck typed objects are defined by their behavior, not by their class. That is how the technique get its name, if an object quacks like a duck and walks like a duck then it is a duck.

## Understanding Duck Typing

In programming languages the term "type" describes the category of a variable.

Using the knowledge of the category of the contents of a variable an application can make assumptions about the behavior of those contents; application reasonably assume that numbers can be used in mathematical expressions, strings can be concatenated and arrays can be indexed.

In Ruby these expectations are beliefs about an object's public interface, knowing an object's type you know to what messages it can respond to. For instance, an instance of *Mechanic* contains the compete public interface of *Mechanic*, but objects in Ruby can implement more than one public interface.

Users of an object don't need and should not worry about its class, which is just one of many ways an object can acquire a public interface. It is not what an object is that matters, is what it does.

There are infinite design possibilities when an object can be any kind of thing and we trust it to be what we expect it to be. These inifnite possibilities enable us to create flexible designs that can either be marvels of structured creativity of terrifying design abominations that cannot be understood by anyone.

To use this flexibility wisely we must recognize these accros-classes types an construct their interfaces intentionally and diligently as we did in Chapter 4. The public interfaces of duck types must be explicit and well documented.

### Overlooking the duck

````(ruby)
class Trip
  attr_reader :bicycles, :customers, :vehicle

  # This mechanic argument could be of any class
  def prepare(mechanic)
    mechanic.prepare_bicycles(bicycles)
  end
end

# If you happen to pass an instance of this class, it works
class Mechanic
  def prepare_bicycles(bicycles)
    bicycles.each { |bicycle| prepare_bicycle(bicycle) }
  end

  def prepare_bicycle(bicycle)
    # Do stuff
  end
end
````

The *Mechanic* class is not referenced but still there is a subtle dependency in the *prepare* method, which expects on objects that responds to a method named *prepare_bicycle*.

### Compounding the problem

Now requirements change and *Trip* needs more than one preparer, it needs a trip coordinator and a driver. We add *TripCoordinator* and *Driver* classes and give them the behavior they're responsible for, also we modify the *prepare* method in *Trip* to receive multiple preparers and invoke the correct behavior from each one.

````
class Trip
  attr_reader :bicycles, :customers, :vehicle

  def prepare(preparers)
    preparers.each do |preparer|
      case preparer
      when Mechanic
        preparer.prepare_bicycles(bicycles)
      when TripCoordinator
        preparer.buy_food(customers)
      when Driver
        preparer.fill_water_tank(vehicle)
      end
    end
  end
end
````

We can see clearly that there are more dependencies, *Trip* now needs to know the specific names of three classes and methods, and said methods arguments. As we have seen earlier, with more dependencies, the costs and risks of change increase.

This code will lead you to a corner with no way out. It is written by programmers who fail to see beyond a class-based perspective, ignoring important messages. As we saw in Chapter 4, messages are at the design center of out application, not classes.

We should look at our application beyond the limited imagination of a class based point of view,

### Finding the Duck

The key to remove the dependencies is to understand that since the *prepare* serves a single purpose, the arguments arrive with the intention to collaborate to accomplish this goal, they're all here for the same reason which is unrelated to the argument's original class.

Consider this problem from *Trip*'s point of view, it expects that every arguments acts like a preparer.

This expectation turns the table, we are freed from existing classes and have discovered a duck type, now we must define the public interface of this duck type, by asking ourselves what message can the *prepare* method send to every preparer.

From this point of view the answer is obvious: *prepare_trip*

Now we rewrite the prepare method, in this version it only expects its arguments to be *Preparers* that can respond to *prepare_trip*.

*Preparer* it's not a concrete class; it's just an abstraction, an agreement about the public interface on an idea.

Objects that implement *prepare_trip* are *Preparers* and objects that interact with *Preparers* only need to trust them to implement the *prepare_trip* method, which is the *Preparer* interface.

````
class Trip
  attr_reader :bicycles, :customers, :vehicle

  def prepare(preparers)
    preparers.each { |preparer| preparer.prepare_trip(self) }
    end
  end
end

class Mechanic
  def prepare_trip(trip)
    trip.bicycles.each { |bicycle| prepare_bicycle(bicycle) }
  end
end

class TripCoordinator
  def prepare_trip(trip)
    buy_food(trip)
  end
end

class Driver
  def prepare_trip(trip)
    vehicle = trip.vehicle
    gas_up(vehicle)
    fill_water_tank(vehicle)
  end
end

````

The *prepare* method can now accept new preparers without the need for change and it's easy to create new *Preparers* when necessary.

### Consequences of Duck Typing

The first version of our code is very concrete, this makes it simple to understand but harder to extend. The final, duck-typed version is more abstract; it places slightly greater demands on your understanding but in return offers ease of extension. We can simply create new *Preparers* and pass them to the *prepare* method without changing anything in the *Trip* class.

This tension between the costs of concretion and the costs of abstraction is fundamental to object-oriented design. Concrete code is easy to understand but costly to extend. Abstract code may seem more obscure at first but once understood, is far easier to change.

### Polymorphism

*Morph* is the greek word for form, *morphism* is the state of having a form, therefore *polymorphism* is the state of having many forms.

Polymorphism in OOP refers to the ability of many different objects to respond to the same message. Senders of the message need no to care about the class of the receiver.

A single message has many (poly) forms (morphs).

There are several ways to achieve polymorphism; duck typing is one of them; inheritance and behavior sharing (via Ruby modules) are others.

Polymorphic methods have an implicit bargain, they agree to be interchangeable from the point of view of the sender. When we are implementing polymorphism it's our responsibility to enforce this agreement and make sure that objects that respond to polymorphic messages are really interchangeable

## Writing Code That Relies on Ducks

It is easy to implement duck types, the real design challenge is being able to detect when our application could benefit from duck typing.

### Recognizing Hidden Ducks

Several common patterns indicate the presence of a hidden duck in our existing code:

+ Case statements that switch on class
+ *kind_of?* and *is_a?*
+ *responds_to?*

#### Case Statements That Switch on Class

The example we already saw, the most common and obvious indicator of the presence of a hidden duck.

#### kind_of? and is_a?

Using *kind_of?* or *is_a?* to determine what message should be sent to an object is a clear indication that there is an undiscovered, it really is no different from the previous case that switches on class.

#### responds_to?

Using *responds_to?* eliminates the dependency on class names but we're still bound to knowing method names and arguments.

````(ruby)
def prepare
  preparers.each do |preparer|
    if preparer.responds_to?(:prepare_bicyles)
      preparer.prepare_bicycles(bicycles)
    elsif preparer.responds_to?(:buy_food)
      preparer.buy_food(customers)
    elsif preparer.responds_to?(:gas_up)
      preparer.gas_up(vehicle)
      preparer.fill_water_tank(vehicle)
    end
  end
end
````

Removing the dependencies on class's names didn't free us from having to know that we need specific class methods, with certain arguments. Even though we're not referencing them explicitly, we're still expecting very specific classes.

### Placing Trust in Your Ducks

Validating an object's class before sending it a message indicates the presence of an unidentified duck. You're basically saying: I know who you are and because of that I know what you do. This exposes a lack of trust in collaborating objects.

This style of code indicates a missing object just like Demeter violations did. Instead of a concrete intermediate class, in this case we're missing an abstract duck type.

Flexible applications are built on objects that operate on trust; it is your job to make the objects trustworthy.

When you find these patters concentrate on the offending code's expectations, they can help you locate the duck type.

Once you have a duck type in mind, define its interface, implement that interface and trust the implementers to behave correctly.

### Documenting Duck Types

The *Preparer* duck type and its public interface are a concrete part of the design but a virtual part of the code. It is just an implicit agreement on method names and arguments, this abstraction makes it strong as a design tool but makes it also less obvious in the code.

Duck types public interfaces must be documented and tested well, and good tests are the best documentation, so you are already halfway done, you only need to write the tests, like we'll see in Chapter 9.

### Choosing Your Ducks Wisely

Let's take a look at a code example that clearly violates the guidelines above, by testing the class of an object before deciding what message to send.

````(ruby)
def firsr(*args)
  if args.any?
    if args.first.kind_of?(Integer) || (loaded? && ! args.first.kind_of?(Hash))
      to_a.first(*args)
    else
      apply_finder_options(args.first).first
    end
  else
    find_first
  end
end
````

Event though this code is deciding the method to send based on the class there is a big difference with the examples studied above and that is the stability of the classes this method is depending on. When `first` depends on `Integer` and `Hash` it is depending on stable Ruby core classes that are very unlikely to change in a way that forces changes on `first`. It is a safe dependency, a duck type probably exists somewhere but we gain too little from finding it and implementing it, our cost of change will remain nearly the same.

The decision to create a duck type relies on judgement, if creating one reduces costs by removing unstable dependencies, do so. Lowering costs is the purpose of design.

The underlying duck in the previous example would force us to monkey patch the Ruby core classes `Hash` and `Integer`; the trade off here is very different, changing Ruby base classes introduces greater risks, the benefit of applying this monkey patch should be big enough to justify and defend this design decision and the risks it would bring.

## Summary

Messages are at the center of object-oriented applications and they pass among objects along public interfaces. Duck typing detaches these public interfaces from specific classes, creating virtual types that are defined by what they do instead of by who they are.

Duck typing reveals underlying abstractions that might otherwise be invisible. Depending on these abstractions reduces risk and increases flexibility, making your application cheaper to maintain and easier to change.
