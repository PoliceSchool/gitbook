本文非原创，原文来自[这里](https://www.javacodegeeks.com/2019/08/serialization-everything-java-serialization-explained.html)，本文只是简单地翻译，英文也不是太好，看到这篇文章觉得写得很好就想记录下来。

#### 什么是Serialization（序列化）？你应该知道的关于Java Serialization的解析，带例子的那种解析。

在前一篇文章里，我们看了[java里5种不同的方式创建对象](https://www.programmingmitra.com/2016/05/different-ways-to-create-objects-in-java-with-example.html),我已经解释了反序列化一个已序列化的对象的过程是如何创建一个新对象的，在今天这篇博客里，我将要详细的讨论Serialization和Deserialization。

我们将用下面这个`Employee` 类对象作为例子来解释Serialization和Deserialization。

```java
public class Employee implements Serializable {
    // This serialVersionUID field is necessary for Serializable as well as Externalizable to provide version control,
    // Compiler will provide this field if we do not provide it which might change if we modify the class structure of our class, and we will get InvalidClassException,
    // If we provide value to this field and do not change it, serialization-deserialization will not fail if we change our class structure.
    private static final long serialVersionUID = 2L;

    private final String firstName; // Serialization process do not invoke the constructor but it can assign values to final fields
    private transient String middleName; // transient variables will not be serialized, serialised object holds null
    private String lastName;
    private int age;
    private static String department; // static variables will not be serialized, serialised object holds null

    public Employee(String firstName, String middleName, String lastName, int age, String department) {
        this.firstName = firstName;
        this.middleName = middleName;
        this.lastName = lastName;
        this.age = age;
        Employee.department = department;

        validateAge();
    }

    private void validateAge() {
        System.out.println("Validating age.");

        if (age < 18 || age > 70) {
            throw new IllegalArgumentException("Not a valid age to create an employee");
        }
    }

    @Override
    public String toString() {
        return String.format("Employee {firstName='%s', middleName='%s', lastName='%s', age='%s', department='%s'}", firstName, middleName, lastName, age, department);
    }

    // Custom serialization logic,
    // This will allow us to have additional serialization logic on top of the default one e.g. encrypting object before serialization
    private void writeObject(ObjectOutputStream oos) throws IOException {
        System.out.println("Custom serialization logic invoked.");
        oos.defaultWriteObject(); // Calling the default serialization logic
    }

    // Custom deserialization logic
    // This will allow us to have additional deserialization logic on top of the default one e.g. decrypting object after deserialization
    private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
        System.out.println("Custom deserialization logic invoked.");

        ois.defaultReadObject(); // Calling the default deserialization logic

        // Age validation is just an example but there might some scenario where we might need to write some custom deserialization logic
        validateAge();
    }
}
```

#### 什么是Serialization和Deserialization？

在java的世界里，我们创建了几个对象之后，它们有可能存活也有可能死亡，如果JVM挂了的话，那么每个对象都会死亡。但有时候我们可能想在不同的JVM上重用这些对象或者说我们想要把这些对象通过网络的形式传送到另一台机器上。

既然这样，**serialization** 机制就允许我们把一个对象的状态（状态可以理解为对象里面的那些属性）转化成**byte stream**（字节流），而字节流能被存储在本地的磁盘