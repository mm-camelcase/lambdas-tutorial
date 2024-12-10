## Function Interface
- Note: see [`java.util.function`](https://download.java.net/java/early_access/panama/docs/api/java.base/java/util/function/Function.html) package below for more...
- The Function Interface is a part of the [`java.util.function`](https://download.java.net/java/early_access/panama/docs/api/java.base/java/util/function/Function.html) package which has been introduced since Java 8, to implement functional programming in Java
- It represents a function which takes in one argument and produces a result.

- Take this lambda expression that argument x and increments it by one...

```java
x -> x + 1
```

What are the types in this function?....It depends!

- In Java the same lambda expression could be bound to variables of different types

```
Function<Integer,Integer> add1 = x -> x + 1;
Function<String,String> concat = x -> x + 1;
```

- To invoke these functions use Function's apply method

```java
Integer two = add1.apply(1); //yields 2
String answer = concat1.apply("0 + 1 = "); //yields "0 + 1 = 1"
```

- Can also create functions that use methods we have already defined

```java
public class Utils {
   public static Integer add1(Integer x) { return x + 1; }
   public static String concat1(String x) { return x + 1; }
}
```

```java
Function<Integer,Integer> addone = x -> Utils.add1(x);
Function<Integer,Integer> concatone = x -> Utils.concat1(x);

```

Or using this cleaner Method Reference (see below) syntax

```java
Function<Integer,Integer> addone = Utils::add1;
Function<String,String> concatone = Utils::concat1;
```