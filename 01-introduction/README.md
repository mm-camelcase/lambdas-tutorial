## Why?
- Allow us to forego anonymous classes, greatly reducing boilerplate code and improving readability
- When you want to declare a piece of code to run at some later point e.g. comparator you generally use an anonymous inner class 

- A basic Comparator

```java
Comparator<String> comparator = new Comparator<String>() {
	public int compare(String s1, String s2) {
		return Integer.compare(s1.length(), s2.length());
	}
};

Arrays.sort(tabStrings, comparator);
```

- What did we do?
	- We wrote some code in an anonymous class
	- And we passed it to another piece of code
	- That executed it “later”
	- Or in another context (thread) if we choose (e.g. `Executors.newSingleThreadExecutor().execute(r);` for a Runnable)

	- **NB: We passed code as a parameter**
	- And we used anonymous class, because it is the only way to do it in Java
	
## Examples of rewriting code as lambda expressions

- Comparator example

```java
Comparator<String> comparator = new Comparator<String>() {
	public int compare(String s1, String s2) {
		return Integer.compare(s1.length(), s2.length());
	}
};
```

Becomes...

```java
Comparator<String> comparator =
	(String s1, String s2) -> Integer.compare(s1.length(), s2.length());
```

- Runnable example (more than 1 line of code)
	
```java
Runnable r = new Runnable() {
	public void run() {
		int i = 0;
		while (i++ < 10) {
			Sytem.out.println("It works!");
		}
	}
};
```

Becomes...

```java
Runnable r = () -> {
	int i = 0;
	while (i++ < 10) {
		Sytem.out.println("It works!");
	}
};
```

- In the case there is a returned value
	
```java
(String s1, String s2) -> {
	System.out.println("I am comparing strings");
	return Integer.compare(s1.length(), s2.length());
};
```
