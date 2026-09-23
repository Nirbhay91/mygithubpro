# Q1. What are the major features introduced in Java 8?

## Interview-ready answer

Java 8 introduced several major features such as Lambda Expressions, Functional Interfaces, Stream API, Optional, Method References, Default and Static methods in interfaces, and the new Date and Time API.

The biggest shift in Java 8 was support for functional-style programming. Lambda Expressions allow behavior to be passed as a value, Functional Interfaces provide the target type for lambdas, and the Stream API provides a declarative way to process data.

Optional provides an explicit representation for a value that may be absent. Java 8 also added default and static interface methods and a modern Date/Time API.

## Major features

1. Lambda Expressions
2. Functional Interfaces
3. Stream API
4. Optional
5. Method References
6. Default and Static methods in interfaces
7. New Date and Time API

## Quick examples

### Lambda

```java
names.forEach(name -> System.out.println(name));
```

### Functional Interface

```java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}

Calculator calculator = (a, b) -> a + b;
```

### Stream API

```java
List<Integer> result = numbers.stream()
        .filter(n -> n % 2 == 0)
        .collect(Collectors.toList());
```

### Optional

```java
Optional<String> name = getName();
name.ifPresent(System.out::println);
```

### Method Reference

```java
names.forEach(System.out::println);
```

### Default method

```java
interface Vehicle {
    default void start() {
        System.out.println("Starting...");
    }
}
```

### Date/Time API

```java
LocalDate today = LocalDate.now();
```

## Important interview distinction

Collection stores/manages data; Stream processes data. A Stream is not a Collection.

## Common follow-ups

- What is a Lambda Expression?
- What is a Functional Interface?
- Why can a Functional Interface have only one abstract method?
- Stream vs Collection?
- Intermediate vs Terminal operations?
- map() vs flatMap()?
- Why are Streams lazy?
- orElse() vs orElseGet()?
- Why were default methods introduced?
