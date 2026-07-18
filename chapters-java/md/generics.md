# GENERICS

<details>
<summary>What is `Generics`?</summary>

Generics is a feature introduced in Java 5 that allows you to write classes, interfaces, and methods with type parameters — placeholders for types that are specified later by the caller. The main goal is to achieve type safety at compile time while writing reusable code.

```java
class Box<T> {         // T is the type parameter
    T value;

    Box(T value) { this.value = value; }
    T getValue()  { return value; }
}

Box<String>  stringBox = new Box<>("hello");
Box<Integer> intBox    = new Box<>(42);
```

</details>


<details>
<summary>What are the various benefits that Generics provide to the Java collection framework?</summary>

Before generics all java collections worked with raw `Object` types - meaning you could put anything into a collection, and the compiler had no way to catch type mismatches. Generics solved this at the collection level specifically, in these ways:

```java
// Before generics
List names = new ArrayList();
names.add("Carlos");
names.add(123);                    // no complaint from compiler
String name = (String) names.get(1); // ❌ ClassCastException at runtime

// With generics
List<String> names = new ArrayList<>();
names.add("Carlos");
names.add(123);                    // ❌ compile error — caught immediately
```

</details>


<details>
<summary>What is the meaning of the statement that `Generics is compiler support`? Are Generics not available runtime?</summary>

Generics in Java are implemented via type erasure — the type parameter exists only at compile time for safety checks, and is erased at runtime. So List<String> and List<Integer> are both just List at runtime. This is a key Java-specific detail that distinguishes it from generics in other languages like C#, where the type information is preserved at runtime.

</details>


<details>
<summary>What are the parameter `naming convention` that is generally used in `Generics`?</summary>

| Letter | Stands for | Typical use |
| --- | --- | --- |
| `T` | Type | General purpose, most common |
| `E` | Element | Collections (`List<E>`, `Set<E>`) |
| `K` | Key | Maps (`Map<K, V>`) |
| `V` | Value | Maps (`Map<K, V>`) |
| `N` | Number | Numeric types |
| `S`, `U`, `W` | 2nd, 3rd, 4th types | When multiple type parameters are needed |

</details>


<details>
<summary>What are the advantages of using generic methods and classes?</summary>

- Type safety — compile-time error detection instead of runtime crashes

- Eliminates casting — the compiler already knows the type

- Code reuse — one class or method works for any type instead of duplicating logic per type

- Readability — intent is self-documenting through the type parameter

</details>


<details>
<summary>What are `Type Wildcards` in Generics?</summary>

A wildcard in Generics is represented by the `?` symbol and means "unkown type", it is used whtn you want to write flexible code that works with a family of types, but you dont need to know the specific type directly

</details>

 
<details>
<summary>How are the different `Type Wildcards` used in generics?</summary>

| Wildcard | Syntax | Use when | Can you read/write? |
| --- | --- | --- | --- |
| ✅ Read as Object / ❌ Cannot write | Upper bounded |
| ✅ Read as T / ❌ Cannot write | Lower bounded |
| Unbounded | <?> | Type irrelevant, only using Object methods | <? extends T> | Reading from a structure, type must be T or subtype | <? super T> | Writing into a structure, type must be T or supertype | ⚠️ Read only as Object / ✅ Can write |

</details>


<details>
<summary>What is `automatic type inference` in Generics? What is the `diamond operator` in Generics?</summary>

Type inference is the compiler's ability to automatically determine the type parameter from the context, so you don't have to write it explicitly. Instead of telling the compiler the type twice, it figures it out from the left side of the assignment.

```java
// Without type inference — redundant, you repeat the type twice
List<String> names = new ArrayList<String>();

// With type inference — compiler infers <String> from the left side
List<String> names = new ArrayList<>();
```

The diamond operator is simply the empty angle brackets <> that trigger type inference. It's called "diamond" because of its shape. It was introduced in Java 7.

Type inference has limits - it cant infer when the context is ambiguous:

```java
// ❌ Not enough context — what type should this be?
var list = new ArrayList<>();  // infers ArrayList<Object>, probably not what you want

// ✅ Give it context
var list = new ArrayList<String>(); // now it knows
```

