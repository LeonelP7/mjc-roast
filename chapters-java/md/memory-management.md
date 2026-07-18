# Memory management

### Heap and Stack

<details>
<summary>What is heap stack?</summary>

The stack is a region of memory of a process, private to each thread that stores method calls and their local data - primitive type values, local variable references and method parameters. It follows **LIFO** (Last In First Out) order - when a method is called a new **frame** (block of memory on the stack for one specific method call) is pushed onto the stack, and when it returns the frame is popped off automatically. If too many nested method calls exhaust the stack, a `StackOverflowError` is thrown.

</details>

<details>
<summary>What is a frame?</summary>

A frame is a dedicated block of memory pushed onto the stack for each method call - it holds that method's local variables, parameters and return address, and is automatically destroyed when the method returns.

When main() calls methodA():

STACK:
┌─────────────────────────┐
│ methodA FRAME           │  ← new frame created
│  - local variables      │
│  - parameters           │
│  - return address       │  ← where to go back when method returns
└─────────────────────────┘
┌─────────────────────────┐
│ main FRAME              │
│  - local variables      │
│  - parameters           │
└─────────────────────────┘

```java
void main() {           // frame 1 created
    int x = 1;          // stored in frame 1
    methodA();          // frame 2 created on top
}

void methodA() {        // frame 2
    int y = 2;          // stored in frame 2
    methodB();          // frame 3 created on top
}

void methodB() {        // frame 3
    int z = 3;          // stored in frame 3
}                       // frame 3 popped — z gone
                        // back to methodA frame 2
```

</details>

<details>
<summary>What is a local variable reference?</summary>

A local variable reference is a variable that stores the address to objects living on the heap rather than storing the object data directly, they're local because they belong to a specific method execution.

```java
void myMethod() {
    int x = 5;           // local primitive — value stored directly on stack
    String name = "Carlos"; // local reference — "name" lives on stack
                            // but the actual String object lives on heap
                            // "name" just holds the heap address
}
// method returns — x and name are gone from stack
// but the String object on heap may still exist if referenced elsewhere
```

</details>

<details>
<summary>What is the heap?</summary>

The heap is the memory region that store objects and their values - created with the new keyword. Unlike the stack which is private to each thread, the heap is shared across all threads, making synchronization necessary when multiple threads access the same object. The heap is managed by the **Garbage Collector** which automatically reclaims memory from objects that are no longer referenced.

```java
// Objects always live on the heap
String name = new String("Carlos"); // "Carlos" object → heap
                                    // "name" reference → stack

int[] numbers = new int[]{1,2,3};  // array object → heap
                                    // "numbers" reference → stack
```

| Feature | Heap |
| :--- | :--- |
| **Scope** | Shared across all threads |
| **Size** | Large, dynamic |
| **Management** | Automatic — Garbage Collector |
| **Speed** | Slower than stack |
| **Error when full** | `OutOfMemoryError` |

When a primitive is a field of an object, it lives on the heap alongside the object it belongs to - not the stack.

HEAP
┌─────────────────────────┐
│ Person object           │
│  name ──────────────────────→ String "Carlos" (also on heap)
│  age = 25               │  ← primitive but lives on heap
└─────────────────────────┘

Primitives live on the stack only when they are local variables inside a method. When they are fields of an object, they live on the heap with the object.

```java
void myMethod() {
    int x = 5;           // local primitive → STACK
    Person p = new Person(); // reference → STACK
                             // Person object → HEAP
                             // p.age = 25 → HEAP (inside the object)
}
```

</details>

<details>
<summary>How can you differentiate between stack memory and heap memory?</summary>

Stack is a memory region that stores local variable references, method calls, their parameters, and local primitive types, the stack is private to each thread. Heap is a memory region that stores all objects and their fields, the heap is shared across all threads so it requires synchronization when multiple threads try to modify the same data. 

