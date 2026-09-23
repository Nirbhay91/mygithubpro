# Q2. What is a Lambda Expression? How does it work internally, and what is its relationship with a Functional Interface?

## 1. Interview-ready answer

A Lambda Expression is a concise way to represent an implementation of a functional interface's single abstract method. It lets us pass behavior as a value and reduces the boilerplate of anonymous inner classes.

For example:

```java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}

Calculator calculator = (a, b) -> a + b;
```

Here `Calculator` is the target functional interface and the lambda provides the implementation of its `add` method.

At the JVM level, Java lambda expressions are generally implemented using the `invokedynamic` bytecode instruction and the LambdaMetafactory mechanism rather than being compiled in the same way as a traditional anonymous inner class. The JVM links the call site to the appropriate lambda implementation at runtime.

## 2. What problem did Lambda solve?

Before Java 8, passing behavior commonly required an anonymous inner class:

```java
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.length() - b.length();
    }
});
```

With Lambda:

```java
Collections.sort(names,
        (a, b) -> a.length() - b.length());
```

The main benefit is less boilerplate and clearer expression of the behavior.

## 3. Lambda is not a standalone method

This is an important interview point.

A lambda needs a target type. In normal Java usage, that target type is a functional interface.

Valid:

```java
Runnable r = () -> System.out.println("Running");
```

Here the compiler knows that the lambda implements:

```java
void run();
```

But this is not valid as a standalone declaration:

```java
// Not valid Java
() -> System.out.println("Hello");
```

## 4. Relationship between Lambda and Functional Interface

Think of it as:

```text
Functional Interface
        |
        | defines one abstract behavior
        v
Single Abstract Method
        |
        | implementation supplied by
        v
Lambda Expression
```

Example:

```java
@FunctionalInterface
interface Greeting {
    void greet(String name);
}

Greeting greeting = name -> System.out.println("Hello " + name);
```

The lambda supplies the implementation of `greet(String name)`.

## 5. Why exactly one abstract method?

The compiler needs an unambiguous target method for the lambda.

For example:

```java
interface Calculator {
    int add(int a, int b);
}
```

The lambda:

```java
(a, b) -> a + b
```

clearly maps to `add`.

If the interface had two unrelated abstract methods, the compiler could not determine which method the lambda was implementing.

A functional interface can still contain:

- one abstract method
- any number of default methods
- any number of static methods
- methods inherited from `Object` do not count as additional abstract methods for functional-interface purposes

## 6. @FunctionalInterface

`@FunctionalInterface` is a compiler-level check/documentation aid.

Example:

```java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}
```

If another abstract method is added, compilation fails because the interface no longer satisfies the functional-interface contract.

The annotation is not what makes an interface functional. The interface is functional because it satisfies the one-abstract-method rule; the annotation asks the compiler to verify that contract.

## 7. Lambda syntax

General form:

```text
(parameters) -> expression
```

or

```text
(parameters) -> { statements }
```

Examples:

```java
x -> x * 2
```

```java
(a, b) -> a + b
```

```java
() -> System.out.println("Hello")
```

```java
(name) -> {
    System.out.println("Hello " + name);
}
```

When there is one parameter, parentheses can usually be omitted:

```java
x -> x * 2
```

## 8. Type inference

The compiler can infer lambda parameter types from the target functional interface.

```java
Function<Integer, Integer> square = x -> x * x;
```

The compiler knows `x` is an `Integer` because `Function<Integer, Integer>` defines:

```java
R apply(T t);
```

The same lambda could be written explicitly as:

```java
Function<Integer, Integer> square = (Integer x) -> x * x;
```

## 9. Lambda and invokedynamic — internal view

A high-level execution path is:

```text
Java source
   |
   | javac
   v
Bytecode containing invokedynamic
   |
   | runtime linkage
   v
LambdaMetafactory
   |
   v
Functional-interface implementation
```

Important interview point:

> A lambda is not simply compiled as a normal anonymous inner-class object in the same way older Java code was.

The compiler emits an `invokedynamic` instruction for the lambda call site. At runtime, the JVM links that call site using the lambda metafactory mechanism to produce an implementation compatible with the target functional interface.

This design gives the JVM/runtime more flexibility than hard-coding a generated anonymous-class implementation into the bytecode.

## 10. Capturing vs non-capturing Lambda

### Non-capturing lambda

A lambda that does not use variables from the surrounding scope:

