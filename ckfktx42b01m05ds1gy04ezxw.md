## Abstract dependencies to reduce technical debt

In Object-Oriented programming language like java, an Object is created using a new keyword. But using the new keyword for all dependencies will make the class bind to that dependency for its whole lifetime.

For example, consider a class which has a dependency to connect to a database. Currently, if the application uses MySQL as database and all classes new connection to the database using new MySQL Connector class, your application is now deeply tied to MySQL database. 

If the database is being changed all classes needs to be refactored, to use the new database and it takes more time and effort. Rather if the get connector implementation is abstracted using design patterns such as  [Factory Pattern](https://en.wikipedia.org/wiki/Factory_method_pattern) ,  [Dependency Injection patter](https://santhoshmanikandan.hashnode.dev/so-whats-a-dependency-injection-1)  etc, doing a single change in the factory class will make all the underlying class to use the new database(dependency)

## So should new be never used?
It's not that you should abstract all the code which uses a *new *keyword. That will create multiple levels of abstraction which will make the code too difficult to understand. Using new is more of a design decision. If one is sure that the class under consideration is tied to its dependency for its lifetime new can be used. 
One example I could think of is a class which does some logs to a file. The custom logger class can new keyword for the file object. Abstracting file object here will be an overkill.
