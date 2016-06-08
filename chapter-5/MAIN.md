# Chapter 5. Reducing Costs with Duck Typing

Duck typing is one technique that helps us to reduce the cost of change, which is the main goal of object oriented design.

Knowing that messages are at the design center of our and being committed to construct rigorously defined public interfaces, we can mix these two ideas into duck typing.

Duck types are public interfaces that are not tied to a specific class, they add enormous flexibility to our application, replacing costly dependencies on classes with more forgiving dependencies on messages. Duck typed objects are defined by their behavior, not by their class. That is how the technique get its name, if an object quacks like a duck and walks like a duck the it is a duck.

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

The key to remove the dependencies is to understand that since the *prepare* serves single purpose, the arguments arrive with the intention to collaborate to accomplish this goal, they're all here for the same reason which is unrelated to the argument's original class.

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



### Polymorphism

## Writing Code That Relies on Ducks

### Recognizing Hidden Ducks

#### Case Statements That Switch on Class

#### kind_of? and is_a?

#### responds_to?

### Placing Trust in Your Ducks

### Documenting Duck Types

### Sharing Code Between Ducks

### Choosing Your Ducks Wisely

## Conquering a Fear of Duck Typing

### Subverting Duck Types with Statis Typing

### Static vs Dynamic Typing

### Embracing Dynamic Typing

## Summary
