## Java Sealed Type

Java classes by default are open for extension. A library class created and compiled years ago can be extended by any of the future classes and its implementation can be altered. Though this looks powerful, there are a few instances where one might need to stop classes/interface for the extension. 

Until J16/J17, an extension can be avoided by using either of the following options.

- Declare classes to be final
- Make superclass or interface package-private.

But the above two options has it's own downsides like interfaces can't be declared final, child classes references can't be assigned to parent class outside the package in which it is defined.

Sealed types have been introduced to handle this case where an author of a class or interface controls which code is responsible for implementing it. 


### Java Grammer

**NormalClassDeclaration:**

{ClassModifier} class TypeIdentifier [TypeParameters]
    [Superclass] [Superinterfaces] [PermittedSubclasses] ClassBody

**ClassModifier:**

  (one of)
  Annotation public protected private
  abstract static sealed final non-sealed strictfp

**PermittedSubclasses:**

  permits ClassTypeList

**ClassTypeList:**

  ClassType {, ClassType}

A sealed class or interface can be extended or implemented only by those classes and interfaces permitted to do so. It is a compile-time error if any class extends a sealed class but is not permitted to do so.

**Example:**

```
public abstract sealed class Vehicle 
        permits Van, Truck, Car { ... }
```

```
sealed interface Celestial 
    permits Planet, Star, Comet { ... }

final class Planet implements Celestial { ... }
final class Star   implements Celestial { ... }
final class Comet  implements Celestial { ... }
```


### Constraints

- The sealed class and its permitted subclasses should be present to the same module([JPMS](https://openjdk.java.net/projects/jigsaw)). If declared in an unnamed module, all the classes should be present in the same package. For example, in the following declaration of Shape its permitted subclasses are all located in different packages of a named module:

```
package com.example.vehicle;

public abstract sealed class Vehicle
    permits com.example.electric.Car,
            com.example.combustion.Car,
            com.example.hydrogen.Car { ... }
```

- Every permitted subclass must directly extend the sealed class.

- Every permitted subclass must use a modifier to describe how it propagates the sealing initiated by its superclass
    -  Stop from being extended further by declaring itself as final. (Record classes are implicitly declared final.)
    - Allow extension by using sealed in a restricted fashion.
    - By specifying non-sealed to allow open for extension by unknown subclasses. A sealed class cannot prevent its permitted subclasses from doing this. (The modifier non-sealed is the first hyphenated keyword proposed for Java.)

```
package com.example.vehicle;

public abstract sealed class Vehicle 
        permits Van, Truck, Car { ... }

public final class Van extends Vehicle { ... }

public sealed class Truck extends Vehicle 
        permits PickUpTruck, TrailerTruck { ... }
public final class TrailerTruck extends Truck { ... }
public final class PickUpTruck extends Truck { ... }

public non-sealed class Car extends Vehicle { ... }
```

As the allowed child classes are listed during declaration, all the allowed implementations are known to the compiler and can be used in the newer pattern-matching/switch expressions similar to enums. But if one of the permitted classes declares itself to be non-sealed, the list again becomes exhaustive and you lose the true power of sealed types.

For example, for the above created sealed class, as the number of permissible classes is known at compile-time, the compiler doesn't throw an error if all the values are handled in various switch conditions.

```
Vehicle refuel(Vehicle vehicle, double value) {
    return switch (vehicle) {   // pattern matching switch
        case com.example.electric.Car e    -> e.refuel(value); 
        case com.example.hydrogen.Car h -> h.refuel(value);
        case com.example.combustion.Car c    -> c.refuel(value);
        // no default needed!
    }
}
```

It is not possible for a class to be both sealed (implying subclasses) and final (implying no subclasses), or both non-sealed (implying subclasses) and final (implying no subclasses), or both sealed (implying restricted subclasses) and non-sealed (implying unrestricted subclasses).

With the addition of sealed type, java gains a powerful feature where pre-determined implementations can be done and future/unexpected extensions can be avoided.