These are two names for essentially the same feature — the diamond operator <> is the syntax, and type inference is the mechanism behind it. If asked separately, make that connection explicit — it shows you understand the why, not just the what.

</details>


<details>
<summary>Explain `Generics` method?</summary>

A generic method is a method that defines its own type parameter, independent of whether the class itself its generic or not. The type parameter is declared before the return type and its resolved at the point of each individual call.

```java
//      ↓ type parameter declaration
public <T> T identity(T value) {
    return value;
}

// Compiler infers T at each call site
String  s = identity("hello");  // T = String
Integer i = identity(42);       // T = Integer
```

</details>


<details>
<summary>Can you add a Generic method to a non-Generic type?</summary>

Yes, you can add a generic method to a non-Generic type:

```java
class Utils {  // non-generic class

    public static <T> void swap(T[] arr, int i, int j) {  // generic method
        T temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```

The type parameter <T> belongs to the method, not the class. The class has no <T> of its own — it has no idea about generics. Each call to the method gets its own independent T resolved at that moment.

</details>


<details>
<summary>Can a generic class have non-generic methods?</summary>

Yes, in a generic class, most methods simply use the class's type parameter without declaring their own, which makes them non-generic methods living inside a generic class.

```java
class Box<T> {
    private T value;

    // non-generic method — uses class T
    public T getValue() { return value; }

    // generic method — declares its OWN type parameter U, independent of T
    public <U> U convert(Function<T, U> converter) {
        return converter.apply(value);
    }
}
```

</details>


<details>
<summary>What are the benefits of defining bounded type as method parameters?</summary>

Bounded type parameters are when you constrain T with extends or super, restricting which types are accepted. The core benefit is that you get both flexibility and safety at the same time — the method works for a family of types, but only types that actually make sense for what the method does.

```java
public static <T extends Number> double sum(List<T> list) {
    double total = 0;
    for (T item : list) {
        total += item.doubleValue(); // ✅ compiler knows T has Number's methods
    }
    return total;
}

sum(List.of(1, 2, 3));      // ✅ Integer extends Number
sum(List.of(1.5, 2.5));     // ✅ Double extends Number
sum(List.of("a", "b"));     // ❌ compile error — caught immediately
```

The & operator lets you require multiple contracts simultaneously — a detail interviewers occasionally probe.

```java
// T must implement both Comparable and Serializable
public static <T extends Comparable<T> & Serializable> T clampAndSave(T value) {
    ...
}
```

</details>


<details>
<summary>How’s compile type safety enforced by Generics?</summary>

Generics enforce type safety by embedding type information into the code at compile time, allowing the compiler to verify that every operation on a typed variable is legal for that specific type - before the program ever runs.

```java
List<String> names = new ArrayList<>();

names.add("Carlos");   // ✅ String goes in
names.add(42);         // ❌ compile error — Integer violates the contract

String name = names.get(0);  // ✅ compiler knows get() returns String
Integer i   = names.get(0);  // ❌ compile error — can't assign String to Integer
```

</details>


<details>
<summary>What are `reifable` types?</summary>

A reifiable type is a type whose complete type information is fully available at runtime. Because Generics use type erasure, generic type parameters are stripped out after compilation — so most generic types are not reifiable. Reifiable types are the ones that survive erasure intact.

Reifiable means "can be fully inspected at runtime" — if type erasure touches it, it's not reifiable. If the type survives erasure completely intact, it is.

```java
// Primitives — always fully known at runtime
int, double, boolean, char...

// Raw types — no generics at all
List, Map, ArrayList...

// Non-generic classes and interfaces
String, Integer, Object...

// Unbounded wildcards — ? carries no specific type to erase
List<?>, Map<?, ?>...

// Arrays of reifiable types
String[], int[], List<?>[]...
```

</details>


<details>
<summary>What are `non-reifable` types?</summary>

Because Generics use type erasure, generic type parameters are stripped out after compilation — so most generic types are not reifiable.

