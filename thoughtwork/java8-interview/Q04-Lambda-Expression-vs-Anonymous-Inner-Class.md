# Q4 — Lambda Expression vs Anonymous Inner Class

## Interview Question
**What is the difference between Lambda Expression and Anonymous Inner Class in Java?**

## 2-Minute Interview Answer
Lambda expression was introduced in Java 8 to provide a concise way of implementing a functional interface. An anonymous inner class is an unnamed class used to create an object and provide implementations for methods.

A lambda can target only a functional interface, while an anonymous class can extend a class or implement an interface and can contain its own fields and initialization logic. Another important difference is `this`: inside an anonymous class, `this` refers to the anonymous class instance; inside a lambda, `this` refers to the enclosing instance.

Lambdas are commonly used with Streams, Collections, and functional programming APIs because they reduce boilerplate code.

## 1. Basic Example

### Anonymous Inner Class
```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running");
    }
};
```

### Lambda
```java
Runnable task = () -> {
    System.out.println("Running");
};
```

A lambda is concise because `Runnable` is a functional interface.

## 2. Functional Interface Requirement
Lambda expressions require a functional-interface target type.

```java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}

Calculator calculator = (a, b) -> a + b;
```

An anonymous class can also implement the interface, but it is not restricted to the lambda model.

## 3. `this` — Important Interview Point

```java
class Employee {
    void test() {
        Runnable r1 = new Runnable() {
            @Override
            public void run() {
                System.out.println(this);
            }
        };

        Runnable r2 = () -> {
            System.out.println(this);
        };
    }
}
```

- In the anonymous class, `this` refers to the anonymous-class instance.
- In the lambda, `this` refers to the enclosing `Employee` instance.

**Interview line:** Lambda expressions do not create a new `this` context.

## 4. Variable Capture
Both lambdas and anonymous classes can access local variables when they are final or effectively final.

```java
int x = 10;
Runnable r = () -> System.out.println(x);
```

If `x` is reassigned, it is no longer effectively final and cannot be captured as a local variable.

## 5. Anonymous Class Can Have Its Own State

```java
Runnable task = new Runnable() {
    private int count = 0;

    @Override
    public void run() {
        count++;
        System.out.println(count);
    }
};
```

An anonymous class can declare its own instance fields. A lambda does not declare its own instance fields in this way.

## 6. Anonymous Class Can Extend a Class

```java
Thread thread = new Thread() {
    @Override
    public void run() {
        System.out.println("Running");
    }
};
```

A lambda cannot extend a class; it targets a functional interface.

## 7. Multiple Abstract Methods
An anonymous class can implement an interface with multiple abstract methods. A lambda cannot, because its target must be a functional interface with exactly one abstract method.

## 8. Stream API Example

```java
List<Integer> numbers = Arrays.asList(10, 20, 30, 40);

numbers.stream()
       .filter(n -> n > 20)
       .forEach(n -> System.out.println(n));
```

Here the lambdas target functional interfaces such as `Predicate<Integer>` and `Consumer<Integer>`.

## 9. JVM-Level Interview Point
Do not simply say that a lambda is syntactic sugar for an anonymous class. Modern Java implementations commonly use the `invokedynamic` mechanism and `LambdaMetafactory` for lambda creation, while anonymous classes have generated class-file semantics.

## 10. Comparison

| Feature | Lambda | Anonymous Class |
|---|---|---|
| Introduced | Java 8 | Earlier Java |
| Syntax | Concise | Verbose |
| Target | Functional interface | Class or interface |
| Multiple abstract methods | No | Yes |
| Extend class | No | Yes |
| Own instance fields | No | Yes |
| Own constructor | No | Yes |
| `this` | Enclosing instance | Anonymous instance |
| Streams | Very common | Less common |
| JVM implementation | Commonly `invokedynamic` | Generated class |

## Senior-Level Follow-ups
1. Is lambda an anonymous inner class?
2. How does JVM implement lambda expressions?
3. What is `invokedynamic`?
4. What is `LambdaMetafactory`?
5. Why does `this` behave differently in lambda and anonymous class?
6. Can lambda access local variables?
7. What is effectively final?
8. Can lambda implement an interface with two abstract methods?
9. Lambda vs method reference?
10. Does lambda always improve performance?

## Memory Trick
**Lambda = functional interface + concise behavior + enclosing `this`**

**Anonymous class = unnamed class/object + its own `this` + more flexibility**
