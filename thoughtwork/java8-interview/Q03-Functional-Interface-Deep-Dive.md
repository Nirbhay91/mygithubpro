# Q3 — What is a Functional Interface? (5–9 Years Interview Depth)

## 1. 2-minute interview answer

A **functional interface** is an interface that has **exactly one abstract method**. It is the target type for a lambda expression or method reference in Java 8+.

It can still contain multiple `default` and `static` methods, and methods inherited from `Object` do not count as abstract methods for functional-interface determination. Java provides built-in functional interfaces such as `Predicate`, `Function`, `Consumer`, `Supplier`, `UnaryOperator`, `BinaryOperator`, and `Runnable`.

`@FunctionalInterface` is an optional compiler-level annotation that documents the intent and makes the compiler reject the interface if its functional-interface contract is broken.

Example:

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);

    default void log() {
        System.out.println("Calculation");
    }
}

Calculator add = (a, b) -> a + b;
System.out.println(add.calculate(10, 20));
```

The important idea is: **one abstract method gives Java an unambiguous target type for a lambda.**

---

## 2. Why was Functional Interface introduced?

Before Java 8, behavior was commonly passed using anonymous classes:

```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running");
    }
};
```

Java 8 made behavior easier to pass as data through lambdas:

```java
Runnable r = () -> System.out.println("Running");
```

For the lambda to be valid, the compiler needs a **target type** that tells it what method signature the lambda represents. A functional interface supplies that target type.

So:

```text
Lambda expression
      ↓
Target type
      ↓
Functional interface
      ↓
Single abstract method
```

---

## 3. What exactly counts as a Functional Interface?

The key rule is not literally "the interface must contain one method in its source code." The rule is based on having **one abstract method** after considering inheritance and methods that do not contribute to the abstract-method count.

Example:

```java
@FunctionalInterface
interface MyTask {
    void execute();

    default void log() {
        System.out.println("log");
    }

    static void info() {
        System.out.println("info");
    }
}
```

This is valid because only `execute()` is abstract.

### Important interview point

`default` and `static` methods do **not** become abstract methods.

---

## 4. Can a Functional Interface have default methods?

Yes.

```java
@FunctionalInterface
interface PaymentProcessor {
    void process();

    default void audit() {
        System.out.println("Audit");
    }
}
```

It remains functional because `process()` is the only abstract method.

This design allows an interface to evolve with additional behavior without requiring every implementation to provide implementations for those default methods.

---

## 5. Can a Functional Interface have static methods?

Yes.

```java
@FunctionalInterface
interface Validator {
    boolean validate(String input);

    static boolean isNull(String input) {
        return input == null;
    }
}
```

`static` interface methods are not abstract methods, so they do not break the functional-interface contract.

---

## 6. Can it have methods from Object?

Methods corresponding to public methods of `Object` do not count as the abstract-method requirement for a functional interface.

For example:

```java
@FunctionalInterface
interface Processor {
    void process();
    boolean equals(Object obj);
}
```

The `equals(Object)` declaration does not create a second distinct abstract method for functional-interface purposes because it corresponds to a public method of `Object`.

### Interview trap

Do not simply say: "Every method declaration except default/static counts." The functional-interface rules also account for methods inherited from `Object` and inherited abstract methods that may have the same signature.

---

## 7. What does @FunctionalInterface do?

`@FunctionalInterface` is a compiler-checking annotation.

```java
@FunctionalInterface
interface Printer {
    void print();
}
```

If somebody later adds another incompatible abstract method:

```java
@FunctionalInterface
interface Printer {
    void print();
    void scan();
}
```

The compiler reports an error because the interface no longer satisfies the functional-interface contract.

### Important

The annotation is **not what makes the interface functional**. An interface can satisfy the functional-interface rules without explicitly using `@FunctionalInterface`.

The annotation mainly provides intent/documentation plus compiler validation.

---

## 8. Is @FunctionalInterface mandatory?

No.

This is valid:

```java
interface Greeting {
    void sayHello();
}