| Feature | Stack | Heap |
| :--- | :--- | :--- |
| **Stores** | Method frames, local primitives, references | Objects and their fields |
| **Scope** | Private to each thread | Shared across all threads |
| **Management** | Automatic — push/pop | Garbage Collector |
| **Thread safe** | ✅ Yes | ❌ Requires synchronization |
| **Error when full** | `StackOverflowError` | `OutOfMemoryError` |
| **Size** | Small, fixed | Large, dynamic |
| **Speed** | Fast | Slower |

</details>

<details>
<summary>When does the stack memory gets released?</summary>

Since it stores frames (block of memory allocated for a specific method call) for method calls, the frames are released when methods return, releasing all local variables and references it held. At thread level - when a thread finishes execution the entire stack is released since the stack is private to each thread.

Thread starts → stack created
    main() called → frame pushed
        methodA() called → frame pushed
        methodA() returns → frame popped ← method level release
    main() returns → frame popped
Thread finishes → entire stack released ← thread level release

</details>

<details>
<summary>When does the heap memory gets released?</summary>

This occurs through a process called garbage collection - an automatic process where the JVM identifies objects that are o longer referenced by any part of the program and reclaims their memory. An object becomes eligible for garbage collection when no active references point to it - meaning no thread, variable or other object holds a reference to it anymore.

```java
void myMethod() {
    Person p = new Person("Carlos"); // Person object created on heap
    p = null; // reference dropped — object now eligible for GC
              // no other references exist — heap memory can be reclaimed
}
// even without null assignment — when method returns
// p goes out of scope — object becomes eligible for GC
```

</details>

<details>
<summary>Why memory allocation in stack is faster as compared to heap?</summary>

Stack allocation is faster because it's just a pointer increment with no searching, no synchronization needed since it's thread private, and its data is frequently cached by the CPU - heap allocation requires finding free space of the right size, synchronizing across threads and dealing with **fragmentation**.

Stack — just moves the top pointer up
BEFORE          AFTER new frame
[ frame3 ] ←top    [ new frame ] ←top
[ frame2 ]          [ frame3   ]
[ frame1 ]          [ frame2   ]
                    [ frame1   ]
// one operation — instant

</details>

<details>
<summary>What is fragmentation?</summary>

Fragmentation is when free memory is scattered in small non-contiguous chunks - making it impossible to allocate large objects even when enough total free memory exists. Java's GC handles this through compaction.

**Why it happens:**

Objects are created and destroyed at different times - when objects are garbage collected they leave gaps of different sizes scattered through the heap. New objects may not fit in those gaps perfectly, leaving smaller unusable gaps.

**How Java handles it - compaction:**

The Garbage Collector periodically performs compaction - moving all live objects together to one end of the heap, leaving all free space as one contiguous block.

Before compaction:           After compaction:
[ OBJ ][ FREE ][ OBJ ][ FREE ][ OBJ ] → [ OBJ ][ OBJ ][ OBJ ][ FREE      ]

</details>

<details>
<summary> What exception do you get when stack memory is exhausted?</summary>

`StackOverflowError` is thrown when the stack runs out of space - caused primarily by too many nested method calls, most commonly infinite recursion. Each call adds a new frame to the stack and if methods keep calling each other without returning, frames accumulate until the stack is full. Local variables alone rarely cause this since they all live within a single method's frame.

```java
// ❌ infinite recursion — new frame created on every call, never released
void infinite() {
    infinite(); // calls itself forever — stack fills up with frames
}
// StackOverflowError

// ✅ many local variables — still just one frame, usually fine
void manyVars() {
    int a = 1, b = 2, c = 3; // all in same frame
    String x = "hello";       // reference in same frame
    // no StackOverflowError — just one frame
}
```

</details>

<details>
<summary>What exception do you get when heap memory is exhausted?</summary>

`OutOfMemoryError` is thrown when the heap runs out of space. It has two main causes:

