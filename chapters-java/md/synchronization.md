# Synchronization

<details>
<summary>Explain `concurrent` processing?</summary>

Concurrent processing is a technique for executing multiple tasks by rapidly switching between them - on a single core CPU only one task executes at time, but the switching is fast enough to give the illusion of simultaneity. It improves responsiveness - allowing a program to handle multiple operations like processing requests or keeping UI responsive while background work runs.

Single core — concurrent (taking turns):
Time: 1ms    2ms    3ms    4ms    5ms
CPU:  Task1  Task2  Task1  Task2  Task1  ← switching between tasks

Concurrency is about dealing with multiple tasks at once - paralellism is about multiple tasks at once on multiple cores.

</details>

<details>
<summary>Explain `parallel` processing?</summary>

Parallel processing is a technique for executing multiple tasks simultaneously by dividing them across multiple CPU cores, meaning no need to take turns like in concurrent processing.

Multiple cores — parallel:
Core 1: Task1  Task1  Task1  Task1  ← truly simultaneous
Core 2: Task2  Task2  Task2  Task2

Parallelism is a form of concurrency - concurrency just means "multiple tasks making progress at the same time". Parallelism satisfies that definition, just with truly simultaneous execution instead of switching. So parallelism is a subset of concurrency, not a separate thing.

Concurrency (broad concept)
    ├── Single core — fake simultaneity through switching
    └── Parallelism — true simultaneity on multiple cores

In practice even with multiple cores, there are usually far more tasks than cores. Each core handles its own concurrency through switching - but across cores, tasks run truly in parallel. So in practice most systems use both concurrency and paralellism simultaneously.

4 cores, 100 tasks:
Core 1: Task1 → Task5 → Task9...  ← still switches between tasks
Core 2: Task2 → Task6 → Task10... ← still switches between tasks
Core 3: Task3 → Task7 → Task11... ← still switches between tasks
Core 4: Task4 → Task8 → Task12... ← still switches between tasks

</details>

<details>
<summary>Explain `synchronous` processing?</summary>

Synchronous processing means the running process waits for each task to finalize before starting the next - when doing calls it waits until it gets a response or a result before continuing any other work.

```java
// Synchronous — each call blocks until complete
String data = fetchDataFromApi(); // blocks here until response arrives
processData(data);                // only starts after fetchData completes
saveToDatabase(data);             // only starts after processData completes

// if fetchData takes 5 seconds — everything waits 5 seconds
```

The tradeoff:

Synchronous processing is simple and predictable - tasks execute in clear order. The downside is that slow operations like network calls or file reading block everything else, potentially wasting CPU time just waiting.

</details>

<details>
<summary>Explain `asynchronous` processing?</summary>

Asynchronous processing means the running process doesn't wait for each task to finalize before starting the next - it continues working under the promise that other tasks will complete eventually. In Java this is achieved through `Future`, callbacks or `CompletableFuture` - allowing the main thread to submit a task and continue doing other work while waiting for the result.

| Feature | Synchronous | Asynchronous |
| :--- | :--- | :--- |
| **Waiting** | Blocks until done | Continues immediately |
| **Simplicity** | ✅ Simple and predictable | ❌ More complex |
| **Performance** | ❌ Wastes CPU waiting | ✅ Better resource usage |
| **Use when** | Order matters, simple flow | I/O bound, network calls |

</details>

<details>
<summary>Explain the difference between `concurrent` and `parallel` processing?</summary>

Concurrent processing refers to multiple tasks making progress at the same time, on a single-core CPU is achieved through the quick switching between tasks to give the illusion of simultaneity and improve responsiveness, beign able to handle multiple tasks. Parallel processing is  multiple tasks running truly at the same time by spreading them across multiple processor cores - no switching needed, genuine simultaneity.

</details>

<details>
<summary>Explain the difference between `synchronous` and `asynchronous` processing?</summary>

On a synchrous processing the main process executes each task one after another, waiting until one finalize before starting the next - when doing calls it waits (block) until getting an answer or result before doing any other work. Asynchronous processing doesn't wait for each task to finalize, it keeps working under the promise that the others tasks will eventually complete, making it more effient for I/O bound operations like network calls or file reading where blocking would waste CPU time.

</details>

<details>
<summary>Does `parallel` processing require multiple threads?</summary>

