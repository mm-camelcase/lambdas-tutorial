## Collecting Data in Complex Containers Using Collectors


- What is a Collector
	- Reduction in a container
	
- The reductions we saw were aggregations: sum, max, average, etc…
- A collector is a special type of reduction
- It is a terminal operation, that triggers the computation of the Stream


Example - Lets collect a stram of strings in a list
	- we saw already - dont do this...

```java
List<String> peopleNames = ... ;
List<String> result = new ArrayList<>() ;
peopleNames.stream()
	.filter(s -> !s.isEmpty())
	.forEach(s -> result.add(s)) ;
```

do this...

```java
List<String> peopleNames = ... ;

List<String> result = 
peopleNames.stream().parallel()		// parallel fine if required
	.filter(s -> !s.isEmpty())
	.collect(Collectors.toList()) ;
```

- This step is called mutable collection
- Just because it collects the data in a mutable container


### The Collectors Class

- The JDK provides a factory class: Collectors
- Collecting data is about gathering data in a mutable (changable) container
	- A String (concatenation)
	- A Collection (adding)
	- A HashMap (grouping by a criteria)


- Collector is the interface that models the collectors
- Collectors is the class factory to build collectors
	- Most collectors can be built through the factory
	- If needed there are other patterns
	

**Collecting a Max**

```java
List<Person> people = ... ;

Optional<Person> oldest = 
	people.stream() 
	.collect(
		Collectors.maxBy(Comparator.comparing(p -> p.getAge()))
	) ;
```

**Collecting an Average**

```java
List<Person> people = ... ;

double average = 
	people.stream() 
	.collect(
		Collectors.averagingDouble(p -> p.getAge())
	) ;
```

**Collecting in a String**

```java
List<Person> people = ... ;

String names = 
	people.stream()
	.map(p -> p.getName())
	.collect(Collectors.joining(", ")) ;
	
prints: Barbara, Charles, Sharon, Peter
```

**Collecting in a Set**

```java
List<Person> people = ... ;

Set<String> names = 
people.stream()
	.map(p -> p.getName())
	.collect(Collectors.toSet()) ;
```

**Collecting in a Collection**

```java
List<Person> people = ... ;

TreeSet<String> names = 
people.stream()
	.map(p -> p.getName())
	.collect(Collectors.toCollection(() -> new TreeSet())) ;
```

**Collecting in a Map**

- Partioning by a predicate:

```java
List<Person> people = ... ;

Map<Boolean, List<Person>> peopleByAge = 
	people.stream()
	.collect(Collectors.partitioningBy(person -> person.getAge() > 21)) ;
```

**Collecting in a Map**

- Grouping by a function:

```java
List<Person> people = ... ;

Map<Integer, List<Person>> peopleByAge = 
	people.stream()
	.collect(Collectors.groupingBy(person -> person.getAge())) ;
```

**Collecting in a Map**

- Grouping and counting:

```java
List<Person> people = ... ;

Map<Integer, Long> peopleByAge = 
	people.stream()
	.collect(
		Collectors.groupingBy(person -> person.getAge()),
		Collectors.counting()		// the « downstream » collector
	) ;
```

**Collecting in a Map**

- Grouping and mapping:

```java
List<Person> people = ... ;

Map<Integer, List<String>> namesByAge = 
	people.stream()
	.collect(
		Collectors.groupingBy(person -> person.getAge()),
		Collectors.mapping(
			person -> person.getName()
		)
	) ;
```

**Collecting in a Map**

- Grouping, mapping and collecting in a TreeSet(i.e. names are sorted):

```java
List<Person> people = ... ;

Map<Integer, TreeSet<String>> namesByAge = 
	people.stream()
	.collect(
		Collectors.groupingBy(person -> person.getAge()),
		Collectors.mapping(
			person -> person.getName(), 
			Collectors.toCollection(() -> TreeSet())
		)
	) ;
```

**Collecting in a Map**

- Grouping in a TreeMap, mapping and collecting in a TreeSet(i.e. ages and names are sorted):

```java
List<Person> people = ... ;

TreeMap<Integer, TreeSet<String>> namesByAge = 
	people.stream()
	.collect(
		Collectors.groupingBy(person -> person.getAge()),() -> new TreeMap(), 
		Collectors.mapping(
			person -> person.getName(), 
			Collectors.toCollection(() -> TreeSet())
		)
	) ;
```

**Collecting in an Immutable Map**

- Collecting in an immutable Map:

```java
List<Person> people = ... ;

Map<Integer, List<Person>> peopleByAge = 
	people.stream()
	.collect(
		Collectors.collectingAndThen(
			Collectors.groupingBy(person -> person.getAge()), 
			Collections::unmodifiableMap
		)
	) ;
```

i.e. you need to use `collectingAndThen` to creat the map once at the end of collecting