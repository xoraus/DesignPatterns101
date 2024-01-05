# DesignPatterns101

* [Design Pattern: Singleton](#singleton-design-pattern)
* [Design Pattern: Builder Design Pattern](#builder-design-pattern)
* [Design Pattern: Prototype Design Pattern](#prototype-design-pattern)
* [Design Pattern: Factory Design Pattern](#factory-design-pattern)


## Singleton Design Pattern

### Problem

- **Shared Resource:** Imagine having a class responsible for managing the database connection or logging mechanism. You want to ensure that only one instance of this class exists to avoid multiple connections or log files.

- **Single Access Point:** Applications often require configuration. For instance, configuring database connection parameters. You want a single access point for the configuration parameters to prevent multiple instances and conflicting configurations.

### Solution

The **Singleton pattern** is a creational design pattern that ensures a class has only one instance while providing a global access point to this instance. The following steps are involved in implementing the Singleton pattern:

1. **Constructor Hiding:** The constructor of the singleton class should be private or protected to prevent external instantiation.

2. **Global Access Point:** The singleton class should provide a global access point to retrieve its instance. This access point should be static and return the same instance every time it's called.

### Simple Singleton

```java
public class Database {
    private static Database instance = new Database();

    private Database() {
    }

    public static Database getInstance() {
        return instance;
    }
}
```

To implement the `getInstance()` method, a static variable is used for lazy initialization.

### Thread-Safe Singleton

To make the simple singleton thread-safe, make the `getInstance()` method synchronized.

```java
public class Database {
    private static Database instance = null;

    private Database() {
    }

    public static synchronized Database getInstance() {
        if (instance == null) {
            instance = new Database();
        }
        return instance;
    }
}
```

### Double-Checked Locking

To improve efficiency, use double-checked locking.

```java
public class Database {
    private static Database instance = null;

    private Database() {
    }

    public static Database getInstance() {
        if (instance == null) {
            synchronized (Database.class) {
                if (instance == null) {
                    instance = new Database();
                }
            }
        }
        return instance;
    }
}
```

### Summary

- The singleton pattern ensures a class has only one instance with a global access point.
- Use cases: Shared resources like database connections, logging mechanisms, and objects that should be instantiated only once, such as configuration objects.
- Hide the constructor of the singleton class by making it private.
- Add a static method to return the instance, creating it if it doesn't exist.
- For thread safety, make the `getInstance()` method synchronized or use double-checked locking.

---

## Builder Design Pattern

### Problems

- **Complex Object Creation:**
    - Multiple ways to create an object using constructors can lead to the telescoping constructor anti-pattern. This arises when there is a need to create an object with many parameters, making constructors unmanageable.

- **Validation and Failing Object Creation:**
    - When parameters need validation before object creation, using the default constructor prevents failing object creation if parameters are invalid.

- **Immutability:**
    - Mutable objects, whose state can change after creation, can lead to bugs. However, using the default constructor makes it challenging to create immutable objects.

### Constructor with a Hash Map

```java
public class Database {
    private String host;
    private int port;
    private String username;
    private String password;

    public Database(Map<String, String> config) {
        // Parameter validation and object creation logic
    }
}
```

Some problems:
- Type safety
- Difficulty identifying parameters

### Inner Class

```java
public class Database {
    private String host;
    private int port;
    private String username;
    private String password;

    public Database(DatabaseParameters parameter) {
        // Parameter validation and object creation logic
    }
}

class DatabaseParameters {
    // Fields for parameters
}
```

Issues:
- Cumbersome to use
- Changes in parameter names require modification in Database class

### Builder Pattern

```java
public class Database {
    private String host;
    private int port;
    private String username;
    private String password;

    private Database() {
    }

    public static class DatabaseBuilder {
        private String host;
        private int port;
        private String username;
        private String password;

        public Database build() {
            // Object creation logic
        }
    }
}
```

Usage:

```java
Database database = new Database.DatabaseBuilder()
    .host("localhost")
    .port(3306)
    .username("root")
    .password("password")
    .build();
```

### Summary

- The builder pattern constructs complex objects step by step.
- Use cases:
    - Complex object creation
    - Validation and failing object creation
    - Immutability
- Steps:
    1. Add a static inner class (builder class).
    2. Add a private constructor to the target class.
    3. Implement the `build()` method in the builder class.
    4. Add a method for each parameter in the builder class.
 
---

# Prototype Design Pattern

### Overview

The **Prototype pattern** allows hiding the complexity of creating new instances from the client. It involves copying an existing object instead of creating a new instance from scratch, particularly useful for objects with costly operations during creation. The existing object acts as a prototype, and the new object may change specific properties if needed, saving resources and time.

### Example

Suppose we need to test a new User API by creating a new user. Instead of repeatedly calling an API for random values, we can clone an existing user and modify necessary fields.

```java
User user = new User("John", "Doe", "john@doe.in", "1234567890");
User user2 = user.clone();
user2.setId(2);
```

### Implementation

#### Clonable Interface

```java
public abstract class User {
    public abstract User clone();
}
```

#### Prototype Registry

Extend the prototype pattern with a registry of pre-defined prototypes, allowing the client code to request a clone from the registry instead of creating a new object from scratch.

```java
public interface UserRegistry {
    User getPrototype(UserRole role);
    void addPrototype(UserRole role, User user);
}

class UserRegistryImpl implements UserRegistry {
    private Map<UserRole, User> registry = new HashMap<>();

    @Override
    public User getPrototype(UserRole role) {
        return registry.get(role).clone();
    }

    @Override
    public void addPrototype(UserRole role, User user) {
        registry.put(role, user);
    }
}
```

**Usage**

```java
UserRegistry registry = new UserRegistryImpl();
registry.addPrototype(UserRole.STUDENT, new Student("John", "Doe", "john@doe.in", "1234567890", UserRole.STUDENT, "CS"));

User user = registry.getPrototype(UserRole.STUDENT);
user.setId(1);
```

### Recap

- The prototype pattern creates objects similar to each other, avoiding the costly process of creating objects from scratch.
- It helps in reducing complexity for client code by allowing it to clone existing objects and modify them as needed.
- Implementation steps:
    1. **Clonable Interface:** Create a common interface for cloneable objects.
    2. **Object Class:** Implement a concrete class that overrides the `clone()` method.
    3. **Registry:** Create a registry of pre-defined prototypes.
    4. **Prototype:** Create a prototype object and store it in the registry.
    5. **Clone:** Request a clone from the registry and modify it as needed.
 

---

## Factory Design Pattern

### Overview

The **Factory Method pattern** is a creational design pattern that addresses the problem of creating objects without specifying their exact class. It utilizes factory methods, either defined in an interface and implemented by child classes or implemented in a base class and optionally overridden by derived classes.

### Motivation

In scenarios where the exact class of an object is unknown, such as when using an external library or creating objects with changing classes, the **prototype pattern** is employed to create a clone of an existing object. However, to enhance code independence from the object's class, the **factory pattern** is introduced.

### Simple Factory

The **Simple Factory pattern** provides a static method in a factory class for creating objects, reducing the need to specify the exact class. This method is implemented using a switch statement based on the input.

```java
class UserFactory {
    public static User createUser(UserRole role) {
        // Object creation logic based on role
    }
}

User user = UserFactory.createUser(UserRole.STUDENT);
```

Complete steps for implementing the simple factory pattern:
1. Factory class: Create a class with a static method for object creation.
2. Conditional: Use a conditional statement to create the object based on the input.
3. Request: Request an object from the factory class without knowing the exact class.

### Factory Method

The **Factory Method pattern** overcomes drawbacks of the simple factory by shifting the responsibility of creating objects to child classes. The base class contains a factory method that child classes can override to create objects of their type.

```java
abstract class UserFactory {
    public abstract User createUser(String firstName, String lastName);
}

class StudentFactory extends UserFactory {
    @Override
    public User createUser(String firstName, String lastName) {
        // Object creation logic for Student
    }
}
```

Usage:

```java
UserFactory factory = new StudentFactory();
User user = factory.createUser("John", "Doe");
```

Complete steps for implementing the factory method pattern:
1. Base factory interface: Create a class with a method for object creation.
2. Child factory class: Create a child class extending the base class and override the factory method.
3. Request: Request an object from the factory class without knowing the exact class.

### Abstract Factory

The **Abstract Factory pattern** provides an interface for creating families of related or dependent objects without specifying their concrete classes. It uses a family of factories, each responsible for creating a different product.

```java
abstract class ClassroomFactory {
    public abstract Student createStudent(String firstName, String lastName);
    public abstract Teacher createTeacher(String firstName, String lastName);
}

class BiologyClassroomFactory extends ClassroomFactory {
    @Override
    public Student createStudent(String firstName, String lastName) {
        // Object creation logic for Biology Student
    }

    @Override
    public Teacher createTeacher(String firstName, String lastName) {
        // Object creation logic for Biology Teacher
    }
}
```

Usage:

```java
ClassroomFactory factory = new BiologyClassroomFactory();
Student student = factory.createStudent("John", "Doe");
Teacher teacher = factory.createTeacher("John", "Doe");
```

The abstract factory becomes a factory of factories, allowing easy exchange of product families, promoting consistency among products, and isolating concrete classes from client code.

### Recap

- The factory pattern enables object creation without specifying the exact class.
- Simple Factory: The factory class contains a static method for object creation. It is easy to implement but not extensible and reusable.
- Factory Method: The responsibility of object creation is shifted to child classes using a factory method. It is extensible, reusable, and follows design principles.
- Abstract Factory: Provides an interface for creating families of related objects, promoting consistency, and isolating concrete classes from client code.