Yes - parallel processing requires multiple threads to run those tasks individually, and also need multiple CPU cores for those threads to run simultaneously. Without multiple cores, threads can only take turns on a single core - achieving concurrency but not true paralellism. Without multiple threads there is nothing to distribute across the cores.

Multiple threads + Multiple cores = True parallelism ✅
Multiple threads + Single core   = Concurrency only ✅
Single thread   + Multiple cores = No parallelism ❌ — nothing to distribute

</details>

<details>
<summary>What is `critical section`?</summary>

A critical section is a block or code that touches shared mutable state - where a race condition could occur if multiple threads execute it simultaneosly - leaving it unprotected risks race conditions and data corruption. Critiical sections must be protected with syncronization mechanisms like `synchronized` or `ReentrantLock` to ensure only one thread executes them at a time.

```java
class Counter {
    int count = 0;

    void increment() {
        // ⚠️ critical section — shared mutable state accessed here
        count++; // read, modify, write — race condition possible
    }

    // ✅ protected critical section
    synchronized void increment() {
        count++; // only one thread can execute this at a time
    }
}
```

</details>

<details>
<summary>What is `thread synchronization`?</summary>

Thread synchronization is the set of mechanisms that allows you to:
* Control access to shared resources - ensuring only one thread executes a critical section at a time.

* Coordinating thread execution order - making threads wait for each other at certain points.

In java this is achieved through `synchronized`, `ReentranLock`, `wait()`/`notify()`, and `join()`.

```java
// Controlling access — synchronized, ReentrantLock
synchronized void increment() { count++; }

// Coordinating order — wait/notify, join, CountDownLatch
thread.join();      // wait for thread to finish
object.wait();      // wait for notification
object.notify();    // signal waiting thread
```

</details>

<details>
<summary>What is the difference between `synchronized method` and `synchronized block`?</summary>

The main difference between them is when a thread executes a synchronized method, all the others have to wait until the entire method finishes - the lock is implicitly on `this`. A synchronized block gives you finer control - you choose the exact amount of code that needs synchronization and which lock object to use. You should always aim to avoid making threads wait longer than needed, keeping performance as high as possible.

```java
// Synchronized method — locks on 'this' implicitly
public synchronized void method() {
    // entire method locked on 'this'
}

// Synchronized block — you choose the lock object
public void method() {
    // multiple threads can be here

    synchronized (this) {        // lock on 'this'
        // critical section only
    }

    // multiple threads can be here again
}
```

</details>

<details>
<summary>Whats the difference between locking with this and with any other object, how i know which to use?</summary>

**Locking on `this` the entire instance is the lock:**

```java
class BankAccount {
    double balance;
    
    synchronized void deposit(double amount) {  // locks on 'this'
        balance += amount;
    }
    
    synchronized void withdraw(double amount) { // locks on 'this'
        balance -= amount;
    }
}
```

If one thread is in `deposit()`, no other thread can enter `withdraw()` either - both methods share the same lock (`this`). Good when all methods access the same shared state.

**Locking on a dedicated object - more granular control:**

```java
class BankAccount {
    double balance;
    String ownerName;
    
    private final Object balanceLock = new Object();
    private final Object nameLock = new Object();
    
    void deposit(double amount) {
        synchronized (balanceLock) { // only locks balance operations
            balance += amount;
        }
    }
    
    void updateName(String name) {
        synchronized (nameLock) { // only locks name operations
            ownerName = name;
        }
    }
}
```

Now `deposit()` and `updateName()` can run simultaneously - they protect different resources with different locks. Better performance when methods access independent shared state.

| Use When | Lock Object |
| :--- | :--- |
| **All methods protect the same shared state** | `this` — simple and straightforward |
| **Methods protect independent state** | Dedicated object — need finer granularity for better performance |
| **You need to synchronize across multiple classes on the same lock** | External object |

In summary:

Lock on `this` when all synchronized code protects the same resource - use dedicated lock objects when different methods protect independent resources and you want them to urn cocurrently without blocking each other.

</details>

<details>
<summary>why cant be fields their own locks?</summary>

* The locks can't be null:

```java
String ownerName = null; // initially null

synchronized (ownerName) { // ❌ NullPointerException — can't lock on null
    ownerName = "Carlos";
}
```

