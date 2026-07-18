<details>
<summary>What is a framework?</summary>



</details>

<details>
<summary>Why to write against interfaces?</summary>

Writing against interfaces hides the implementation detail behind a contract, making your code flexible, testable and easy to maintain - changing the underlying data structure becomes a one line decision instead a system-wide refactor.

</details>

<details>
<summary>What is the difference between a side effect and a result?</summary>

Result - the operation returns a value you can use.
Side effect - the operation does something but returns nothing useful.

```java
// Terminal operations that return a RESULT
long count    = stream.count();                    // returns a number
List<String> list = stream.collect(toList());      // returns a collection
Optional<String> first = stream.findFirst();       // returns an element
boolean match = stream.anyMatch(s -> s.equals("Carlos")); // returns boolean

// Terminal operations that produce a SIDE EFFECT
stream.forEach(System.out::println); // returns void — just prints
```

You already know pure functions have no side effects — `forEach` is the classic impure stream operation because it does something (prints, saves, sends) without returning anything. `collect`, `count`, `findFirst` are closer to pure — they return a value without modifying external state.

</details>

<details>
<summary>What does the flatMap() method?</summary>



</details>

<details>
<summary>What does the map() method?</summary>



</details>