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

既然这样，**serialization** 机制就允许我们把一个对象的状态（状态可以理解为对象里面的那些属性）转化成**byte stream**（字节流），而字节流能被存储在本地的磁盘或者通过网络发送给其他主机。而**deserialization** 机制允许我们将**serialization**这一过程逆转，也就是说能把已经序列化的**byte stream**再次转化为对象。

简单来说，**object serialization**(对象序列化)是将对象的状态保存为字节序列的过程，而**deserialization** 就是用这些**byte stream**重新生成对象的过程。一般来说，完整的过程称为序列化，但我认为最好将两者分开进行分类，以便更清楚地说明问题。

**serialization** 过程是平台无关的，也即是说在这一个平台上序列化的对象能够在另一个平台上反序列化。

为了能使对象序列化为一个文件或者从一个文件中反序列化成一个对象，我们需要调用`ObjectOutputStream.writeObject()`和`ObjectInputStream.readObject()`方法，就像下面代码做的一样：

```java
public class SerializationExample {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Employee empObj = new Employee("Shanti", "Prasad", "Sharma", 25, "IT");
        System.out.println("Object before serialization  => " + empObj.toString());

        // Serialization
        serialize(empObj);

        // Deserialization
        Employee deserialisedEmpObj = deserialize();
        System.out.println("Object after deserialization => " + deserialisedEmpObj.toString());
    }

    // Serialization code
    static void serialize(Employee empObj) throws IOException {
        try (FileOutputStream fos = new FileOutputStream("data.obj");
             ObjectOutputStream oos = new ObjectOutputStream(fos))
        {
            oos.writeObject(empObj);
        }
    }

    // Deserialization code
    static Employee deserialize() throws IOException, ClassNotFoundException {
        try (FileInputStream fis = new FileInputStream("data.obj");
             ObjectInputStream ois = new ObjectInputStream(fis))
        {
            return (Employee) ois.readObject();
        }
    }
}
```

#### 只有实现了Serializable 接口的类才能被序列化？

正如**Cloneable** 接口一样，我们有一个**marker interface**(标记接口)，那就是**Serializable** ；这个接口对JVM来说是一个标记。直接实现Serializable 接口或者通过父类间接实现了Serializable 接口的类都能被序列化，而没有实现Serializable 的类不能被实例化。

Java默认的序列化过程是递归调用的，所以不论何时我们序列化一个对象，序列化过程都会序列化类中的所有字段(基本类型和引用类型)，除了static字段和transient。

当一个类实现了序列化接口后，它的子类全部是可序列化的。如果一个对象A持有另一个对象B的引用，那么B必须要实现序列化接口。在序列化时哪怕我们的类中有一个引用属性不能被序列化JVM都会抛`NotSerializableException`异常。

#### 为什么Object类不直接实现Serializable 接口？

如果序列化是一个普遍的功能并且没实现序列化接口的类不能被序列化的话，那么有一个问题，那就是为什么Object类不直接实现Serializable 接口？这样的话，所有的对象都默认能够序列化了。

Object类不实现序列化接口的原因是不是所有的对象都需要序列化的。比如序列化一个Thread对象是没有意义的，因为一个线程运行在我的JVM里面，用的是我的系统内存，持久化这个Thread对象然后将它运行在你的JVM里面是没有用的。

####  transient字段和static字段不会被序列化

如果我们想序列化一个对象但又不想序列化某个字段时，我们可以把这个字段设置为**transient**。

所有的**static**字段属于类而非对象实例，而序列化过程序列化的是对象实例，所以**static**字段不会被序列化。

1. 序列化一个对象实例时，对象实例的字段的访问修饰符是不起作用的，比如类中有个被**private**修饰的字段，那这个字段仍然可以被序列化。所有的非**transient**和非**static**的字段都是对象实例的可持久化状态的一部分，并且都可被序列化。
2. 对于对象实例中的final字段，原本只能通过构造方法赋值。但是反序列化的时候是不会调用对象实例的构造方法的，但是final字段仍然可以被赋值。

#### 什么是**serialVersionUID** ？为什么我们要声明它？

假设我们有个类，并且这个类没有声明**serialVersionUID**，并且已经把这个类的某个对象实例序列化到了磁盘的某个文件里。由于一些新的需求，我们新增/删除了类中的某些字段。现在我们通过文件再去反序列化对象实例，我们会得到一个`InvalidClassException`异常。为什么？

如果我们没有在我们的类里面声明**serialVersionUID**的话，JVM会默认为每一个可序列化的类自动生成一个**version number**，这个**version number**用来控制类的版本的。这个字段用来校验那些具有相同属性的类的，并以此来选择相对应的反序列化方式。这个**version number**由一个**serialVersionUID** 字段维护，如果一个序列化类没有声明**serialVersionUID** 的话，JVM会在运行时自动生成一个。

如果我们改变了类的结构，例如删除/新增字段的话**version number**也会相应的改变。因此，由于JVM的存在我们的类版本已经不能和已经序列化的类的版本号兼容了。这就是为什么我们会得到一个异常的原因。但如果你仔细想一想，为什么就因为我加了一个字段就会抛异常呢？难道不能把那个字段设置为默认值然后在下一次序列化的时候再把这个字段写回去吗？

答案是可以的，只要你手动提供一个**serialVersionUID**并且确保它总是一致就可以了。这里非常推荐每一个可序列化的类都声明一个**serialVersionUID**，因为自动生成的**serialVersionUID**是编译器相关的，而这可能导致**InvalidClassExceptions**异常。

你可以使用JVM附带的程序`serialver` 看到默认的号码，这其实就是object对象的hashcode。

#### 通过**writeObject** 和 **readObject** 方法自定义序列化和反序列化