* The reference could change:

```java
String ownerName = "Carlos";

synchronized (ownerName) { // locks on "Carlos" string object
    ownerName = "Maria";   // now ownerName points to a different object
}

// another thread
synchronized (ownerName) { // locks on "Maria" — different lock than before!
                           // both threads could be in critical section simultaneously
}
```

When  you reassign the field, the lock object changes - two threads could end up locking on different object, completely defeating the purpose of synchronization.

**That's why you use a dedicated `final` lock object:**

`final` guarantees the reference never changes - every thread always lock on the exact same object, no matter what happens to the field beign protected.

Lock objects should always be `private final` - private so no outside code can interfere with your lock, and final so the reference never changes.

</details>

<details>
<summary>Why synchronized block is preferred to synchronized method?</summary>

Because synchronized block gives you finer control, making you able to choose which object is the lock and the exact amount of code that needs to be synchronized, so you don't make other threads wait more than necessary, keeping performance as high as possible.

</details>

<details>
<summary>Does `synchronized` method locks the entire object?</summary>

Yes - a synchronized method uses `this` as the lock implicitly, so while one thread executes it no other thread can enter any other synchronized method on the same instance. However non-synchronized methods remain freely accessible - `synchronized` doesn't truly lock the entire object, only the synchronized entry points.

```java
class Counter {
    int count = 0;

    synchronized void increment() { count++; } // locked
    synchronized void decrement() { count--; } // locked — shares same lock

    void printCount() { System.out.println(count); } // NOT locked
                                                      // any thread can call this
                                                      // even while increment() is running
}
```

</details>

<details>
<summary>What is `volatile` field?</summary>

A `volatile` field guarantees that any write to it is immediately visible to all threads. Without volatile, threads my cache a variable locally for performance - meaning a change made by one thread might not be visible for others reading from their own cache. Marking field as `volatile` forces every read to come from main memory and every write go directly to main memory, bypassing the CPU cache.

```java
class Task implements Runnable {
    // ❌ without volatile — thread may read stale cached value
    boolean running = true;

    // ✅ with volatile — change immediately visible to all threads
    volatile boolean running = true;

    public void run() {
        while (running) { // always reads fresh value from main memory
            doWork();
        }
    }
}

// From another thread
task.running = false; // ✅ with volatile — task thread sees this immediately
```

</details>

<details>
<summary>When use `volatile`?</summary>

Don't make everything `volatile` - it hurts performance by bypassing CPU cache, and it doesn't solve race conditions on compound operations anyway. Use `volatile` only for simple flags where visibility is the only concern, and `synchronized` or atomic classes when atomicity is needed.

* Performance cost:
Every `volatile` read goes to main memory instead of the fast CPU cache, main memory is significantly slower. Making every variable `volatile` would eliminate the performance benefit of the CPU caching entirely.

* Volatile doesn't guarantee atomicity:
`volatile` only guarantees visibility, not that compound operations are atomic. This is exactly `count++` problem.

```java
volatile int count = 0;

// Thread 1 and Thread 2 both call this simultaneously
void increment() {
    count++; // still 3 steps — read, modify, write
             // volatile guarantees visibility of each step
             // but doesn't prevent interleaving between steps
}

// Thread 1 reads count = 0
// Thread 2 reads count = 0  ← both read same value
// Thread 1 writes count = 1
// Thread 2 writes count = 1  ← Thread 1's update lost
```

| Problem | Solution |
| :--- | :--- |
| **Visibility only** — simple flag | `volatile` |
| **Atomicity** — compound operations | `synchronized` or `AtomicInteger` |
| **Both visibility and atomicity** | `synchronized` or atomic classes |

</details>

<details>
<summary>Explain the problem volatile solves?</summary>

`volatile` solves visibility across multiple threads when one makes changes on one variable, making immediately visible to all others.

</details>

<details>
<summary>What is the purpose of join method?</summary>

The `join()` method is a tool for thread synchronization that makes the calling thread wait until the target thread finishes execution before continuing. It's used when you need to ensure a thread has completed its work before processing its results.

```java
Thread t = new Thread(() -> {
    System.out.println("A"); // step 1
    heavyComputation();
});

t.start();
System.out.println("B"); // step 2 — runs immediately, doesn't wait

t.join(); // main thread waits here until t finishes

System.out.println("C"); // step 3 — guaranteed to run after t completes

// Possible output:
// B
// A
// C ← always last, guaranteed by join()
```

