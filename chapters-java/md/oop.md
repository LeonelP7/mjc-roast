# OOP

<details>
<summary>What are the main programming paradigms?</summary>

* Imperative programming: This is a style where you provide a sequence of commands for the computer to follow
    * procedural programming
    * structured programming
    * object-oriented programming(OOP)
* Declarative programming: A style that focuses on "what" the program should accomplish rather than "how"
* Functional programming: A paradigm that treats computation as the evaluation of mathematical functions and avoids changing-state and mutable data
* Reactive Programming: A paradigm oriented around data streams and the propagation of change

</details>

<details>
<summary>What are the advantages and disadvantages of an object-oriented programming approach?</summary>

Advantages of OOP
The primary benefit of OOP is code reuse, as all the fundamental principles work together to facilitate the efficient use of existing code. Specific advantages include:
* Implementation Hiding (Encapsulation): By wrapping data and methods into a single unit, OOP protects an object's internal state from outside interference.
* Extensibility (Inheritance): It allows for the creation of new entities based on existing ones, enabling developers to build upon existing logic without starting from scratch.
* Flexibility (Polymorphism): It provides the ability to have different forms for the same entity, allowing for more generic and flexible code.
* Effective Modeling (Abstraction): OOP allows for describing complex entities through a set of common characteristics, defining a clear "contract" for how to interact with an object without needing to understand its internal complexity.

Trade offs
* Performance Overhead: Abstract classes and interfaces are slightly slower than concrete implementations because the system must perform a search to locate the correct overridden method at runtime.
* Slower Memory Access: Because OOP relies heavily on objects, data is primarily stored in the Heap, which has slower access, allocation, and deallocation speeds compared to the Stack.
* Thread-Safety Complexity: Unlike Stack memory (which is private to a thread), the Heap is shared among all threads. This means objects are not thread-safe by default and require explicit synchronization, which can increase the complexity of the code and potentially slow down the application.
* Memory Fragmentation: Dynamic allocation on the heap can lead to fragmentation, which requires a Garbage Collection cycle to manage. This process can cause "stop-the-world" events where the application temporarily pauses

</details>

<details>
<summary>What are the basic principles of OOP?</summary>

* `Encapsulation` is the practice of bundling data (variables) and methods that act on that data into a single unit (a class). We use access modifiers to achieve data hiding, ensuring that the interal state of an object is protected and only accessible through public methods like getters and setters.
* `Inheritance` is the creation of a new entity based on an existing one.
* `Polymorphism` is the ability to have different behaviors for different forms of the same entity. Is the ability of a subclass to provide a specific implementation of a method that is already defined in its parent class
* `Abstraction` is the process of hiding the internal complexity and only exposing the essential features  interacting with an entity (often called a contract). This allows developers to interact with a clean, high-level interface without needing to understant the underlaying code machinery.
* `Reuse` - everything listed above works for code reuse.

</details>

<details>
<summary>Tell us about the basic concepts of OOP: `class`, `object`, `interface`?</summary>

* A `class` is a way of describing an entity, defining the state and behavior that depends on this state, as well as the rules for interacting with this entity (contract). +
From a programming point of view, a class can be viewed as a set of data (attributes, class members) and functions to work with them (methods). +
From the point of view of the structure of the program, a class is a complex data type.

* An `object` (instance) is a separate representative of a class with a specific state and behavior that is completely determined by the class. Each object has specific attribute values and methods that operate on those values based on the rules defined in the class.

* An `interface` is a collection of class methods available for use. The interface of a class will be a set of all its public methods together with a set of public attributes. Basically, an interface specifies a class, clearly defining all possible actions on it.

In summary, the class defines the structure, the object is the actual entity living in memory, and the interface defines the rules for how external code can interact with that entity

</details>

<details>
<summary>Can you explain when to use is-a-kind relationship and when to use has-a-kind relationship in Java?</summary>

* Is-a relationship implies inheritance. This is used when creating a new entity based on an existing one. It represents a strict hierarchy where the child entity possesses all the characteristics of the parent entity plus its own specific features

* Has-a relationship represents an association between objects, which is often described as a part-whole relationship

</details>

<details>
<summary>What is the difference between composition and aggregation?</summary>

* `Association` refers to the relationship between objects. Composition and aggregation are special cases of the `part-whole` association.
* `Aggregation` assumes that objects are connected in a `part-of` relationship. the parts can exist independently of the whole (like a Department and its Professors).
* `Composition` the parts' lifecycle is strictly managed by the whole; if the whole is destroyed, the parts are destroyed too (like a House and its Rooms).

</details>