JVM在默认的序列化过程中完全控制了对象的序列化，而使用JVM默认的序列化会有以下几个缺点：

1. 不能处理某些序列化过程，例如类中有些字段不能被序列化
2. 反序列的过程中不会调用构造方法就构造了一个对象，所以如果构造方法中有某些逻辑代码的话是不会被调用的。

但是我们可以在我们的java类中override默认的序列化行为，通过提供额外的逻辑去增强正常的处理流程。通过在类中提供两个方法就可以完成就可以完成上述的功能，这两个方法分别是**writeObject** 和**readObject**。

```java
// Custom serialization logic will allow us to have additional serialization logic on top of the default one e.g. encrypting object before serialization
private void writeObject(ObjectOutputStream oos)throws IOException{
    // Any Custom logic
    oos.defaultWriteObject(); // Calling the default serialization logic
    // Any Custom logic
    }

// Custom deserialization logic will allow us to have additional deserialization logic on top of the default one e.g. decrypting object after deserialization
private void readObject(ObjectInputStream ois)throws IOException,ClassNotFoundException{
    // Any Custom logic
    ois.defaultReadObject(); // Calling the default deserialization logic
    // Any Custom logic
    }
```

这两个方法必须声明为private的，这样的话除了JVM其他类是不能调用这两个方法的。这也证明了这两个方法不是继承而来的，也不是重写的，也不是重载的。JVM会在序列化或者反序列化的过程中自动检查这两个方法。JVM能够调用这些private方法，而其他对象不能，因此这保证了类的完整性并且序列化协议能够正常的进行。

即使提供了这些特殊的private方法，类的序列化仍然是跟之前是一样的，那就是调用`ObjectOutputStream.writeObject()` 或 `ObjectInputStream.readObject()`.

调用`ObjectOutputStream.writeObject()` 或`ObjectInputStream.readObject()`就启动了序列化协议过程。首先这个对象实例会被检查是否实现了Serializable 接口，然后查看类里面是否提供了上述的两个方法，如果提供了的话stream对象会作为参数传入方法中，把代码控制权交给这两个方法（类似于控制反转）。

我们可以在上述的方法里调用`ObjectOutputStream.defaultWriteObject()` 和
`ObjectInputStream.defaultReadObject()`方法，这样就能调用默认的序列化逻辑。defaultWriteObject和defaultReadObject这两个方法就像它们的名字一样----它们对序列化对象执行默认的**writing** and **reading**操作，这很重要，因为我们不是替换原来的处理过程，而是在它的基础之上添加我们自己的东西。

在序列化处理过程汇总，这些private方法能够让你自定义你想要的东西。比如你可以在输出的时候加密，在读取的时候解密（注意，这些字节流都是以纯文本方式读写的，没有任何的加密）。这些private方法能够将额外的数据加入stream中，可能是公司版本号。

#### 停止序列化和反序列化

假设有一个类A从它的父类那里得到了可序列化的能力，也就是说因为它的父类实现了`Serializable`接口所以它也能被序列化。

这就意味着任何人能够序列化和反序列化类A。但如果我们不想让类A的实例被序列化或者反序列化呢？例如类A是一个单例模式的类，然后我们不想让其他类创建类A的实例，要记得**反序列化过程不用通过构造方法就能够生成一个实例**。要停止序列化和反序列化功能，我们可以再一次使用前面讲过的那两个private方法。我们可以在那两个private方法里面抛出`NotSerializableException`异常。任何想尝试序列化或者反序列化类A实例的操作现在都会导致一个异常。又因为这两个方法是private的，所以不用担心这两个方法被重载。

```java
private void writeObject(ObjectOutputStream oos) throws IOException {
        throw new NotSerializableException("Serialization is not supported on this object!");
        }

private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
        throw new NotSerializableException("Serialization is not supported on this object!");
        }
```

#### 结论

1. 序列化的含义是将一个类的状态转化为一串字节序列的过程，而这串字节序列能被保存在一个文件中或者能够通过网络传输。反序列化的含义是将那些字节序列重新构造成对象的过程。
2. 只有实现了`Serializable` 接口的类才能被序列化。
3. 如果我们的类没有实现`Serializable` 接口或者类中有些引用字段不是可序列化的，那么JVM会抛一个`NotSerializableException`.异常。
4. 所有的`transient` 和 `static` 字段不会被序列化。
5. `serialVersionUID` 用来验证已序列化的对象跟当前类的对象那个是否有相同的属性，以此来判断是否跟反序列化过程兼容。
6. 我们应该在我们的类中手动创建一个`serialVersionUID` 字段，这样的话我们以后改变我们的类结构的时候（增加/删除字段），JVM不会抛异常。如果我们不提供该字段，将来如果我们改动了类结构的话JVM会自动生成一个值。
7. 通过在我们的类里面实现 `writeObject` 和`readObject` 这两个方法的逻辑，我们可以重载默认的序列化行为。
8. 然后我们可以从 `writeObject` 和`readObject` 方法里面分别调用 `ObjectOutputStream.defaultWriteObject()` 和 `ObjectInputStream.defaultReadObject` 方法，以获得默认序列化和反序列化的逻辑。
9. 如果我们不想我们的类被序列化或者反序列化，我们可以在 `writeObject` 和`readObject` 方法里面抛出 `NotSerializableException` 异常。

通过实现`Externalizable`接口能够更好的自定义或者增强JAVA的序列化过程。这部分在[这篇文章](https://www.programmingmitra.com/2019/08/how-to-customize-serialization-in-java-by-using-externalizable-interface.html)里面有做介绍。



拓展：

Spring有将我们的http response序列化吗？有的话是如何序列化的？

dubbo有序列化接口吗？为什么dubbo要自己弄一套序列化接口？dubbo的分布式是如何实现的？

序列化广泛用于什么地方？