```java
List<String>      // becomes just List at runtime
Map<String, Integer> // becomes just Map at runtime
List<? extends Number> // partially erased
T                 // completely erased, becomes Object
```

```java
List<String> names = new ArrayList<>();

// ✅ works — List is reifiable (raw type)
if (names instanceof List) { ... }

// ❌ compile error — List<String> is not reifiable,
//    the <String> is gone at runtime, nothing to check against
if (names instanceof List<String>) { ... }

// ✅ works — unbounded wildcard is reifiable
if (names instanceof List<?>) { ... }
```

</details>


<details>
<summary>What is `type erasure`?</summary>

Generics are a compile-time only feature. After the compiler finishes checking everything, it erases the type parameters — this is called type erasure. At runtime, List<String> and List<Integer> are both just List.

</details>


<details>
<summary>What would happen if type erasure is not there?</summary>

Type erasure exists entirely for backwards compatibility — without it, Java 5 Generics would have broken every existing Java program and required a completely new JVM. The limitations it introduces are the price paid for a smooth migration path.

```java
// Pre-Java 5 code everywhere in production
List names = new ArrayList();
Map config = new HashMap();
```

</details>


<details>
<summary>Why can’t you assign a Generic collection object of sub type to a Generic collection object of super type?</summary>

Even though Integer extends Number, List<Integer> does not extend List<Number>. This is called generic invariance and it's intentional — allowing it would silently break type safety.

This behavior is called invariance — List<Integer> and List<Number> are treated as completely unrelated types, even though Integer and Number have a parent-child relationship. Generic collections are invariant by default in Java.

```java
List<Integer> ints = new ArrayList<>();
List<Number> numbers = ints;  // pretend this was allowed

numbers.add(3.14);  // Double is a Number, so this looks legal...
                    // but ints is actually a List<Integer>!

Integer i = ints.get(0); // ❌ ClassCastException — 3.14 is not an Integer
```

You'd have a Double sitting inside a List<Integer> with no warning — exactly the kind of runtime crash Generics exist to prevent. So the compiler blocks the assignment entirely.

</details>


<details>
<summary>Why it’s allowed to assign an array object of sub type to an array object of super type? Are Java array more polymorphic than Generics?</summary>

Technically true but practically a flaw — array covariance exists for historical reasons and leads to runtime exceptions that Generics were specifically designed to eliminate.

</details>


<details>
<summary>Explain `co-variance`?</summary>

Covariance means the subtype relationship between types is preserved when they are wrapped in a container — arrays are covariant by default in Java, generics are not, and wildcards offer a controlled safe version of covariance.

Covariance is when a container or return type preserves the subtype relationship of its elements. Meaning if `Integer exteds Number`, then `Container<Integer>` is also treated as a subtype of `Container<Number>`

```java
// This is plain polymorphism — not covariance
Number n = new Integer(42); // Integer IS-A Number, direct assignment

// This is covariance — the relationship is PRESERVED through a container
Number[] numbers = new Integer[3]; // Integer[] IS-A Number[] — array is covariant
```

</details>


<details>
<summary>Why can’t you add an element of subtype to a generic defined with `Upper Bound` type wildcard?</summary>

Because the compiler can't guarantee which specific subtype the  list holds at runtime, so adding anything would risk corrupting it

Example:
When you declare List<? extends Number>, you're telling the compiler "this is a list of some specific subtype of Number, but I don't know which one":

```java
List<? extends Number> list = new ArrayList<Integer>(); // could be Integer
List<? extends Number> list = new ArrayList<Double>();  // could be Double
List<? extends Number> list = new ArrayList<Float>();   // could be Float

list.add(42);   // ❌ what if list is actually a List<Double>?
list.add(3.14); // ❌ what if list is actually a List<Integer>?
```

It can't allow any addition because it genuinely doesn't know which specific type is safe to insert — allowing it would risk the exact runtime corruption Generics exist to prevent.

`? extends` — you can read safely as `Number`, but you can never write. The list is effectively read-only.

</details>


<details>
<summary>Explain `contra-variance`?</summary>