```java
Runnable task = () -> System.out.println("Hello");
```

It does not capture an external local variable.

### Capturing lambda

A lambda that uses a variable from its surrounding scope:

```java
String prefix = "Hello";

Function<String, String> greeting =
        name -> prefix + " " + name;
```

Here the lambda captures `prefix`.

A captured local variable must be final or effectively final:

```java
String prefix = "Hello";

Function<String, String> greeting =
        name -> prefix + " " + name;
```

This is valid.

But:

```java
String prefix = "Hello";
prefix = "Hi";

// Lambda cannot capture prefix here
```

because `prefix` is no longer effectively final.

## 11. Why must captured local variables be effectively final?

Local variables live in the method's stack frame. A lambda may outlive the execution of that method, so Java does not capture a mutable stack variable by reference in the same way some languages do.

Instead, the lambda captures the value available to it, and Java requires that local variable to be final or effectively final to avoid mutable-capture ambiguity.

## 12. Lambda vs Anonymous Inner Class

| Lambda | Anonymous Inner Class |
|---|---|
| Concise syntax | More boilerplate |
| Targets a functional interface | Can implement/extend applicable types with broader anonymous-class semantics |
| `this` refers to the enclosing instance | `this` refers to the anonymous-class instance |
| Does not introduce a new named class declaration in source | Creates an anonymous class construct |
| Commonly implemented using `invokedynamic` | Traditionally compiled using an anonymous-class implementation |

Important `this` example:

```java
class Demo {
    void test() {
        Runnable lambda = () ->
                System.out.println(this.getClass().getSimpleName());

        Runnable anonymous = new Runnable() {
            @Override
            public void run() {
                System.out.println(this.getClass().getSimpleName());
            }
        };
    }
}
```

In the lambda, `this` refers to the enclosing `Demo` instance. In the anonymous class, `this` refers to the anonymous `Runnable` object.

## 13. Real project-style example

Suppose an application needs to filter active customers:

```java
List<Customer> activeCustomers = customers.stream()
        .filter(customer -> customer.isActive())
        .collect(Collectors.toList());
```

The lambda:

```java
customer -> customer.isActive()
```

represents the filtering behavior expected by `Predicate<Customer>`.

Conceptually:

```text
Stream.filter()
      |
      v
Predicate<Customer>
      |
      v
customer -> customer.isActive()
```

## 14. Common interviewer traps

### Trap 1: "Lambda is a functional interface."

Incorrect.

Lambda is an expression that can provide the implementation for a functional interface's abstract method.

### Trap 2: "@FunctionalInterface creates a functional interface."

Not exactly.

The interface is functional because it has one abstract method. The annotation makes the compiler verify that contract.

### Trap 3: "Lambda always creates a new object every time."

Do not make that blanket claim. Lambda implementation details are runtime-dependent. In particular, non-capturing lambdas may be reused/cached, while capturing lambdas need state associated with captured values.

### Trap 4: "Lambda is always faster than an anonymous class."

Do not claim this without context. Lambda mainly improves expressiveness and can give the runtime implementation flexibility; actual performance depends on the workload and JVM/runtime behavior.

## 15. 2-minute interview answer

> A Lambda Expression is a concise way to represent behavior and is primarily used as an implementation of a functional interface's single abstract method. For example, `Predicate<Integer> p = x -> x > 10;` uses a lambda to implement `Predicate.test()`.
>
> Lambda expressions were introduced in Java 8 to reduce boilerplate and enable functional-style programming, especially with the Stream API.
>
> The compiler uses the target functional-interface type for type inference and emits an `invokedynamic` instruction for lambda creation. At runtime, the JVM links the call site using the lambda metafactory mechanism. This differs from the traditional anonymous-inner-class approach.
>
> Lambdas can be capturing or non-capturing. A capturing lambda can use local variables only when those variables are final or effectively final. Also, `this` inside a lambda refers to the enclosing instance, unlike `this` inside an anonymous inner class.

## 16. Follow-up questions to prepare next

1. What is a Functional Interface?
2. What is the difference between Predicate, Function, Consumer and Supplier?
3. How does `invokedynamic` work with Lambda?
4. What is LambdaMetafactory?
5. Capturing vs non-capturing lambda?
6. Why must captured local variables be effectively final?
7. Lambda vs anonymous inner class?
8. What does `this` mean inside a lambda?
9. Can a lambda throw checked exceptions?
10. Can we overload methods using different functional interfaces?