Common use case - waiting for results:

```java
Thread fetchThread = new Thread(() -> fetchData());
fetchThread.start();

fetchThread.join(); // wait for data to be fetched

processData(); // ✅ safe — fetch guaranteed complete
```
In summary:
`join()` makes the calling thread pause until the target thread finishes — guaranteeing execution order when you need one thread's results before another can proceed.

</details>

<details>
<summary>What are atomic classes?</summary>

Atomic classes are thread safe classes that wrap primitive types and provide compound operations - like increment, decrement, and compare-and-set - that execute atomically as a single indivisible unit. No other thread can interrupt or interleave in the middle of the operation, eliminating race conditions without needing synchronized.

```java
// ❌ not atomic — race condition possible
int count = 0;
count++; // 3 steps — can be interrupted between them

// ❌ volatile — visibility only, still not atomic
volatile int count = 0;
count++; // still 3 steps

// ✅ atomic — compound operation executes as one indivisible unit
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet(); // read + increment + write — atomic, no race condition
```

| Class | Wraps |
| :--- | :--- |
| **AtomicInteger** | `int` |
| **AtomicLong** | `long` |
| **AtomicBoolean** | `boolean` |
| **AtomicReference<T>** | Any object reference |

</details>

<details>
<summary>How works `AtomicReference`?</summary>

`AtomicReference` only works like this:

```java
class BankAccount {
    String owner;
    double balance;
    
    BankAccount(String owner, double balance) {
        this.owner = owner;
        this.balance = balance;
    }
}

// AtomicReference with your custom class
AtomicReference<BankAccount> account = new AtomicReference<>(
    new BankAccount("Carlos", 1000.0)
);

// atomically read
BankAccount current = account.get();

// atomically update
account.set(new BankAccount("Carlos", 2000.0));

// atomically compare and swap
BankAccount oldAccount = account.get();
BankAccount newAccount = new BankAccount("Carlos", oldAccount.balance + 500);
account.compareAndSet(oldAccount, newAccount); // ✅ only updates if still same reference
```

`AtomicReference` only makes the reference swap atomic - it doesn't make the object's internal fields atomic. If multiple threads modify the object's fields directly you still have a race condition.

```java
// ✅ atomic — swapping the reference
account.set(new BankAccount("Carlos", 2000.0));

// ❌ not atomic — modifying fields directly
account.get().balance += 500; // race condition — balance is not atomic
```

With `AtomicReference` always create a new object with the updated state and swap the reference - never modify the existing object's fields directly.

The most powerful method — `compareAndSet()`:

Updates the reference only if it still has the expected value — like a conditional atomic swap:

```java
AtomicReference<String> name = new AtomicReference<>("Carlos");

// "if the current value is still Carlos, update to Maria"
boolean updated = name.compareAndSet("Carlos", "Maria"); // ✅ true — updated
boolean updated = name.compareAndSet("Carlos", "Juan");  // ❌ false — already "Maria"

// Practical example

AtomicReference<String> status = new AtomicReference<>("PENDING");

// Only one thread can successfully transition from PENDING to PROCESSING
if (status.compareAndSet("PENDING", "PROCESSING")) {
    processOrder(); // ✅ only one thread gets here
} else {
    System.out.println("Already being processed");
}
```

`AtomicReference<T>` provides atomic reads and updates for object references — most useful when you need to atomically check and update a reference based on its current value using compareAndSet().

</details>

<details>
<summary>How to exit gracefully from ExecutorService?</summary>

Use `shutdown()` for gracefully exit - it waits for all tasks to complete before stopping. Use `shutdownNow()` when you need to stop immediately - it interrupts running tasks and returns unstarted ones. The complete pattern combines both with a timeout.

```java
ExecutorService pool = Executors.newFixedThreadPool(4);

// Option 1 — graceful shutdown
pool.shutdown(); // stops accepting new tasks
                 // waits for running and queued tasks to finish
                 // ✅ preferred — no tasks are lost

// Option 2 — immediate shutdown
pool.shutdownNow(); // stops accepting new tasks
                    // attempts to cancel running tasks via interrupt()
                    // returns list of queued tasks that never started
                    // ⚠️ use when you need to stop immediately
```