`contra-variance` is the opposite of covariance - instead of preserving the subtype relationship through a container, it reverses it. If `Integer extends Number`, a contra-variant container of `Number` is treated as a supertype of a container of `Integer`.

In java this is expressed with lower bounded wildcards `<? super T>`:

```java
List<? super Integer> list = new ArrayList<Integer>(); // ✅
List<? super Integer> list = new ArrayList<Number>();  // ✅
List<? super Integer> list = new ArrayList<Object>();  // ✅

List<? super Integer> list = new ArrayList<Double>();  // ❌ Double is not a supertype of Integer
```

And PECS ties it all together:

Producer Extends → covariance → read only
Consumer Super → contra-variance → write only

The list produces data for you → extends. The list consumes data you give it → super.

</details>


<details>
<summary>Notes about `covariance` and `contra-variance`:</summary>

Covariance `<? extends Number>`:

"This list holds some specific subtype of Number, but I don't know which one"

- Could be `List<Integer>`, `List<Double>`, `List<Float>`...
- Safe to read as `Number` — whatever it holds, it's guaranteed to be a `Number`
- Unsafe to write — compiler doesn't know which specific subtype it is

Contra-variance `<? super Integer>`:

"This list holds Integer or some supertype of Integer, but I don't know which one"

  - Could be List<Integer>, List<Number>, List<Object>...
  - Safe to write Integer — whatever the list is, Integer will always fit in it
  - Unsafe to read specifically — could come out as Number or bare Object, compiler can't guarantee which

</details>


<details>
<summary>Why can’t you read element from a Generic defined with lower bound type wildcard?</summary>

Because the complier can't guarantee the type of the element that will come out, so you can only read as `Object`.

Because <? super Integer> could be a List<Integer>, List<Number>, or List<Object> — the compiler doesn't know which specific supertype it is at compile time. Since the actual type could be anything up the hierarchy, the only type the compiler can safely guarantee for a returned element is Object — the top of every hierarchy in Java. 

</details>


<details>
<summary>When should you use `co-variance`?</summary>

Use it when you only need to read from a collection and you want to accept multimple subtypes as input - typically in methods parameters that process data without modifiying the collection

```java
// Works for List<Integer>, List<Double>, List<Float> — any subtype of Number
static double sum(List<? extends Number> list) {
    double total = 0;
    for (Number n : list) {       // ✅ safe to read as Number
        total += n.doubleValue();
    }
    return total;
}
```

</details>


<details>
<summary>When should you use `contra-variance`?</summary>

Use it when you only need to write into a collection and you want to accept multiple supertypes as a destination - typically in methods parameters that receive or collect data.

Use it when you only need to write into a collection, and you want the method to accept not just List<Integer> but also List<Number> or List<Object> — any list that is "wide enough" to hold an Integer.

```java
// Works for List<Integer>, List<Number>, List<Object> — any supertype of Integer
static void fill(List<? super Integer> list) {
    for (int i = 0; i < 5; i++) {
        list.add(i);              // ✅ safe to write Integer into any supertype
    }
}
```

</details>


<details>
<summary>Example of PECS:</summary>

```java
// Copying from a source into a destination
static <T> void copy(List<? extends T> source,   // reading — covariance
                     List<? super T> destination) { // writing — contra-variance
    for (T element : source) {
        destination.add(element);
    }
}

List<Integer> source      = List.of(1, 2, 3);
List<Number>  destination = new ArrayList<>();
copy(source, destination); // ✅
```

This is actually how Collections.copy() is implemented in Java's own standard library — it's the most complete real-world example of PECS in action.

The intuition is:
covariance — the collection is the source, data flows out of it → extends
contra-variance — the collection is the destination, data flows into it → super

Which maps back to PECS cleanly:
- Producer = source = data flows out = extends
- Consumer = destination = data flows in = super

</details>


<details>
<summary>Is It Possible to Declared a `Multiple Bounded Type Parameter` (e.g. Cage<T extends Animal & Comparable>)?</summary>

Yes, it is possible. Multiple bounds are declared with the & operator and mean that T must satisfy all the specified constraints simultaneously — not just one of them.

