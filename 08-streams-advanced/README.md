# Advanced Streams 

## The Spliterator pattern

- What Is a Spliterator?
	- The object on which a Stream is built
	
- We can build our own bespoke stream processor using the Spliterator pattern

-Example: We want to construct people object from a text file that looks like this...

```
Alice
52
New York
Brian
25
Chicago
Chelsea
19
London
David
44
Paris
Erica
32
Berlin
Francisco
64
Mexico
```

- The standard file stream will just return 1 line <String> at a time.
	- What if we want to update it to return a person object each iteration...
	
```java
Path path = Paths.get("files/people.txt");
        
try (Stream<String> lines = Files.lines(path);) {
	
	Spliterator<String> lineSpliterator = lines.spliterator();
	Spliterator<Person> peopleSpliterator = new PersonSpliterator(lineSpliterator);
	
	Stream<Person> people = StreamSupport.stream(peopleSpliterator, false);
	people.forEach(System.out::println);
	
} catch (IOException ioe) {
	ioe.printStackTrace();
}
```

- We can grab a Stream, wrap it (i.e. create a Spliterator interface implementation and hold a ref to origial inside),  and change the behaviour to
	- iterate 3 lines at a time
	- return `Person()` objects
	- see code for `PersonSpliterator` impl
	

## File Concatenation Example

- Using `Stream.of` (`Stream.concat` can only take 2 params)
- `Stream.of` returns a Stream of Streams
- flatmap can unwrap it easily 
	- first case unrwap streams containing 4 other streams
	- second case unwrap stream of strings into words

```java
Stream<String> stream1 = Files.lines(Paths.get("files/TomSawyer_01.txt")) ;
Stream<String> stream2 = Files.lines(Paths.get("files/TomSawyer_02.txt")) ;
Stream<String> stream3 = Files.lines(Paths.get("files/TomSawyer_03.txt")) ;
Stream<String> stream4 = Files.lines(Paths.get("files/TomSawyer_04.txt")) ;

//        System.out.println("Stream 1 : " + stream1.count());
//        System.out.println("Stream 2 : " + stream2.count());
//        System.out.println("Stream 3 : " + stream3.count());
//        System.out.println("Stream 4 : " + stream4.count());


Stream<Stream<String>> streamOfStreams = 
	Stream.of(stream1, stream2, stream3, stream4);

//        System.out.println("# streams: " + streamOfStreams.count());
Stream<String> streamOfLines = 
	streamOfStreams.flatMap(Function.identity());	//NB: Function.identity() is the same as stream -> stream, ie fn that returns passed value

//        System.out.println("# lines " + streamOfLines.count());

Function<String, Stream<String>> lineSplitter = 
		line -> Pattern.compile(" ").splitAsStream(line);

Stream<String> streamOfWords = 
	streamOfLines.flatMap(lineSplitter)
		.map(word -> word.toLowerCase())
		.filter(word -> word.length() == 4)
		.distinct();

System.out.println("# words :" + streamOfWords.count());
```


## Stream State

- Remember a stream doesnt store  data
	- but it does have state called 'characteristics' as an encoded bit word
	
```java
Spliterator.ORDERED
Spliterator.SIZED
Spliterator.SUBSIZED
Spliterator.DISTINCT
etc.
```

- It is all about performance!

```java
// HashSet
HashSet<Person> people = ... ;
people.stream()
	.distinct() // no processing is triggered
	.sorted() 	// quicksort is triggered
	.forEach(System.out::println);
```

## Streams of Numbers

- we would like to compute the average of the ages our list of people

```java
// average of the ages of the people from our list
List<Person> people = ... ;
people.stream()                            
	.map(person -> person.getAge()) // Stream<Integer>
	.filter(age -> age > 20)
	.average();					// no method for this on Stream<T>
```


- How to convert a Stream<Integer>to an IntStream?

```java
// average of the ages of the people from our list
List<Person> people = ... ;
people.stream()                            // average
	.map(person -> person.getAge()) // Stream<Integer>
	.filter(age -> age > 20)
	.mapToInt(i -> i) // IntStream
	.average();
```

but above mapToInt will do auto un boxing for each element so slow

```java
// average of the ages of the people from our list
List<Person> people = ... ;
people.stream()                            // average
	.mapToInt(person -> person.getAge()) // IntStream
	.filter(age -> age > 20)
	// .mapToInt(i -> i)
	.average();
```

this is much faster


What Is a Stream of Numbers?
- Streams of numbers are there to avoid the cost of boxing / unboxing
- Three types: IntStream, LongStreamand DoubleStream
- The patterns to build them are simple:

```java
// build from a varargs
LongStream streamOfLongs = LongStream.of(1L, 2L, 3L);
```

```java
// convert from a Stream<Integer>
IntStream streamOfInts = people.stream().mapToInt(Person::getAge);
```

```java
// box a Stream if needed
Stream<Long> boxedStream = LongStream.of(1L, 2L, 3L).boxed();
```

```java
// box a Stream if needed
Stream<Long> boxedStream = LongStream.of(1L, 2L, 3L).mapToObj(l -> l);
```

- Stream of numbers have special methods, not on Stream<T>

```java
// on IntStream
int sum = intStream.sum();
OptionalInt min = intStream.min();
OptionalInt max = intStream.max();
OptionalDouble average = intStream.average();
IntSummaryStatistics stats = intStream.summaryStatistics();
```

- Summary statistics compute sum, min, max, count and average in one pass

## TODO: Parallel Stream Processing 