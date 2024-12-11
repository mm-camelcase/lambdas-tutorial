## Streams

**What is a stream?**

- In computer science, a stream is a sequence of data elements made available over time. A stream can be thought of as items on a conveyor belt being processed one at a time rather than in large batches.

- From a technical point of view: a typed interface

**Stream Definition**

- A Stream does not hold any data
	- It pulls the data it processes from a source
- A Stream does not modify the data it processes
	- Because we want to process the data in parallel with no visibility issues
- The source may be unbounded
	- Which can mean it is not finite
	- But most of the time, it only means that the size of this source is not known at build time
	

**How to Build Streams**

Some examples...

```java
Person p1 = new Person("Alice", 23);
Person p2 = new Person("Brian", 56);

List<Person> people = new ArrayList(Arrays.asList(p1, p2));
Stream<Person> stream = people.stream();

stream.forEach(System.out::println);

// prints
// Person{name=Alice, age=23}
// Person{name=Brian, age=56}

Stream<String> stream2 = Stream.of("one");
stream2.forEach(System.out::println);

// prints
// one

Stream.of("one", "two", "three")
	.forEach(System.out::println);   	
// prints
// one
// two
// three
```

Other examples

```java
// a constant Stream (keeps growing unless limited)
Stream.generate(() -> "one");

// a growing Stream
Stream.iterate("+", s -> s + "+");

// a random Stream
ThreadLocalRandom.current().ints();
```

Character and file processing streams

```java
// a Stream on the letters of a String
IntStream stream = "hello".chars();

// a Stream on a regular expression
Stream<String> words = Pattern.compile("[^\\p{javaLetter}]")
	.splitAsStream(book);

// a Stream on the lines of a text file
Stream<String> lines = Files.lines(path);
```

StreamBuilder pattern

```java
// first build a Stream.Builder
Stream.Builder<String> builder = Stream.builder();

// by chaining the add() method
builder.add("one").add("two").add("three");
// or by calling accept()
builder.accept("four");

// call the build() method
Stream<String> stream = builder.build();

// use the stream
stream.forEach(System.out::println);
```

**Note:** A built stream will throw an exception on an `add()` or `accept()` call if `build()` is already called

**Map, Filter, Reduce on a stream**

- Print out the ages of the people older than 20

```java
// a first way of writing it
persons.stream()                    // Stream<Person>
	.map(p -> p.getAge())          	// Stream<Integer>
	.filter(age -> age > 20) 		// Stream<Integer>
	.forEach(System.out::println);
```

- What if we want the names?
	- can skip the map step
	
```java
// a second way of writing it
persons.stream()
	// .map(p -> p.getAge())
	.filter(p -> p.getAge() > 20)  // Stream<Person>
	.forEach(System.out::println);
```


**Intermediate & Terminal Calls**

Some examples

- this does not compile because forEach does not return anything

```java
persons.stream()
	.map(p -> p.getAge())
	.forEach(System.out::println) // !!! DOES NOT COMPILE !!!
	.filter(age -> age > 20)
	.forEach(System.out::println);
```

- use `peek()` instead

```java
persons.stream()
	.map(p -> p.getAge())
	.peek(System.out::println) // !!! DOES NOT COMPILE !!!
	.filter(age -> age > 20)
	.forEach(System.out::println);
```

- why not use it instead of the `forEach()` call?

```java
persons.stream()
	.map(p -> p.getAge())
	.peek(System.out::println)
	.filter(age -> age > 20)
	.peek(System.out::println);
```

- it does not print anything ...why?
	- `peek()` is an intermediate operation
	- `forEach()` is a terminal operation
	
	

**Terminal vs Intermediate Call**

- A terminal operation must be called to trigger the processing of a Stream
- No terminal operation = no data is ever processed

Check javadoc
Rule of thumb..
- A call that returns a Stream is an intermediate call
- A call that returns something else, or void is a terminal call that triggers the processing

**Skip and Limit example**

```java
persons.stream()
	.skip(2)
	.limit(3)
	.filter(person -> person.getAge() > 20)
	.forEach(System.out::println);
```

**Match Reductions**

These 3 are called short-circuiting terminal operations (they may not process all operations in stream)

- `anyMatch`

```java
List<Person> people = ...;
boolean b = 
people.stream()
	.anyMatch(p -> p.getAge() > 20);
```

Returns true if at least one element matches the predicate


- `allMatch()`

```java
List<Person> people = ...;
boolean b = 
people.stream()
	.allMatch(p -> p.getAge() > 20);
```

Returns true if all the elements match the predicate

- `noneMatch()`

```java
List<Person> people = ...;
boolean b = 
people.stream()
	.noneMatch(p -> p.getAge() > 20);
```

Returns true if no element matches the predicate


**Find Reductions**

- There are two types of find reduction: findAll() and findAny()
- They might have nothing to return:
	- If the stream is empty
	- Or if there is no value that matches the predicate

- So they both return an Optional, that can be empty


- `findFirst()`

```java
List<Person> people = ...;
Optional<Person> opt =
people.stream()
	.findFirst(p -> p.getAge() > 20);
```

Returns the first person, if any, wrapped in an Optional


- `findAny()`

```java
List<Person> people = ...;
Optional<Person> opt =
people.stream()
	.findAny(p -> p.getAge() > 20);
```

Returns any person, if it exists, wrapped in an Optional


**Reduce Reduction**

- There are three types of reduce reduction
- If no identity element is provided, then an Optional is returned
- Associativity is assumed for the reduction function, but not enforced


1) If identity is provided then return type is same as identity type

```java
List<Person> people = ...;
int sumOfAges =
people.stream()
	.reduce(0, (p1, p2) -> p1.getAge() + p2.getAge());
```

An identity element is provided, so the result is an int

2) If identity is provided then return type is an Optional

```java
List<Person> people = ...;
Optional<Integer> opt =
people.stream()
	.reduce((p1, p2) -> Integer.max(p1.getAge() + p2.getAge()));
```

3) parallel operations

```java
List<Person> people = ...;
List<Integer> ages =
people.stream()
.reduce(
	new ArrayList<Integer>(), 
	(list, p) -> { list.add(p.getAge()) ; return list ;}, 
	(list1, list2) -> { list1.addAll(list2) ; return list1 ; }
);
```