1. Creating objects faster than GC can collect them - the application creates so many objects so quickly that the garbage collector can't reclaim memory enough fast enough to keep up.

2. Memory leaks - objects that are no longer needed but still have active references preventing the GC from collecting them. Over time these accumulate and slowly exhaust the heap.

```java
// Cause 1 — creating too many objects too fast
List<byte[]> list = new ArrayList<>();
while (true) {
    list.add(new byte[1024 * 1024]); // 1MB object every iteration
} // ❌ OutOfMemoryError — heap fills up faster than GC can collect

// Cause 2 — memory leak
static List<Object> cache = new ArrayList<>(); // static — lives forever
void addToCache(Object obj) {
    cache.add(obj); // objects added but never removed
                    // GC can't collect — still referenced by cache
} // ❌ heap slowly fills up over time
```

</details>

<details>
<summary>What is Garbage Collection?</summary>

Garbage Collection is an automatic process in which the JVM detects objects that are no longer being actively referenced and marks them for the garbage collector to reclaim their heap memory. It runs automatically in the background on a daemon thread - triggered by the JVM when the heap space is running low - so developers don't need to manually free memory like in languages such as C or C++.

```java
void myMethod() {
    Person p = new Person("Carlos"); // object created on heap
    p = null; // reference dropped — object eligible for GC

    Person p2 = new Person("Maria"); // object created on heap
} // method returns — p2 goes out of scope — also eligible for GC

// GC runs automatically at some point and reclaims both objects' memory
```

| Phase | What happens |
| :--- | :--- |
| **Mark** | GC identifies all objects still actively referenced |
| **Sweep** | GC reclaims memory from all unreferenced objects |

</details>

<details>
<summary>Explain Garbage Collection cycle?</summary>

The Garbage Collection cycle consists of two main phases - Mark and Sweep - sometimes followed by Compaction:

Phase 1 - Mark:

GC starts from root references (active threads, static variables, local variables on stack) and traverses all object references, marking every object it can reach as alive. Anything not reached is considered garbage.

Root references → Object A → Object B → Object C  (all marked alive ✅)
                → Object D                         (marked alive ✅)

Object E — not reachable from any root             (marked garbage ❌)
Object F — not reachable from any root             (marked garbage ❌)

Phase 2 - Sweep:

GC reclaims memeory from all objects marked as garbage - their heap space is freed.

BEFORE sweep:
[ OBJ A ][ OBJ E ][ OBJ B ][ OBJ F ][ OBJ C ]

AFTER sweep:
[ OBJ A ][ FREE  ][ OBJ B ][ FREE  ][ OBJ C ]

Phase 3 - compaction (optional):

To prevent fragmentation, GC moves all live objects together leaving one contiguous free block.

AFTER compaction:
[ OBJ A ][ OBJ B ][ OBJ C ][ FREE              ]

The stop-the-world problem:

During GC all aplications threads are paused - called stop-the-world. This is why GC can cause application slowdowns.

</details>

<details>
<summary>Can you explicitly request Garbage Collection?</summary>

Yes, you can request Gargabe Collection by calling `System.gc()` or Runtime.`getRuntime().gc()` - but it's just a hint to the JVM, no t a command. The JVM may ignore it entirely or delay itnot guaranteed that it will happen right after you call for it. There is no guarantee it will run immediately or at all.

```java
// Two ways to request GC — both just hints
System.gc();                    // hint to JVM
Runtime.getRuntime().gc();      // equivalent — same hint

// ❌ don't rely on this in production code
// JVM decides when GC actually runs
```

**Why you shouldn't call it in production:**

Manually requesting GC is generally considered bad practice - the JVM's GC algorithms are well optimized and know better than the developer when to run. Forcing it can actually hurt performance by triggering a stop-the-world pause at an inappropiate time.

</details>

<details>
<summary>What is memory leak?</summary>

