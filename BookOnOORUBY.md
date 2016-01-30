 single responsabilty 
   - even methods has to have this approach
   - depend on behavior, not data


dependency 
  - try to avoid classes that are not Loosely Coupled Code.
  - inject dependency - pass class object to another class, instead of creating the object inside it. 
   - remove arguments order dependency, use hash if you can. 
   - isolate dependency: it’s best to break all unnecessary dependences but, unfortunately, while this is always technically possible it may not be actually possible.
   - Embedding this external dependency inside the gear_inches method is unnecessary and increases its vulnerability.
    - The second idea concerns itself with the concreteness and abstractness of code. The term abstract is used here just as Merriam-Webster defines it, as “disassociated from any specific instance,” and, as so many things in Ruby, represents an idea about code as opposed to a specific technical restriction.This concept was illustrated earlier in the chapter during the section on injecting dependencies. There, when Gear depended on Wheel and on Wheel.new and on Wheel.new(rim, tire), it depended on extremely concrete code. After the code was altered to inject a Wheel into Gear, 
Gear suddenly begin to depend on some- thing far more abstract, that is, 
the fact that it had access to an object that could respond to the diameter message.
  - An Class has to try, at maximum, be independent.   
```           
###############IMPORTANT
Depend on things that change less often than you do is a heuristic that stands in for all the ideas in this section.                  #
```           

## Public vs private methods
Private changes more than public methods, public methods should be more stable.
When programming, you have to keep you attention to what not how. 
For example, you ask calculator how he does her stuff, but you asked what. I give this number, plus this symbol, and another number. What
do you give me? It's like a context.

Every time you create a class, declare its interfaces. Methods in the public
interface should
• Be explicitly identified as such
• Be more about what than how
• Have names that, insofar as you can anticipate, will not change
• Take a hash as an options parameter

Minimize Context
Construct public interfaces with an eye toward minimizing the context they require
from others. Keep the what versus how distinction in mind; create public methods
that allow senders to get what they want without knowing how your class implements
its behavior

Defining Demeter
Demeter restricts the set of objects to which a method may send messages; it prohibits
routing a message to a third object via a second object of a different type. Demeter is
often paraphrased as “only talk to your immediate neighbors” or “use only one dot.”

In order to correct demeter problem, when it appears, you can use delegator.

```
 The purpose of object-oriented design is to reduce the cost of change 
```

Duck types are public interfaces that are not tied to any specific class.
Duck typed objects are chameleons that are defined more by their behavior than
by their class. This is how the technique gets its name; if an object quacks like a duck
and walks like a duck, then its class is immaterial, it’s a duck.

If your design imagination is constrained by class and you find yourself unexpectedly
dealing with objects that don’t understand the message you are sending, your
tendency is to go hunt for messages that these new objects do understand. 
And you should be worried by what interface do, not you knowledge: It’s the
 interface that matters, not the class of the object that implements it.

Use of kind_of?, is_a?, responds_to?, and case statements that switch on
your classes indicate the presence of an unidentified duck. 

###  Polymorphis
```
Polymorphism in OOP refers to the ability of many different objects to
respond to the same message. Senders of the message need not care about
the class of the receiver; receivers supply their own specific version of the
behavior.
A single message thus has many (poly) forms (morphs). 
```

## Inheritance 

Variables with these kinds of names are your cue to notice the underlying pattern.
Type and category are words perilously similar to those you would use when describing
a class. After all, what is a class if not a category or type?

Inheritance provides a way to define two objects as having a relationship such that
when the first receives a message that it does not understand, it automatically forwards,
or delegates, the message to the second. It’s as simple as that.

When you define a new class but do not specify its superclass, Ruby
automatically sets your new class’s superclass to Object. Every class you create is, by
definition, a subclass of something.

You also already benefit from automatic delegation of messages to superclasses.
When an object receives a message it does not understand, Ruby automatically forwards
that message up the superclass chain in search of a matching method implementation.

This technique of defining a basic structure in the superclass and sending messages
to acquire subclass-specific contributions is known as the template method pattern.

## Module Role  

This chapter has been careful to maintain a distinction between classical inheritance
and sharing code via modules. This is-a versus behaves-like-a difference definitely
matters, each choice has distinct consequences. However, the coding techniques for
these two things are very similar and this similarity exists because both techniques rely
on automatic message delegation.

However, it is also possible to add a module’s methods to a single object, using
Ruby’s extend keyword. Because extend adds the module’s behavior directly to an
object, extending a class with a module creates class methods in that class and extending
an instance of a class with a module creates instance methods in that instance. 


In this situation all of the possible receiving objects play a common role.
This role should be codified as a duck type and receivers should implement the duck
type’s interface. Once they do, the original object can send one single message to
every receiver, confident that because each receiver plays the role it will understand
the common message.
In addition to sharing an interface, duck types might also share behavior. When
they do, place the shared code in a module and include that module in each class or
object that plays the role.

Superclasses should not contain code that applies to some, but not all, subclasses.


If you cannot correctly identify the abstraction there may not be one, and if no
common abstraction exists then inheritance is not the solution to your design
problem.



Ruby’s OpenStruct class is a lot like the Struct class that you’ve already seen, it
provides a convenient way to bundle a number of attributes into an object. The difference
between the two is that Struct takes position order initialization arguments
while OpenStruct takes a hash for its initialization and then derives attributes from
the hash.
There 

 In most cases when you see composition it will indicate nothing more
than this general has-a relationship between two objects

However, as formally defined it means something a bit more specific; it
indicates a has-a relationship where the contained object has no life independent
of its container. 

Aggregation is exactly like composition except that the contained object has
an independent life

Duck types works as behave-like-a
