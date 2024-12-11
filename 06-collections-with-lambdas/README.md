## Collections with lambdas


**List Examples**

- Using new forEach, removeIf, replaceAll, sort methods

```java
Person p1 = new Person("Alice", 23);
Person p2 = new Person("Brian", 56);
Person p3 = new Person("Chelsea", 46);
Person p4 = new Person("David", 28);
Person p5 = new Person("Erica", 37);
Person p6 = new Person("Francisco", 18);

List<Person> people = new ArrayList(Arrays.asList(p1, p2, p3, p4, p5, p6));

people.removeIf(person -> person.getAge() < 30);

people.replaceAll(person -> new Person(person.getName().toUpperCase(), person.getAge()));

//people.sort(Comparator.comparing(person -> person.getAge()));
people.sort(Comparator.comparing(Person::getAge)); //sort desc
people.sort(Comparator.comparing(Person::getAge).reversed()); //sort asc

//people.forEach(p -> System.out.println(p));
people.forEach(System.out::println);
```

**Map Examples**

```java
Person p1 = new Person("Alice", 23);
Person p2 = new Person("Brian", 56);
Person p3 = new Person("Chelsea", 46);
Person p4 = new Person("David", 28);
Person p5 = new Person("Erica", 37);
Person p6 = new Person("Francisco", 18);

City newYork = new City("New York");
City shanghai = new City("Shanghai");
City paris = new City("Paris");

Map<City, List<Person>> map1 = new HashMap<>();
map1.computeIfAbsent(newYork, city -> new ArrayList<>()).add(p1);
map1.computeIfAbsent(shanghai, city -> new ArrayList<>()).add(p2);
map1.computeIfAbsent(shanghai, city -> new ArrayList<>()).add(p3);

System.out.println("Map 1");
map1.forEach((city, people) -> System.out.println(city + " : " + people));


Map<City, List<Person>> map2 = new HashMap<>();
map2.computeIfAbsent(shanghai, city -> new ArrayList<>()).add(p4);
map2.computeIfAbsent(paris, city -> new ArrayList<>()).add(p5);
map2.computeIfAbsent(paris, city -> new ArrayList<>()).add(p6);

System.out.println("Map 2");
map2.forEach((city, people) -> System.out.println(city + " : " + people));

// merge people from map 2 into map 1
// merge method signature ... merge(K key, V value, BiFunction<? super V,? super V,? extends V> remappingFunction)
// note BiFunction acts on values from map 1 and map 2
map2.forEach(
		(city, people) -> {
			map1.merge(
					city, people, 
					(peopleFromMap1, peopleFromMap2) -> {
						peopleFromMap1.addAll(peopleFromMap2);
						return peopleFromMap1;
					});
		}
);

System.out.println("Merged map1 ");
map1.forEach(
		(city, people) -> System.out.println(city + " : " + people)
);
```

## Map, Filter, Reduce Pattern

Here we focus on Reduction because it has some gotchas
We use a theoretical implementation of Map, Filter, Reduce using jdk7 style factories do demonstrate design issues that needed to be solved
(see C:\workspace\tutorials\lambda_collections_streams\lambda\src\main\java\com\mm\lambda for code examples)

Example 
- **Example** Let us compute the average of the age of people older than 20 (from some list of people objects)
	- **Map** e.g. `List<Person> → List<Integer>` [get ages]...........same size, different type (maybe...could be different value)
	- **Filter** e.g. `List<Integer> → List<Integer>` [filter]............smaller size, same types
	- **Reduce** e.g. `List<Integer> → Integer` [aggregate].............differnt size and type e.g. aggregate fn like sum




**Reduce**
- This algorithm is easily computed in parallel, which makes it fast, however...
- **NB:** There are restrictions....
	**1)** The lambda expression passed to reduce must be associative 
		- In programming languages, the associativity of an operator is a property that determines how operators of the same precedence are grouped in the absence of parentheses
	
```java
// Which of the following are associative expressions
BinaryOperator<Integer> op1 = (i1, i2) -> i1 + i2;					// yes
BinaryOperator<Integer> op2 = (i1, i2) -> Integer.max(i1, i2);		// yes
BinaryOperator<Integer> op3 = (i1, i2) -> i1*i1 + i2*i2;			// no
BinaryOperator<Integer> op4 = (i1, i2) -> i1;						// yes
BinaryOperator<Integer> op5 = (i1, i2) -> (i1 + i2)/2;				// no
```

Example of non-associative operation with parallel processing simulation...

```java
List<Integer> ints = Arrays.asList(0, 1, 2, 3, 4, 5, 6, 7, 8, 9);
        
List<Integer> ints1 = Arrays.asList(0, 1, 2, 3, 4);
List<Integer> ints2 = Arrays.asList(5, 6, 7, 8, 9);

//BinaryOperator<Integer> op = (i1, i2) -> Integer.max(i1, i2) ; // associative op (works fine)
BinaryOperator<Integer> op = (i1, i2) -> (i1 + i2) * (i1 + i2) ; // non-associative op

int reduction1 = reduce(ints1, 0, op);	// cpu1 simulation
int reduction2 = reduce(ints2, 0, op);	// cpu2 simulation
int reductionMerged = reduce(Arrays.asList(reduction1, reduction2), 0, op);
int reduction = reduce(ints, 0, op);

System.out.println("Reduction : " + reduction);				//prints Reduction : 791706833
System.out.println("Reduction Merged : " + reductionMerged);//prints Reduction : 502703265
```

   **2)** Using reduction that has no identity element
		- Identity – an element that is the initial value of the reduction operation and the default result if the stream is empty

Example of incorrect identity for operation

```java
List<Integer> ints = Arrays.asList(-1, -2, -3, -4);
        
BinaryOperator<Integer> op = (i1, i2) -> Integer.max(i1, i2) ;

// int reduction = reduce(ints, 0, op); // wrong, ans would be 1 
int reduction = reduce(ints, Integer.MIN_VALUE, op); // correct -1

System.out.println("Reduction : " + reduction);
```

**Conclusion on the Reduction Step**
- The reduction is critical so be extra careful
- It is very easy to write a non-associative reduction
- It is very easy to write a reduction with no identity element


## Theoretical Design of Map, Filter, Reduce Pattern

Imagine we implemeted using static methods on a factory class called lists

```java
List<Person> people = ... ;
List<Integer> ages = Lists.map(people, person -> person.getAge());
List<Integer> agesGT20 = Lists.filter(ages, age -> age > 20);
int average = Lists.reduce(agesGT20, (a1, a2) -> a1 + a2);
```

- 2 List duplications : ages and agesGT20
- High memory footpring, CPU load!...(imagine millions of records)


We would like to write using this style...

```java
List<Person> people = ... ;
int average = people
	.map(p -> p.getAge())
	.filter(age -> age > 20)
	.average();
```

But would have same issues as above i.e.  duplicating and reprocessing collections

Solution is **Streams** 
- The call to stream() returns a Stream, a new interface in Java 8

```java
List<Person> people = ... ;
int average = people.stream()
	.map(p -> p.getAge())
	.filter(age -> age > 20)
	.average();
```