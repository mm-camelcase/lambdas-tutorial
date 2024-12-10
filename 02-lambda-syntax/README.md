## Some Remarks on synthax
- One can put modifiers on the parameters of a lambda expression
	- The final keyword
	- Annotations 
- It is not possible to specify the returned type of a lambda expression (i.e. cannot cast it)
- We can also omit the types of the parameters, this code

```java
(String s1, String s2) -> {
	System.out.println("I am comparing strings");
	return Integer.compare(s1.length(), s2.length());
};
```

becomes

```java
(s1, s2) -> {
	System.out.println("I am comparing strings");
	return Integer.compare(s1.length(), s2.length());
};
```