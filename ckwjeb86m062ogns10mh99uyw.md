## Java Records

Java language is known for being too much verbose which results in writing an ample number of lines to store immutable data fetched from a database or URL connection for processing. Java records(first introduced in J14 and finalized in J16) provide an easier way to declare data carriers.

### Java Grammer

**RecordDeclaration:**

  {ClassModifier} `record` TypeIdentifier [TypeParameters]
    RecordHeader [SuperInterfaces] RecordBody

**RecordHeader:**

 `(` [RecordComponentList] `)`

**RecordComponentList:**

 RecordComponent { `,` RecordComponent}

**RecordComponent:**

 {Annotation} UnannType Identifier
 VariableArityRecordComponent

**VariableArityRecordComponent:**

 {Annotation} UnannType {Annotation} `...` Identifier

**RecordBody:**

  `{` {RecordBodyDeclaration} `}`

**RecordBodyDeclaration:**

  ClassBodyDeclaration
  CompactConstructorDeclaration

**CompactConstructorDeclaration**:

  {ConstructorModifier} SimpleTypeName ConstructorBody

Java record is nothing but a special class that the compiler generates based on its declaration. All the auto-created Java record classes extend java.lang.Record object and are declared as a final class.

During compilation, the following are automatically created for a Java Record.


- For each component in the header, two members: a public accessor method with the same name and return type as the component, and a private final field with the same type as the component.
- A canonical constructor whose signature is the same as the header, and which assigns each private field to the corresponding argument from a new expression that instantiates the record.
- equals and hashCode methods which ensure that two record values are equal if they are of the same type and contain equal component values.
- toString method that returns a string representation of all the record components, along with their names.

Any of the above automatically created methods can be overridden if needed. 

A sample class created by the compiler for a record class is shown below.

![record.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1638111110979/OMbCJREzC.png)

In case validation/processing needs to be done on the supplied parameters to a constructor, records provide an easier way to write the implementation part alone leaving out the assignments of parameters passed. This in turn saves time and avoids errors where a variable assignment gets missed or assigned wrongly.


```
record Range(int lo, int hi) {
        Range {
            if (lo > hi)
                throw new IllegalArgumentException(String.format("(%d,%d)", lo, hi));
        }
}
``` 

Similar to any Java type, records come with some restrictions on their usage.
-Records cannot extend any class.
- Records cannot declare instance fields (other than the private final fields that correspond to the components of the record component list); any other declared fields must be static.
- Records cannot be abstract; they are implicitly final.
- The components of a record are implicitly final.


Further reading
-  [JEP 395](https://openjdk.java.net/jeps/395)
- [Records Come to Java](https://blogs.oracle.com/javamagazine/post/records-come-to-ja
va)