Greeting g = () -> System.out.println("Hello");
```

The compiler can recognize the interface as a functional interface from its structure.

But in production code, using `@FunctionalInterface` is useful because it protects the contract during future changes.

---

## 9. Functional Interface vs Normal Interface

| Functional Interface | Normal Interface |
|---|---|
| Exactly one abstract method | Can have multiple abstract methods |
| Can be target type of a lambda | Generally cannot be directly targeted by a lambda if multiple abstract methods exist |
| Can have default/static methods | Can also have default/static methods |
| `@FunctionalInterface` can enforce the contract | Annotation is not applicable if contract is violated |

Example:

```java
interface NormalInterface {
    void a();
    void b();
}
```

This cannot be used as:

```java
// NormalInterface x = () -> ...; // invalid
```

because Java cannot determine whether the lambda should implement `a()` or `b()`.

---

## 10. Why does Lambda need a Functional Interface?

A lambda by itself does not represent a standalone named object type.

For example:

```java
x -> x * 2
```

The compiler needs contextual information such as:

```java
Function<Integer, Integer> f = x -> x * 2;
```

Here `Function<Integer, Integer>` tells Java:

```text
Input  -> Integer
Output -> Integer
Method -> apply(Integer)
```

Therefore the lambda can be type-checked against the functional interface's abstract method.

---

## 11. Built-in Functional Interfaces — Must Know

### Predicate<T>

Takes one argument and returns boolean.

```java
Predicate<Integer> p = n -> n > 10;
```

Abstract method:

```java
boolean test(T t)
```

### Function<T, R>

Takes one argument and returns a result.

```java
Function<String, Integer> f = String::length;
```

Abstract method:

```java
R apply(T t)
```

### Consumer<T>

Takes an argument and returns nothing.

```java
Consumer<String> c = System.out::println;
```

Abstract method:

```java
void accept(T t)
```

### Supplier<T>

Takes no argument and supplies a result.

```java
Supplier<Double> s = Math::random;
```

Abstract method:

```java
T get()
```

### UnaryOperator<T>

Takes and returns the same type.

```java
UnaryOperator<Integer> square = n -> n * n;
```

### BinaryOperator<T>

Takes two values of the same type and returns that same type.

```java
BinaryOperator<Integer> sum = Integer::sum;
```

### Runnable

Takes no argument and returns nothing.

```java
Runnable task = () -> System.out.println("Task");
```

---

## 12. Function vs Predicate vs Consumer vs Supplier

A very common interview question:

```text
Predicate<T>       T -> boolean
Function<T,R>      T -> R
Consumer<T>        T -> void
Supplier<T>        () -> T
```

Easy memory trick:

```text
Predicate  = Ask a question
Function   = Transform
Consumer   = Consume/do something
Supplier   = Provide something
```

---

## 13. Can two abstract methods make a Functional Interface if they have same signature?

If inherited declarations represent the same abstract method signature and are compatible, they can contribute to a single functionally relevant method rather than necessarily creating two distinct abstract methods.

For interview purposes, the safe explanation is:

> A functional interface must have exactly one functionally relevant abstract method after Java's inheritance and method-signature rules are applied.

Do not reduce the rule to simply counting source-code lines containing abstract methods.

---

## 14. Functional Interface with Generics

Functional interfaces can be generic:

```java
@FunctionalInterface
interface Converter<T, R> {
    R convert(T value);
}

Converter<String, Integer> converter = Integer::valueOf;
```

Here the same functional interface can represent different conversions depending on its type parameters.

---

## 15. Functional Interface and Method Reference

Method references also require a compatible target functional interface.

```java
Function<String, Integer> f = Integer::valueOf;
```

Conceptually:

```text
Method reference
      ↓
Target functional interface
      ↓
Abstract method signature
      ↓