```java
class Cage<T extends Animal & Comparable<T> & Serializable> {
    private T occupant;

    void place(T animal)  { this.occupant = animal; }
    T    get()            { return occupant; }

    boolean isLargerThan(T other) {
        return occupant.compareTo(other) > 0; // ✅ Comparable is guaranteed
    }
}
```

Two rules worth remembering fo the interview:

1 At most one class, the rest must be interfaces:
```java
// ✅ valid — Animal is a class, the rest are interfaces
T extends Animal & Comparable<T> & Serializable

// ❌ invalid — can't have two classes (Animal and Pet)
T extends Animal & Pet & Comparable<T>
```

2 The class must come first:
```java
// ✅ valid — class first, interfaces after
T extends Animal & Comparable<T>

// ❌ invalid — interface before class
T extends Comparable<T> & Animal
```

Why would you actually use this?
When your method or class genuinely needs capabilities from multiple contracts — for example a Cage that needs its occupant to be an Animal (domain type), Comparable (to sort or rank animals), and Serializable (to save state). Each bound unlocks that type's API inside the class.

</details>


## PRACTICAL TASKS: Generics

<details>
<summary>Can we do the following</summary>

```java
public class Test {

    public static void main(String... args) {
        List<Object> list = new ArrayList<String>();
        List<? extends Object> list = new ArrayList<String>();
        List<? super String> list = new ArrayList<Object>();
    }
}
```

</details>


<details>
<summary>Generics basics</summary>

```java
class Vehicle {}
class Car extends Vehicle {}
class Bike extends Vehicle {}
public class NodeId<T> {
    private final T Id;

    public NodeId(T id) {
        this.Id = id;
    }
    public T getId() {
        return Id;
    }
}
public interface List<E> {
    void add(E x);
    Iterator<E> iterator();
}

public class Test {
    public static void main(String... args) {
        List<Integer> integerList = new ArrayList<Integer>(); //L1

        NodeId<Object> objectNodeId = new NodeId<>(new Object()); //L2
        // Parameter Type - Integer
        NodeId<Integer> integerNodeId = new NodeId<>(1); //L3
        // what is happening here?
        objectNodeId = integerNodeId; //L4
    }

}
```

</details>



<details>
<summary>Type Wildcards</summary>

```java
class Vehicle {}
class Car extends Vehicle {}
class Bike extends Vehicle {}

public class Test {
    public static void main(String... args) {
        List<Integer> integerList = new ArrayList<>();
    }

    public void addVehicles1(List<?> vehicles) {
        // do something
    }
    public void addVehicles2 (List<? extends Truck> vehicles) {
        // do something
    }
    public void addIds1(List<? extends Number> T) {}
    public void addIds2(List<? super Float> T) {}

}
```

</details>



<details>
<summary>Java Generics vs Java Array</summary>

```java
class Vehicle {}
class Car extends Vehicle {}
class Bike extends Vehicle {}

public class Test {
    public static void main(String... args) {
        // Generics
        List<Bike> bikes = new ArrayList<Bike>(); //L1

        List<Vehicle> vehicles = bikes; //L2
        vehicles.add(new Car()); //L3

        Bike bike = vehicles.get(0); //L4

        // Arrays
        Car[] cars = new Car[3];  //L5
        Vehicle [] vehicles = cars; //L6
        vehicles[0] = new Bike(); //L7
    }
}
```

</details>


<details>
<summary>Co-variance</summary>

```java
class Vehicle {}
class Car extends Vehicle {}
class Bike extends Vehicle {}

public class Test {
    public static void main(String... args) {
        List<? extends Vehicle> vehicles = new ArrayList<Bike>();
        Vehicle vehicle = vehicles.get(0);
        vehicles.add(new Bike());
    }
}
```

</details>


<details>
<summary>Contra-variance</summary>

```java
class Vehicle {}
class Car extends Vehicle {}
class Bike extends Vehicle {}

public class Test {
    public static void main(String... args) {
        List<? super Car> cars = new ArrayList<Vehicle>();
        cars.add(new Car());
        Car vehicle = cars.get(0);
    }
}
```

</details>