<details>
<summary>What are `static` and `dynamic` linking?</summary>

* Attaching a method call to the body of a method is called `binding`. If the binding is done by the compiler (linker) before starting the program, then it is called `static` or `early binding`.

* In turn, `late` binding is binding carried out directly at runtime, depending on the type of object. Late binding is also called `dynamic` or `runtime` binding. In languages that implement late binding, there must be a mechanism for determining the actual type of the object at runtime to invoke the appropriate method. In other words, the compiler does not know the type of the object, but the method invocation mechanism determines it and calls the corresponding method body. `Late-binding` is *language-specific*, but it's easy to assume that some additional information must be included in objects to implement it.

* All Java methods use late (dynamic) binding, unless the method has been declared `final` (private methods are final by default).

</details>

<details>
<summary>`Interface` vs `Abstract` Class?</summary>

* state vs stateless: `Abstract classes` can have instace variables (state), while `interfaces` are essentially stateless - this is a key technical difference.

* Identity vs contract: An abstract class defines what a class is, an interface defines what a class can do. A common way to put it: "Use an abstract class for shared identity, use an interface for shared behavior"

* Java 8+: Since Java 8, interfaces can have default and static methods, which blurs the line a bit — worth mentioning to show you're up to date.

* by extending the abstract class you can not extend another class because Java does not support multiple inheritances but you can implement multiple inheritances in Java.
* An interface defines a contract (what a class can do); an abstract class defines an identity (what a class is). Use an interface when unrelated classes could share behavior (Serializable, Comparable). Use an abstract class when classes share state or a common base implementation (HttpServlet, AbstractList).

</details>

<details>
<summary>`override` vs `overload`?</summary>

* `override` — the ability to override the behavior of a method in descendant types
* `overload` — the ability to override a method with the same name, but with a different set of arguments

</details>

<details>
<summary>Name `Object` methods?</summary>

Are those defined in the root class java.lang.Object. Since every class inherits from Object, these methods are available in any Java object

* toString()
* equals()
* hashCode()
* wait()
* notify()
* notifyAll()
* finalize() — deprecated(Java 9+)
* getClass()

</details>

<details>
<summary>Is it possible to destroy object in Java?</summary>

Discuss GC, destructor, finalize, Runtime.getRuntime().gc(), null references.

In Java, it is not possible to explicitly destroy an object manually.
Instead, Java uses the Garbage Collection (GC), which identifies and removes objects from the heap once they are no longer referenced by the application.

To make an object eligible for destruction, you should mark its reference as null, which signals to the Garbage Collector that the memory can be reclaimed. Although you can request a Garbage Collection cycle by calling Runtime.getRuntime().gc(), there is no guarantee that the JVM will execute the collection immediately

</details>

<details>
<summary>Difference between String/StringBuilder/StringBuffer?</summary>

* `String` — immutable byte array
* `StringBuilder` — helper-class to build strings, thread unsafe
* `StringBuffer` — the same as StringBuilder, thread safe

</details>

<details>
<summary>What are the characteristics of an object?</summary>

* State and Behavior: An object's state is defined by its specific attribute values, while its behavior consists of the methods that operate on those values based on the rules established in its class.
* Memory Location: When an object is created (typically using the new operator), its actual space is allocated in the Heap memory. While the object itself lives in the Heap, a reference to that space is stored in the Stack.
* Visibility and Thread Safety: Because the Heap is a shared area of memory, objects are visible to all threads. Consequently, they are not thread-safe by default and require proper synchronization of code if they are shared.

</details>

<details>
<summary>What is a `constructor` and its role?</summary>

Is a special type of method used to initialize a newly created object. Its primary role is to set the initial state of an object and ensure that it is ready for use immediately after its creation.

Is essential for defining the initial values of an object's attributes.

* *Abstract classes:* Cannot be instantiated directly, it stil contain constructors. These are used to initializate the superclass when a concrete subclass is created.

</details>

<details>
<summary>Can we have a private constructor? Why and when?</summary>

Yes we can have a private contructor.
They are used when you want to control object creation, maintain strict instance limits like in enums or classes that are singleton.

</details>

<details>
<summary>Why all the instance variables have default values in Java?</summary>

In Java, instance variables are assigned default values (such as 0 for numeric types, false for booleans, and null for object references) to ensure program safety and predictability. Because these variables are stored on the Heap, Java automatically initializes them so that every object begins its life in a known, valid state. This prevents the system from reading "garbage" data left over in memory

</details>

<details>
<summary>What are the accessor and mutator methods in Java, and what is their significance?</summary>