**The complete graceful shutdown pattern:**

```java
pool.shutdown(); // stop accepting new tasks
try {
    // wait up to 10 seconds for running tasks to finish
    if (!pool.awaitTermination(10, TimeUnit.SECONDS)) {
        pool.shutdownNow(); // force stop if tasks take too long
    }
} catch (InterruptedException e) {
    pool.shutdownNow(); // force stop if interrupted while waiting
}
```

</details>

<details>
<summary>`notify` vs `notifyAll`:</summary>

`notify()` wakes up one thread if there are multiple threads waiting the JVM chooses which one - you can't control it. `notifyAll()` wakes up all waiting threads at once, letting them complete for the lock. `notifyAll()` is generally pereferred because using `notify()` with multiple waiting threads risk starvation - the wrong thread gets nofied while the other wait indefinitely for a notification that will never come.

Three rules to remember about `wait()` and `notify()`:

```java
// 1. Must always be called inside synchronized block
synchronized (lock) {
    lock.wait();    // ✅
    lock.notify();  // ✅
}
lock.wait(); // ❌ IllegalMonitorStateException

// 2. Always use while loop, not if, for wait()
while (condition) {  // ✅ rechecks condition after waking
    wait();
}
if (condition) {     // ❌ dangerous — condition may have changed
    wait();
}

// 3. wait() releases the lock, notify() doesn't
synchronized (lock) {
    lock.wait();    // releases lock while waiting
    lock.notify();  // keeps lock until synchronized block exits
}
```

Example:
producer-consumer scenario:

```java
class SharedQueue {
    private List<Integer> queue = new ArrayList<>();
    private int maxSize = 5;

    // Producer — adds items to queue
    synchronized void produce(int item) throws InterruptedException {
        while (queue.size() == maxSize) {
            wait(); // queue full — wait for consumer to remove items
        }
        queue.add(item);
        System.out.println("Produced: " + item);
        notifyAll(); // wake consumer — item available
    }

    // Consumer — removes items from queue
    synchronized int consume() throws InterruptedException {
        while (queue.isEmpty()) {
            wait(); // queue empty — wait for producer to add items
        }
        int item = queue.remove(0);
        System.out.println("Consumed: " + item);
        notifyAll(); // wake producer — space available
        return item;
    }
}
```
Why notifyAll() instead of notify() here:

```java
// If you used notify() with multiple producers and consumers:
// Producer 1 adds item → notify() → wakes Producer 2 instead of Consumer
// Producer 2 sees queue full → waits again
// Consumer never woken → starved indefinitely ❌

// With notifyAll():
// Producer 1 adds item → notifyAll() → wakes everyone
// Consumer sees item available → consumes it ✅
// Producers see queue full → wait again ✅
```

</details>

<details>
<summary>ReentrantLock and tryLock():</summary>

`ReentrantLock` is a more flexible alternative to `sychronized` that gives you explicit control over acquiring and releasing locks - `tryLock()` prevents indefinite waiting and potentital deadlocks by attempting to acquire the lock and returning immediately if unavailable, letting the thread decide what to do instead of blocking forever.

```java
// synchronized — no way out if lock is never released
synchronized (lock) { // waits forever ❌
    // critical section
}

// ReentrantLock with tryLock() — you decide what to do if lock unavailable
Lock lock = new ReentrantLock();

if (lock.tryLock()) { // returns immediately with true or false
    try {
        // critical section
    } finally {
        lock.unlock();
    }
} else {
    // lock not available — do something else instead of waiting forever
    System.out.println("Lock not available — skipping");
}

// wait up to 5 seconds, then give up
if (lock.tryLock(5, TimeUnit.SECONDS)) {
    try {
        // critical section
    } finally {
        lock.unlock();
    }
} else {
    System.out.println("Could not acquire lock in time — moving on");
}
```

The name "Reentrant" means a thread that already holds the lock can acquire it again without deadlocking itself:

```java
// Without reentrancy — deadlock
synchronized void methodA() {
    methodB(); // tries to acquire the same lock again
}

synchronized void methodB() {
    // needs the same lock — but current thread already holds it
    // non-reentrant lock would deadlock here — waiting for itself ❌
}
```