Compatibility check
```

This is why method references and lambdas are closely related to functional interfaces.

---

## 16. Real project example — Payment processing

Suppose different payment validations are required:

```java
@FunctionalInterface
interface PaymentRule {
    boolean validate(Payment payment);
}
```

Different rules can be supplied as behavior:

```java
PaymentRule amountRule = p -> p.getAmount() > 0;
PaymentRule currencyRule = p -> p.getCurrency() != null;
```

A service can consume the rule without knowing its implementation:

```java
boolean valid = amountRule.validate(payment);
```

This is useful when behavior needs to be passed, composed, or configured without creating many small implementation classes.

---

## 17. Functional Interface vs Abstract Class

A functional interface is not a replacement for an abstract class.

### Functional Interface

- Defines one abstract behavior contract.
- Works naturally with lambdas and method references.
- Does not hold instance state like an ordinary class.
- Can have default/static interface methods.

### Abstract Class

- Can contain instance fields/state.
- Can have constructors.
- Can have multiple abstract methods.
- Can have concrete instance methods.
- A class can extend only one class.

Use a functional interface when the primary requirement is passing a single piece of behavior.

---

## 18. What happens if @FunctionalInterface is used incorrectly?

Example:

```java
@FunctionalInterface
interface Invalid {
    void first();
    void second();
}
```

Compilation fails because the interface has more than one functionally relevant abstract method.

This is useful in a large codebase because someone cannot accidentally change a lambda-based API into a non-functional interface without the compiler highlighting the problem.

---

## 19. Interview traps

### Trap 1: "Functional interface can contain only one method."

Incorrect.

It can contain one abstract method plus multiple default/static methods.

### Trap 2: "@FunctionalInterface makes it functional."

Incorrect.

The interface must satisfy the functional-interface rules. The annotation asks the compiler to verify the intent.

### Trap 3: "Lambda can implement any interface."

Incorrect.

A lambda needs a compatible functional interface target type.

### Trap 4: "Runnable was introduced in Java 8."

Incorrect.

`Runnable` predates Java 8; Java 8 made lambda-based usage concise.

### Trap 5: "One method written in the interface always means functional."

Usually, but interview-level answers should mention inherited methods and the special treatment of `Object` methods.

---

## 20. Deep connection: Functional Interface + Stream API

Functional interfaces are heavily used by the Stream API.

Examples:

```java
list.stream()
    .filter(x -> x > 10)
    .map(x -> x * 2)
    .forEach(System.out::println);
```

Conceptually:

```text
filter()  -> Predicate
map()     -> Function
forEach() -> Consumer
```

This is one of the most important Java 8 interview connections.

---

## 21. Deep connection: Functional Interface + CompletableFuture

Java concurrency APIs also accept functional interfaces.

Example:

```java
CompletableFuture
    .supplyAsync(() -> fetchData())
    .thenApply(data -> transform(data))
    .thenAccept(result -> save(result));
```

Conceptually:

```text
supplyAsync -> Supplier
thenApply  -> Function
thenAccept -> Consumer
```

This demonstrates that functional interfaces are not only a Stream API feature; they are a general mechanism for passing behavior.

---

## 22. Senior-level answer: Why use @FunctionalInterface in production?

A good 5–9 year answer:

> I use `@FunctionalInterface` when an interface is intentionally designed as a behavior contract for lambdas or method references. It communicates the API design intent and lets the compiler protect that contract if another abstract method is added later. This is particularly useful for reusable utility APIs, callbacks, strategies, predicates, and stream/concurrency-related behavior.

---

## 23. Common interviewer follow-ups

### Q: Can a functional interface extend another interface?

Yes, provided the resulting interface still has exactly one functionally relevant abstract method.

### Q: Can a functional interface extend two interfaces?

Potentially yes, if the inherited abstract methods are compatible and the resulting interface still satisfies the single-abstract-method rule. If they introduce distinct abstract methods, it is not functional.

### Q: Can an interface have private methods and still be functional?

Yes. Private interface methods do not add abstract methods and therefore do not violate the functional-interface contract.

### Q: Can a functional interface have default methods?

Yes.

### Q: Can a functional interface have static methods?

Yes.

### Q: Can a functional interface have zero abstract methods?

No. It needs one functionally relevant abstract method.

---

## 24. One-line revision

> **Functional Interface = exactly one functionally relevant abstract method + lambda/method-reference target type + optional @FunctionalInterface compiler check.**

---

## 25. Interview-ready final answer

> A functional interface in Java is an interface with exactly one functionally relevant abstract method. It is used as the target type for lambda expressions and method references. It can still contain multiple default, static, and private methods. `@FunctionalInterface` is optional, but I prefer using it because the compiler validates that the interface remains functional if the code evolves. Java provides standard functional interfaces such as Predicate, Function, Consumer, Supplier, UnaryOperator and BinaryOperator, and these are heavily used in Streams, CompletableFuture and callback/strategy-style designs.