* Accessor methods (Getters): These methods are used to read or retrieve the current value of an object's private attribute
* Mutator Methods (Setters): Are used to modify or update the value of an object's attribute, changing its internal state

They are the primary mechanism for Encapsulation. By using mutator methods instead of allowing direct access to variables, a developer can add logic (like validation) to ensure that the object's state remains valid.

</details>

<details>
<summary>What are the different ways to create objects?</summary>

Objects can be created in the following ways:

1. Using the `new` keyword with a constructor
2. Using the `newInstance()` method (reflection)
3. Using the `clone()` method on an existing object
4. Using deserialization
5. Using factory methods or builders

</details>

<details>
<summary>How to create an object using the newInstance method?</summary>

You can create a copy of an existing object through:

1. Shallow copy: Creating a new object and copying all primitive values and references (shallow copy copies references)
2. Deep copy: Creating a new object with completely independent copies of all objects it references
3. Using the `clone()` method if the class implements `Cloneable`
4. Using copy constructors
5. Using serialization and deserialization

</details>

<details>
<summary>How to use the clone method to create a new object?</summary>

The `clone()` method creates a shallow copy of an object. To use it:

1. The class must implement the `Cloneable` interface (marker interface)
2. Override the `clone()` method (inherited from `Object`)
3. Call `super.clone()` within your override
4. Handle the `CloneNotSupportedException` exception

Note: `clone()` performs a shallow copy by default, so you may need to manually handle deep copying of mutable objects.

</details>

<details>
<summary>When does your code throw the CloneNotSupportedException?</summary>

`CloneNotSupportedException` is thrown when:

1. A class does not implement the `Cloneable` interface but attempts to call `clone()`
2. The `clone()` method explicitly throws the exception
3. A class overrides `clone()` and throws the exception under certain conditions

This exception is a checked exception, so it must be caught or declared in the method signature.

</details>

<details>
<summary>How to serialize an object?</summary>

To serialize an object:

1. Implement the `Serializable` interface (marker interface)
2. Use `ObjectOutputStream` to write the object to a stream
3. All referenced objects must also be serializable

Example:
```java
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("file.dat"));
oos.writeObject(myObject);
oos.close();
```

</details>

<details>
<summary>What is `de-serialization`, and how does it generate a new object?</summary>

De-serialization is the process of reconstructing an object from its serialized (binary) form. It:

1. Reads the serialized bytes from a stream using `ObjectInputStream`
2. Reconstructs the object in memory with the same state as when it was serialized
3. Creates a new object instance without calling constructors or initializers (the object state is directly restored from the stream)

Example:
```java
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("file.dat"));
Object myObject = ois.readObject();
ois.close();
```

| Modifier | Class | Package | Subclass | Other |
| :--- | :---: | :---: | :---: | :---: |
| **public** | ✓ | ✓ | ✓ | ✓ |
| **protected** | ✓ | ✓ | ✓ | ✗ |
| **default** | ✓ | ✓ | ✗ | ✗ |
| **private** | ✓ | ✗ | ✗ | ✗ |

</details>

<details>
<summary>Can we apply a private modifier to a class?</summary>

The `private` modifier can be applied to a nested class (a class defined inside another class). A private nested class is only accessible within the scope of its enclosing outer class

It cannot be applied to a top-level class, because if a top-level class were private no other class could see or used, making it useless

</details>

<details>
<summary>How do you explain the term `coupling`?</summary>

It is the degree of dependency between different classes or modules. It measures how much one unit of code relies on the internal details of another to function correctly

</details>

<details>
<summary>How do you explain the term `cohesion`?</summary>

It is the measure of how focused the responsibilities of a single class or module are. A "highly cohesive" class is one where all its methods and attributes are closely related to a single purpose

</details>

<details>
<summary>What do you mean by `loose coupling`?</summary>

This is a design goal where the classes are minimally dependent on one another. This is often achieved by writing code against interfaces (contracts), rather that concrete implementations, which allows you to change one part of the system without breaking others

</details>

<details>
<summary>What do you mean by `tight coupling`?</summary>

This occurs when classes are highly dependent on each other's specific implementation details. In a tightly coupled system, a change to one class frequently requires a "ripple effect" of changes across many other related classes, making the code harder to reuse and maintain

</details>

<details>
<summary>What is `abstract class`?</summary>

It is a class designed to represent a set of common characteristics and behaviors for an entity, serving as a base for other classes to extend

</details>

<details>
<summary>What is evolution?</summary>

It refers to the ability to update and expand a codebase over time without breaking existing functionality

</details>

<details>
<summary>.What is an entity?</summary>