This occurs when you leave active references to objects that you no longer need or use, preventing the GC from collecting them. Since the GC can only reclaim unreferenced objects, these leaked objects accumulate on the heap over time, slowly consuming memory until eventually causing an OutOfMemoryError.

```java
// Classic memory leak — static collection holds references forever
static List<Object> cache = new ArrayList<>();

void addToCache(Object obj) {
    cache.add(obj);  // objects added but never removed
}                    // GC can't collect — still referenced by cache
                     // heap slowly fills up ❌
```

| Cause | Example |
| :--- | :--- |
| **Static collections** | Objects added but never removed |
| **Unclosed resources** | Streams, connections never closed |
| **Listeners never unregistered** | Event listeners holding references |
| **Long lived objects holding short lived ones** | Cache holding stale objects |

</details>

<details>
<summary>What is Metaspace?</summary>

Java 7 — PermGen (fixed size, on heap):
[ class metadata + static variables ] → OutOfMemoryError: PermGen space ❌

Java 8 — Metaspace (dynamic, native memory):
[ class metadata ] → grows as needed ✅
Static variables → moved to heap

Metaspace is the memory area introduced in Java 8 that stores class metadata - class names, methods definitions, bytecode, and field information. Unlike its predecessor PermGen (Permanent Generation) which had a fixed size, Metaspace uses native memory and grows dynamically - which is why it's said to be virtually unlimited space, bounded only by available system memory.

```java
public class Person {                    // class name "Person" → Metaspace
    
    // field names and types → Metaspace
    private String name;                 // field descriptor → Metaspace
    private int age;                     // field descriptor → Metaspace
    
    // static variable → HEAP (not Metaspace since Java 8)
    static int personCount = 0;
    
    // method definitions and bytecode → Metaspace
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
        personCount++;
    }
    
    public String getName() { return name; } // method definition → Metaspace
    public int getAge() { return age; }      // method definition → Metaspace
}

// Actual object instances → HEAP
Person p1 = new Person("Carlos", 25); // object → Heap
Person p2 = new Person("Maria", 30);  // object → Heap
```

METASPACE                          HEAP
┌─────────────────────┐           ┌──────────────────────────┐
│ class name: Person  │           │ Person{ name="Carlos",   │
│ fields:             │           │         age=25 }         │
│  - name: String     │           │                          │
│  - age: int         │           │ Person{ name="Maria",    │
│ methods:            │           │         age=30 }         │
│  - getName()        │           │                          │
│  - getAge()         │           │ personCount = 2          │
│  - constructor      │           │ (static → heap)          │
└─────────────────────┘           └──────────────────────────┘
loaded once when                  new instance every
class is first used               time new Person() called

Metaspace stores the blueprint - loaded once per class regardless of how many instances exist. Heap stores the instances - one per `new` call. Even if you create a million `Person` objects, `Person`'s metadata in Metaspace is loaded only once. 

</details>

<details>
<summary>What is serialization and deserialization?</summary>

Serialization is the process of converting an object or value into a stream of bytes - allowing it to be saved to a file, sent over a network or stored in a database. Deserialization is the opposite process, converting a stream of bytes into an object. In Java 8 class must implement the `Serializable` marker interface to support serialization.

```java
// Serialization — object to bytes
Person person = new Person("Carlos", 25);
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("person.dat"));
out.writeObject(person); // converts Person object to bytes and saves to file

// Deserialization — bytes back to object
ObjectInputStream in = new ObjectInputStream(new FileInputStream("person.dat"));
Person restored = (Person) in.readObject(); // converts bytes back to Person object
System.out.println(restored.getName()); // "Carlos"
```

| Use case | Example |
| :--- | :--- |
| **Persistence** | Saving object state to a file |
| **Network communication** | Sending objects between services |
| **Caching** | Storing objects in Redis or Memcached |
| **Deep copying** | Serialize then deserialize to create a true copy |

