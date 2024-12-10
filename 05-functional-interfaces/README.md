## Functional Interfaces

- A lambda expression is an instance of a functional interface (not to be confused with [Function interface previously](../03-function-interface/README.md))
- **Definition**
	- With only one abstract method
	- Default methods do not count (can have many)
	- Static methods do not count (can have many)
	- Methods from the Object class do not count (e.g. equals)
- A functional interface may be annotated with @FunctionalInterface
	- It is not mandatory, for legacy reasons
	- The compiler will tell us if an annotated interface is functional or not
	
	
## The java.util.function package

- A new package from Java 8, with the most useful functional interfaces
- There are 43 of them!
- Four categories: 
1) The Consumers
2) The Supplier
3) The Functions
4) The Predicates
	
	
## 1) The Consumers

A consumer consumes an object, and does not return anything

```java
public interface Consumer<T> {
	public void accept(T t);
}
```

```java
Consumer<String> printer = s -> System.out.println(s);
							= System.out::println;
```

BiConsumer takes 2 objects and does not return anything

```java
public interface BiConsumer<T, V> {
	public void accept(T t, V v);
}
```

## 2) The Supplier

A supplier provides an object, takes no parameter

```java
public interface Supplier<T> {
	public T get();
}
```

```java
Supplier<Person> personSupplier = () -> new Person();
								= Person::new;
```

## 3) The Functions

A function takes an object an returns another object

```java
public interface Function<T, R> {
	public R apply(T t);
}
```

```java
Function<Person, Integer> ageMapper = person -> person.getAge();
									= Person::getAge;
```

See BiFunction and BiOperator above

## 4) The Predicates

A predicate takes an object an return a boolean

```java
public interface Predicate<T> {
	public boolean test(T t);
}
```

```java
Predicate<Person> ageGT20 = person -> person.getAge() > 20;	//could be used to filter out people less that 20 years old from a list
```



## Function Interfaces for Primitive Types

Other functional interfaces have been defined to handle primitive types, for instance: 
- IntPredicate
- IntFunction
- IntToDoubleFunction
- Etc…


## Predicate JDK Example 

This shows how predicate was implemented in jdk using Lambda experssions, Default methods, and Static methods

```java
package com.mm.lambda.predicate;

@FunctionalInterface
public interface PredicateMM<T> {
	
	public boolean test(T t);
    
    public default PredicateMM<T> and(PredicateMM<T> other) {
        return t -> test(t) && other.test(t) ;
    }
    
    public default PredicateMM<T> or(PredicateMM<T> other) {
        return t -> test(t) || other.test(t);
    }
    
    public static <U> PredicateMM<U> isEqualsTo(U u) {
        return s -> s.equals(u);
    }
	
	public static <U> PredicateMM<U> print() {
    	return g -> {
    		System.out.println(g); // print param passed to abstract method
    		return false;
    	};
    }

}
```

```java
PredicateMM<String> p1 = s -> s.length() < 20 ;
PredicateMM<String> p2 = s -> s.length() > 5 ;

boolean b = p1.test("Hello");
System.out.println("Hello is shorter than 20 chars : " + b);

PredicateMM<String> p3 = p1.and(p2) ;

System.out.println("P3 for Yes: " + p3.test("Yes"));
System.out.println("P3 for Good morning: " + p3.test("Good morning"));
System.out.println("P3 for Good morning gentlemen: " + p3.test("Good morning gentlemen"));

PredicateMM<String> p4 = p1.or(p2) ;

System.out.println("P4 for Yes: " + p4.test("Yes"));
System.out.println("P4 for Good morning: " + p4.test("Good morning"));
System.out.println("P4 for Good morning gentlemen: " + p4.test("Good morning gentlemen"));

PredicateMM<String> p5 = PredicateMM.isEqualsTo("Yes");

System.out.println("P5 for Yes: " + p5.test("Yes"));
System.out.println("P5 for No: " + p5.test("No"));

PredicateMM<Integer> p6 = PredicateMM.isEqualsTo(1);

PredicateMM<String> p7 = PredicateMM.print();
System.out.println("P7 param: " + p7.test("99"));

// prints 
// 99
// P7 param: false

```

Note how...
- `and` and `or` methods were implemented using lambda expressions based on the 1 abstract method
- `isEqualsTo` compared the parameter passed to the Predicate to the fixed value (e.g. yes or 1)