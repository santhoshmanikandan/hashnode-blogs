## Pattern matching comes to Switches in Java

Switches in Java lacked an option to match a value based on an expression that is computed at runtime(similar to C). So the values against which a value can be matched should be constant known at compile time.

Below is the current format of switch that is allowed until pattern matching was added to switch

```
switch(status) {
    case 1: ....
    case 2: ....
}
```

So if one needs to do branching on expressions, multiple if-else statements are needed as shown below,

```
void log(Object o, String text) {
    if (o instanceof FileLogger f) {
        f.writeToFile(text);
    } else if (o instanceof SysLogger s) {
       s.writeToSyslog(text);
    } else if (o instanceof CentralLogger c) {
        c.writeToCentralLogger(text);
    }
}
```

Though the above works as expected, in places where a variable gets assigned to a value, a block can miss setting the value and still the code is compilable(but with an error). Also if the compiler doesn't do optimizations, it could be an O(n) time-space.


```
void log(Object o, String text) {
    switch(o) {
        case FileLogger f     -> f.writeToFile(text);
        case SysLogger s      -> s.writeToSyslog(text);
        case CentralLogger c  -> c.writeToCentralLogger(text);
        default               -> o.writeToConsoe(text);
    }
}
```

### Pattern matching and null

Traditionally, switch statements and expressions throw NullPointerException if the selector expression evaluates to null. To handle this, testing for null must be done outside of the switch:

```
static void testFooBar(String s) {
    if (s == null) {
        System.out.println("oops!");
        return;
    }
    switch (s) {
        case "Foo", "Bar" -> System.out.println("Great");
        default           -> System.out.println("Ok");
    }
}
```

With the switch allowing selector expression of any type, and case labels can have type patterns, then the standalone null test for all the possible expressions will lead to boilerplate and opportunity for error. to overcome this, a null test was added to the switch:

```
static void testFooBar(String s) {
    switch (s) {
        case null         -> System.out.println("Oops");
        case "Foo", "Bar" -> System.out.println("Great");
        default           -> System.out.println("Ok");
    }
}
```

If the null case is not added, the switch falls back to default implementation and throws NullPointerException.

### Guarded Patterns and Parenthesized Patterns

A guarded pattern enables a pattern to be refined with a boolean expression. A value matches a guarded pattern if it matches the pattern and the boolean expression evaluates to true.

```
void height(Object obj) {
        switch (obj) {
            case Integer s:
                if (s <= 100) {
                    System.out.println("Short person " + s);
                } else {
                    System.out.println("Tall Person " + s);
                }
                break;
            default:
                System.out.println("Not a valid height");
        }
    }
```
With the power of expressions made available in case statements, the above condition can be simplified as below

```
void height(Object obj) {
        switch (obj) {
            case Integer s &&  s <= 100 -> System.out.println("Short person " + s);
            case Integer s              -> System.out.println("Tall Person " + s);
            default                     -> System.out.println("Not a valid height");
        }
    }
```

