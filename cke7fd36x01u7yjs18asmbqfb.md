## So what's a dependency injection

Consider this situation, a class named EmployeeList which reads a list of employee names from files

```
public class Employee {
     ...

     List<Employee> getEmployeesJoinedAfterYear(int year) {
        String content = readFromFile("employees.txt");
        List<Employee> employee = parseAndGetData(content); 
        //check for name and send back required list.
     }
     ...
}
```

In the above example, assume that the way how data of employees is read is from a text file separated with ;(semicolon). So now the Employee class depends on the way the data is stored(*that's a dependency*).

If a developer using the above class decides to go with database or read data from XML file, the source code needs to be changed so that data can be obtained properly. This looks simple for this case, but in case of huge projects, things can get difficult and one might end up refactoring lot of code to change the dependency.

What if Employee class provides a way for us to **inject** the list of employees to it? If the class supports its, that's a dependency injection. Dependency can be injected in three ways, using constructor, using setter, using Interface. Of all three and simpler and recommended way is to go with constructor method as it tells clearly during initialization of a class that it depends on X.


```
public Interface EmployeeDataProvider {
    public List<Employee> readAndParseData();
}

public class EmployeeFileReader implements EmployeeDataProvider {
    @override
    publc List<Employee> readAndParseData() {
         String content = readFromFile("employees.txt");
        List<Employee> employee = parseAndGetData(content);  //Impl of parsing data is skipped for brevity
    }
}
```

The above is a simple interface and a class which read data from a txt file and give back list of employees. Now the Employee class is changed to support option to inject dependency using constructor.


```
public class Employee {
    EmployeeDataProvider provider;
    public Employee( EmployeeDataProvider provider ) {
        this.provider = provider;
    }
    ...

    List<Employee> getEmployeesJoinedAfterYear(int year) {
       List<Employee> employee = provider.readAndParseData();
       //check for name and send back required list.
    }
    ...
}

public class EmployeeTest {
    public static void main(String[] args) {
        EmployeeDataProvider provider = new EmployeeFileReader();
        Employee emp = new Employee(provider);
    }
}
```

There are few injection frameworks like [PicoContainer](http://picocontainer.com/),  [Google Guice](https://github.com/google/guice) which can be used to inject dependency instead of our own custom implementation like the above example.

One might think why should one separate dependency from the class which requires more number of lines. Here are few things which I could think of,

1. Make sure  [Singe Responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle)  is not violated.
2. Makes code testable(dependency can be mocked if needed).
3. Class doesn't need to know internals of a dependency(email/SMS/call alert or file read/database read) and different implementation can be injected without altering the class which depends on it.

#design-pattern #java #dependency-injection 



 