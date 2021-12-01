## Pattern Matching in instanceof

The traditional usage of instanceof operator requires typecasting inside the if block, i.e.,

```
if (obj instanceof String) {
    String s = (String) obj;
    ...
}
```

Though the block inside if gets executed only if obj X is a type of String object, the java language expects us to typecast it to String before working on that object. With the introduction of pattern matching, Java eliminates this explicit typecasting.

From the docs, a pattern is defined as, 

> A pattern is a combination of (1) a predicate, or test, that can be applied to a target, and (2) a set of local variables, known as pattern variables, that are extracted from the target only if the predicate successfully applies to it.

In accordance with the above definition, the above instanceof usage can be simplified as follows,

```
if (obj instanceof String s) {
    System.out.println(s);
    ...
}
```

If obj(**pattern variable**) is an instance of String(**test**), then it is cast to String and the value is assigned to the variable s(**applied to target**).

If the normal scope of a variable is applied to pattern variables, in the following example, the second condition cannot use the same variable name. This will in turn require one to find unique names for each step. To overcome this flow scoping(similar to  [definite assignment](https://docs.oracle.com/javase/specs/jls/se15/html/jls-16.html) ) is used for pattern variables. A pattern variable is only in scope where the compiler can deduce that the pattern has definitely matched and the variable will have been assigned a value. 
```
if (obj instanceof String s) {
    System.out.println(s);
    ...
}
if (obj instanceof String s) {  //err
    System.out.println(s);
    ...
}
```

To conclude, the use of pattern matching in instanceof should significantly reduce the overall number of explicit casts in Java programs. 

From,

```
public boolean equals(Object o) {
    return (o instanceof CaseInsensitiveString) &&
        ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
````

To,

```
public boolean equals(Object o) {
    return (o instanceof CaseInsensitiveString cis) &&
        cis.s.equalsIgnoreCase(s);
}
```

Further Reading: [JEP 394](https://openjdk.java.net/jeps/394)