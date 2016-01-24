# Chapter 3. Managing Dependencies

Objects reflect qualities of a real-world problem and the interactions between those objects provide solutions. If we follow the single responsibility principle, an object cannot know everything, inevitably it will have to talk to another object.

All of the behavior in an application, invoked via messages, is dispersed among objects. Therefore, for any desired behavior, and object either knows it personally, inherits it, or knows another object who knows it.

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

Each *coupling* creates a *dependency*. The more one object knows about the other, the more coupled tightly coupled they are. The more tightly coupled they are, the more they behave like a single entity.

Objects that behave like a single entity are harder to use independently, to reuse and to test.

When two (or three or more) objects are so tightly coupled that they behave as a unit, it's impossible to reyse just one. Changes to one object force changes to all. Left unchecked, unmanged dependencies cause an entire application to become an entangled mess.

A day will come when it's easier to rewrite everything than to change anything.

#### Other Dependencies

The four dependencies named before will discussed in the remainder of this chapter but it is worth mentioning a few other common dependency related issues.

One specially destructive kind of dependency is when one object knows another, who knows another, who knows something. Many messages are chained together to reach behavior that lives in a distant object. Message chaining creates a dependency between the original object and every object and message along the way.

This coupling increases the change that the original object has to change because of a change in any of the intermediate objects.

Test dependencies on code can also exist and are very costly. Developers that are new to testing write test code that is too tightly coupled to the code they're testing. This coupling leads to frusration, test break every time the cost is refactored, even when the fundamental behavior of the code does not change.

### Writing Loosely Coupled Code