It is the conceptual thing or real-world object that you are trying to represent in your code

</details>

<details>
<summary>Can abstract classes contain constructors?</summary>

Yes. Abstract classes can and should contain constructors. Constructors in abstract classes are used to initialize the state of the parent entity when a concrete subclass is instantiated. Although abstract classes cannot be directly instantiated, their constructors are invoked through the constructor chain when a subclass object is created.

</details>

<details>
<summary>Why do abstract classes provide constructors even though they are not instantiated?</summary>

Abstract classes provide constructors to:

1. Initialize the state of the parent entity
2. Enforce initialization of common attributes across all subclasses
3. Allow code reuse for initialization logic
4. Ensure that when a subclass is instantiated, the parent portion of the object is properly initialized

When a subclass calls `super()` in its constructor, the abstract class's constructor is executed.

</details>

<details>
<summary>What are concrete methods and abstract methods?</summary>

* **Abstract methods**: Methods declared in a class or interface without an implementation (using the `abstract` keyword). They define a contract that must be implemented by subclasses.
* **Concrete methods**: Methods that have a complete implementation. In abstract classes, concrete methods provide default or reusable behavior that all subclasses can use or override.

</details>

<details>
<summary>`default` vs `static`:</summary>

default is about giving implementing classes a ready-to-use or overridable behavior, while static is about utility logic tied to the interface itself, not to any instance.

```java
interface Greeter {
    static void staticHello() {
        System.out.println("Hello from static");
    }

    default void defaultHello() {
        System.out.println("Hello from default");
    }
}

class MyClass implements Greeter { }

// static — called on the interface directly
Greeter.staticHello();

// default — called on the instance, inherited
MyClass obj = new MyClass();
obj.defaultHello();

// ❌ this doesn't work — static is NOT inherited
obj.staticHello();
```

</details>

<details>
<summary>Can we declare all the methods of abstract class as a concrete method?</summary>

Yes, it is possible. An abstract clas is often used when you want to provide a base implementation for code reuse and evolution, even if you don't have any abstract methods to declare yet

</details>

<details>
<summary>Can we have abstract methods as private?</summary>

No, because private methods are considered final by default. Since the purpose of an abstract method is to be overriden by a subclass to provide an implementation, and final methods cannot be overriden, it would be a contradiction.

</details>

<details>
<summary>Can we declare abstract methods as static?</summary>

No, abstract methods rely on dynamic binding, wich is the process of determining which method body to invoke at runtime based on the actual type of the object.

In contrast static methods use static binding, where the method call is attached to the body by the compiler before the program even starts.

Because abstract methods must be implemented by subclasses and resolved at runtime, they cannot be static.

These constraints ensure that the contract defined by the abstract class can be properly fulfilled by its descendants.

</details>

<details>
<summary>What does it mean `final` in Java?</summary>

* The keyword `final` can be applied to class, method, variables(inc. method parameters).
* `final class` = no subclasses, inheritance is forbidden. It is useful to create unmodifiable objects.

Its primary use is to  restrict further modification or inheritance, which is essential for creating stable code and unmodifiable objects.

</details>

<details>
<summary>Can we declare an abstract class as final?</summary>

It is not possible, because abstract class is designed specifically to be extended by other classes to provide, implementation, whereas a final class forbids inheritance.

</details>

<details>
<summary>What is a final method?</summary>

Is a method that cannot be overriden by descendant types

</details>

<details>
<summary>Can we make the reference as final?</summary>

Yes, the final modifier can be applied to variables, wich includes reference variables.

When a reference is declared as final, it means that once the variable has been assigned to an object, it cannot be reassigned to point to a different object or memory location.

It is important to distinguish between the reference and the object itself. While a final reference cannot point to a new object, the internal state of the object it points to can still be changed unless the class itself is designed to be immutable.

</details>

<details>
<summary>What is the difference between variables and reference variables?</summary>

* Variables (primitive types): These variables store the actual value of the data.
* Reference variables: These variables do not store the actual object data. Instead they store a reference (memory address) that points to the location in memory where the object is stored.

</details>

<details>
<summary>Can interfaces have constructors?</summary>

No. Interfaces cannot have constructors. Interfaces are contracts that define behavior, not objects to be instantiated. Since interfaces cannot be instantiated directly, they have no need for constructors.

**Difference from abstract classes:**

* Abstract classes can have constructors to initialize inherited state
* Interfaces cannot have constructors (no state to initialize)
* When a class implements an interface and extends an abstract class, the abstract class constructor is called through `super()`, but the interface has no role in initialization

</details>