Without reentrancy, a thread calling `methodA()` acquires the lock - then when it calls `methodB()` which needs the same lock, it would wait for itself to release it - which never happens - deadlock.

With `ReentrantLock` - the thread counts its own acquisitions:

```java
// Thread already holds the lock — can acquire it again safely
lock.lock(); // acquisition count = 1
    lock.lock(); // same thread — acquisition count = 2, no deadlock ✅
    lock.unlock(); // count = 1
lock.unlock(); // count = 0 — truly released now
```

The lock keeps a counter - each `lock()` increments it, each `unlock()` decrements it. Only when the count reaches zero is the lock truly released for other threads.

</details>

<details>
<summary>What is ReadWriteLock?</summary>

`ReadWriteLock` is a tool in Java that allows to read from shared data across multiple threads without locking, but locking when writing on that same data to avoid race conditions.

`ReadWriteLock` is a locking mechanism that maintains two separate locks - a read lock and a write lock. Multiple threads can hold the read lock simultaneously since reading doesn't cause race conditions. The write lock is exclusive - when a thread is writing, all other threads (both readers and writters) must wait. This improves performance in scenarios where reads are frequent and writes are rare.

```java
ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
Lock readLock  = rwl.readLock();
Lock writeLock = rwl.writeLock();

// Multiple threads can read simultaneously
void readData() {
    readLock.lock();
    try {
        // ✅ multiple threads here at the same time
        System.out.println(data);
    } finally {
        readLock.unlock();
    }
}

// Only one thread can write — blocks everyone else
void writeData(String newData) {
    writeLock.lock();
    try {
        // ✅ exclusive — no readers or writers allowed
        data = newData;
    } finally {
        writeLock.unlock();
    }
}
```

The read lock isn't about protecting reads from each other - it's about communicating to the `ReadWriteLock` that reading is in progress, so writters know when it's safe to proceed and don't starve waiting for reads to stop.

So the read lock serves two purposes:

| Purpose | Explanation |
| :--- | :--- |
| **Signals active readers** | Writers wait until all readers finish |
| **Prevents writer starvation** | New readers blocked when writer is waiting |

</details>

<details>
<summary>Atomic vs Volatile difference?</summary>

Atomic is a broad concept that means indivisible operation. It appears in multiple contexts in java:

* Atomic classes - `AtomicInteger`, `AtomicReference`, etc.
* Atomic operations - some operations are naturally atomic without needing atomic classes:

```java
// These are atomic by nature in Java — single indivisible operations
int x = 5;        // ✅ atomic — simple assignment
boolean b = true; // ✅ atomic — simple assignment

// These are NOT atomic — multiple steps
count++;           // ❌ read + modify + write
count = count + 1; // ❌ read + modify + write

// Note: long and double assignments are NOT atomic on 32-bit JVMs
// because they're 64-bit values written in two 32-bit operations
long x = 100L; // ⚠️ not guaranteed atomic on 32-bit JVM
```

`volatile` guarantees visibility - reads and writes go directly to main memory so changes are immediately visible to all threads. However it doesn't guarantee atomicity - compound operations like `count++` are still three steps and can cause race conditions. Atomic classes solve both - they guarantee visibility and wrap compound operations into asingle indivisible unit, making them truly thread safe without needing `synchronized`.

Atomic classes use `volatile` internally for their fields, so all reads and writes go directly to main memory guaranteeing visibilty. Plus they add atomicity on top through compare-and-swap (CAS) operations at the hardware level.

</details>

<details>
<summary>Thread pool size criteria:</summary>

Thread pool size depends on the type of tasks beign executed:

CPU intensive tasks: set threads equal to the number of CPU cores. Adding more would cause excessive context switching that wastes CPU time instead of doing real work.

I/O bound tasks: set threads higher than core count since threads spend most of their time waiting for responses, keeping the CPU free to run other threads. A common formula is:

`threads = cores * (1 + wait time / compute time)`

```java
// Get available cores in Java
int cores = Runtime.getRuntime().availableProcessors();

// CPU intensive — threads = cores
ExecutorService cpuPool = Executors.newFixedThreadPool(cores);

// I/O bound — threads = more than cores
// if threads wait 9x longer than they compute: cores * (1 + 9/1) = cores * 10
ExecutorService ioPool = Executors.newFixedThreadPool(cores * 10);
```

</details>