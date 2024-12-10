## Method References
- An alternative syntax for lambda expressions
- **Method references are a special type of lambda expressions.** 
- They're often used to create simple lambda expressions by referencing existing methods.



**example 1**
```java 
Function<Person, Integer> f = person -> person.getAge() ;
```

can be...

```java
Function<Person, Integer> f = Person::getAge ;
```
**example 2**

- Example using BinaryOperator (extends BiFunction but 2 inputs and 1 return ar all same types)

example of BiFunction and BinaryOperator for background
```java
// BiFunction
BiFunction<Integer, Integer, Integer> func = (x1, x2) -> x1 + x2;

Integer result = func.apply(2, 3);

System.out.println(result); // 5

// BinaryOperator
BinaryOperator<Integer> func2 = (x1, x2) -> x1 + x2;

Integer result2 = func.apply(2, 3);

System.out.println(result2); // 5
```

actual example...

```java
BinaryOperator<Integer> sum = (i1, i2) -> i1 + i2 ;
							= (i1, i2) -> Integer.sum(i1, i2) ;
```

i.e. can leverage sum method on Integer to give...
**NB: can have multiple params ones inputs match**

```java
BinaryOperator<Integer> sum = Integer::sum ;
```

could do a max...

```java
BinaryOperator<Integer> max = Integer::max ;
```

**example 3**

```
// Consumer is a "functional interface" that accepts 1 parameter and returns nothing
Consumer<String> printer = s -> System.out.println(s) ;
Consumer<String> printer = System.out::println ;
```

**example 4**

The pre-java8 Comparator just had a compare and equals method to implement. In Java 8 it has been extended to include many functional programming style methods. Java 8 provides new ways of defining Comparators by using lambda expressions and the comparing() static factory method.

```java
Comparator<Person> cmp = Comparator.comparing(Person::getLastName)		// static interface method
										.thenComparing(Person::getFirstName)
										.thenComparing(Person::getAge);
```

In theory any legacy interface (that can only be called using anonymous or concrete classes) can be updated in this way (adding functional programming static & default methods) to build a new api