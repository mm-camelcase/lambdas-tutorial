## Optionals (Building Errorless Processing Pipelines with Optionals)

### Basic usage

- A first pattern
	- but verbose
	- creates nesting in streams (see example at end)

```java
Optional<Person> opt = ...;
if (opt.isPresent()) {
	Person p = opt.get();
} else {
	// there is nobody here...
}
```

- There is a better way...

```java
Optional<Person> opt = ...;
Person p1 = opt.orElse(Person.getDefault());			// creates the default and return if required
Person p2 = opt.orElseGet(() -> Person.getDefault());	// only create and return default if required (does not create instance unnecessarily)Java 8 what is an interface

```

**Patterns to Build an Optional**

- First, the default constructor of the Optional class is private
- So we cannot build an optional using new
- We have static methods:

```java
Optional<String> empty = Optional.empty(); 
Optional<String> nonEmpty = Optional.of(s); 			// if s is null then ....NullPointerException thrown
Optional<String> couldBeEmpty = Optional.ofNullable(s);	// if s is null then .... empty Optional returned
```

### Advanced usage

- Second Type of Patterns
	- sees an optional as a special stream 
	- That can hold only one or zero element
	

It has extra methods that allow it to integrate well with streams (they are similar methods to streams api)

```java
public Optional<U> map(Function<T, U> mapper); 		// Returns an empty optional if `this` is empty
public Optional<T> filter(Predicate<T> filter);		// Returns an empty optional if `this` is empty
public void ifPresent(Consumer<T> consumer);		// Does nothing if `this` is empty
```

Also has a `flatMap()` method (like `Stream`)

```java
Optional<U> flatMap(Function<T, Optional<U>> flatMapper);
```

- It takes the content of the optional
- If there is one it maps it
- And then returns a wrapping optional that can be empty


**Example using Streams ans Optionals ...No nulls, no exceptions, parallel computations**

- Take a stream of doubles, calculate the inverse, then the square root

- Use these `NewMath` util methods

```java
public class NewMath {

    public static Optional<Double> inv(Double d) {
        return d == 0d ? Optional.empty() :
                         Optional.of(1d/d);
    }
    
    public static Optional<Double> sqrt(Double d) {
        return d < 0d ? Optional.empty() :
                        Optional.of(Math.sqrt(d));
    }
}
```

- First attempt

```java
List<Double> result = new ArrayList<>();

ThreadLocalRandom.current()
	   .doubles(10_000).boxed()
	   .forEach(
			   d -> NewMath.inv(d)
					   .ifPresent(
							   inv -> NewMath.sqrt(inv)
										   .ifPresent(
												   sqrt -> result.add(sqrt)
										   )
					   )
	   );
```

- Works but, looks shit because of `forEach` and `ifPresent` nested checks
- Also, cannot use parallel, e.g.

```java
List<Double> result = new ArrayList<>();

ThreadLocalRandom.current()
	   .doubles(10_000).boxed().parallel()
	   .forEach(
			   ...
```

Because we would have e.g. 8 threads (num cores in cpu) trying to write to ArrayList declared outside - race condition - indexoutofbounds exception


- Correct methods uses `flatMap` on Stream api and `Optional` class
- Create a Function that returns a `Stream<Double>`

```java
Function<Double, Stream<Double>> invSqrt = 
d -> NewMath.inv(d)                        	// Optional<Double>
	.flatMap(d -> NewMath.sqrt(d)) 			// Optional<Double>
	.map(d -> Stream.of(d)) 				// Optional<Stream<Double>>
	.orElseGet(() -> Stream.empty()) ; 		// Stream<Double>
```

- Or better again with method references

```java
Function<Double, Stream<Double>> invSqrt = 
d -> NewMath.inv(d)                 		// Optional<Double>
	.flatMap(NewMath::sqrt) 				// Optional<Double>
	.map(Stream::of) 						// Optional<Stream<Double>>
	.orElseGet(Stream::empty) ; 			// Stream<Double>
```

- Then call it using `flatMap`

```java
List<Double> doubles = ...;
List<Double> invSqrtOfDoubles = 
	doubles.stream()
	.flatMap(invSqrt)
	.collect(Collectors.toList()); // collects the elements in a list
```

- And we can now go safely parallel

```java
List<Double> doubles = ...;
List<Double> invSqrtOfDoubles = 
	doubles.stream().parallel()
	.flatMap(invSqrt)
	.collect(Collectors.toList()); // collects the elements in a list
```