</details>

<details>
<summary>Which interface needs to be implemented for serialization?</summary>

You must implement the marker interface `Serializable` so the JVM knows the class is safe for serialization. Since `Serializable` has no methods to implement it just acts as a tag - without it the JVM throws `NotSerializableException` at runtime when you try to serialize the object.

```java
// ✅ implements Serializable — safe to serialize
class Person implements Serializable {
    String name;
    int age;
}

// ❌ doesn't implement Serializable
class Person {
    String name;
    int age;
}

ObjectOutputStream out = new ObjectOutputStream(...);
out.writeObject(new Person()); // ❌ NotSerializableException at runtime
                               // compiles fine — error only at runtime
```

</details>

<details>
<summary>When a class is truly safe for serialization?</summary>

`Serializable` itself doesn't make a class truly safe, it just tells the JVM to attempt serialization. A class has problems with serialization in these cases:

1. Fields that are no serializable:

```java
class Person implements Serializable {
    String name;          // ✅ String is Serializable
    Thread thread;        // ❌ Thread is not Serializable
                          // NotSerializableException at runtime
}
```

2. Fields referencing non-serializable objects:

```java
class Order implements Serializable {
    Person person;        // ✅ only if Person also implements Serializable
                          // ❌ NotSerializableException if Person doesn't
}
```
Serialization is recursive - when you serialize an object, Java tries to serialize all its fields too. Every object in the graph must be Serializable

</details>

<details>
<summary>How can you prevent a member from serialization?</summary>

Mark fields that souldn't or can't be serialized as transient - they  are skipped diring serialization and set to their default value on deserialization:

```java
class Person implements Serializable {
    String name;              // ✅ serialized
    transient Thread thread;  // ✅ skipped — won't cause exception
    transient String password; // ✅ skipped — sensitive data protected
}
```

</details>

<details>
<summary>What is the difference between JDK, JRE and JVM?</summary>

JVM (Java Virtual Machine) - executes Java bytecode, translating it to machine code the processor understands via JIT compilation, it also manages memory throug Garbage Collection and provides platorm independence - same bytecode runs on any OS that has a JVM.

JRE (Java Runtime Enviroment) - contains the JVM plus the standard Java class libraries needed to run Java programs. Use it when you only need to run Java applications.

JDK (Java Development Kit) - contains the JRE plus development tools like the compiler (`javac`), debugger, and other utilities. Use it when you need to develop Java applications.

JDK
└── JRE
    └── JVM

| Component | Compiles? | Runs? | Develops? |
| :--- | :--- | :--- | :--- |
| **JVM** | ✗ No | ✅ Yes | ✗ No |
| **JRE** | ✗ No | ✅ Yes | ✗ No |
| **JDK** | ✅ Yes | ✅ Yes | ✅ Yes |

</details>

<details>
<summary>What is JIT compilation?</summary>

JIT (Just In Time) compilation is a technique used by the JVM to improve performance. Initially the JVM interprets bytecode line by line - which is slow. The JIT compiler monitors execution and identifies frequently executed code (called hot code) - methods or loops that run many times. It then compiles that hot code directly into native machine code so it runs much faster on subsequent executions instead of beign interpreted every time.

First few executions:
bytecode → JVM interprets line by line → slow but starts immediately

JIT detects hot code (methods called many times)
    ↓
JIT compiles hot code to native machine code

Subsequent executions:
bytecode → native machine code → fast ✅

**Why not compile everytime upfront?**

// Compiling everything upfront (like C++) — long startup time
// JIT approach — start fast, optimize as you go
// best of both worlds — quick startup + optimized hot paths

Summarize:

JIT compilation monitors running bytecode and compiles frequently executed hot code into native machine code - trading a small compilation cost for much faster subsequent executions, giving Java performance close to natively compiled languages.

</details>

<details>
<summary>How heap memory is divided — generations?</summary>



</details>

