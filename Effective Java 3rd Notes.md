# Effective Java 3rd Notes

## Chapter 2

### Item 1 static factory

Assume that there’s a class called `Person`. You can get an object by `new Person()` or a static factory method like `PersonFactory.getOnePerson()`.

Why do we choose the latter way? Here are some advantages and disadvantages:

advantages:

1. Static factory methods can offer clear guidance. In Java, constructors can be overloaded, but they cannot have identical parameter list. So this sentence means that we can change the order of a sequence of parameters to get an object of the same attributes. However, these constructors may confuse us because they are similar and we can’t know the differences directly by different orders of parameters.
2. `new Person()` creates a new `Person` object when it is called. But static factory method can avoid this circumstance when we don’t want to create an object every time we call them.
   - The target of `Flyweight pattern`: to reduce the occupation of memory and the cost of creating objects so that to enhance the performance.

3. The combination of static factory method and factory method pattern can let users acquire any actual class of an abstract class.
4. Different input parameters determine different returned types that are mentioned in the 3rd advantage.
5. When you are writing the class of `PersonFactory`, you don't need to know what object you will get finally. In the static factory method `getOnePerson()`, the return type is simply `Person`. Of course, this advantage also have a strong relationship with 3rd advantage.
   - `Bridge pattern`:
     1. `Service provider framework` is a set of some functions. These functions can be called as `service interface`.
     2. Provider use `provider registration API` to implement these abstract functions and can also add some new functions.
     3. Client, when using this framework, can acquire a richer functions than the original functions by `service access API`. 


disadvantages:

1. The factory class like `PersonFactory` cannot have sub-class, because its constructors are not public or protected(are private).
2. Static factory methods are hard to find, compared to constructors.

### Item 2 builder

The disadvantages of `telescoping constructor pattern`:

1. Clients don't know the meanings of the parameters, if they don't review the API, especially when they encounter long sequences of identically typed parameters.

You can use `Java Bean pattern` to avoid using `telescoping constructor pattern` by invoking parameterless constructor and using setters. However, there are some disadvantages:

1. A simple construction may be split by a host of calls, which means that this certain object can’t be used unless we give all the necessary parameters(a status of inconsistency).
2. If this object is required immutable, in other words, we don’t want to change this object after creating it, `Java Bean pattern` can’t guarantee this expect because of setters.

So, we want a mode that can let user know the meaning of a parameter and that can offer an object by an integrated way. `Builder pattern` is a good solution. The example is shown below:

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Computer {
    String cpu;
    String cache;
    String mm;
    String em;
    String screen;
    String frame;
    String keyboard;
    String battery;
    String touchBoard;
    String fan;
    String gpu;

    public Computer(Builder builder) {
        this.cpu = builder.cpu;
        this.cache = builder.cache;
        this.mm = builder.mm;
        this.em = builder.mm;
        this.keyboard = builder.keyboard;
        this.touchBoard = builder.touchBoard;
        this.screen = builder.screen;
        this.frame = builder.frame;
        this.battery = builder.battery;
        this.fan = builder.fan;
        this.gpu = builder.gpu;
    }

    public static Builder builder() {
        return new Builder();
    }

    public static final class Builder {
        String cpu;
        String cache;
        String mm;
        String em;
        String screen;
        String frame;
        String keyboard;
        String battery;
        String touchBoard;
        String fan;
        String gpu;

        public Builder cpu(String cpu) {
            this.cpu = cpu;
            return this;
        }

        public Builder cache(String cache) {
            this.cache = cache;
            return this;
        }

        public Builder mm(String mm) {
            this.mm = mm;
            return this;
        }

        public Builder em(String em) {
            this.em = em;
            return this;
        }

        public Builder screen(String screen) {
            this.screen = screen;
            return this;
        }

        public Builder frame(String frame) {
            this.frame = frame;
            return this;
        }

        public Builder keyboard(String keyboard) {
            this.keyboard = keyboard;
            return this;
        }

        public Builder battery(String battery) {
            this.battery = battery;
            return this;
        }

        public Builder touchBoard(String touchBoard) {
            this.touchBoard = touchBoard;
            return this;
        }

        public Builder fan(String fan) {
            this.fan = fan;
            return this;
        }

        public Builder gpu(String gpu) {
            this.gpu = gpu;
            return this;
        }

        public Computer build() {
            return new Computer(this);
        }
    }
}
```

We can use `Computer c = Computer.builder().xxx()….build()` to get a Computer object. Thus, you can know that the implementation of  `@Builder`(a lombok annotation) is the same as this example.

`Builder pattern` in class hierarchies can implement a variety of sub-classes of an abstract class.

```
```





### Item 3 singleton

Singleton guarantees that the class will be instantiated exactly once.

Ways to create singleton:

1. eager initialization: Thread-safe because of static and final. However, the singleton object is instantiated  when the class is loaded, whether it is used or not.

   ```java
   public class Logger {
   	private static final Logger logger = new Logger();
   	
   	private Logger() {
   	}
   	
   	public static Logger getLogger() {
   		return logger;
   	}
   }
   ```
   
2. static block initialization: Thread-safe because of static block will execute only once when loading the class. However, the singleton object is instantiated  when the class is loaded, whether it is used or not.

   ```java
   public class Logger {
   	private static Logger logger;
   	
   	static {
   		try {
   			logger = new Logger();
   		} catch (Exception e) {
   			// handle exception
   		}
   	}
   	
   	private Logger() {
   	}
   	
   	public static Logger getLogger() {
   		return logger;
   	}
   }
   ```
   
3. lazy initialization: Thread-safe in single-thread but unsafe in multi-thread

   ```java
   public class Logger {
   	private static Logger logger;
   	
   	private Logger() {
   	}
   	
   	public static Logger getLogger() {
           if (logger == null) {
               // sleep 1s to cause thread unsafe problem
               try {
                   Thread.sleep(1000);
               } catch (InterruptedException e) {
                   Thread.currentThread().interrupt();
               }
               
               logger = new Logger();
           }
   		return logger;
   	}
   }　
   ```
   
4. thread-safe lazy initialization: 

   ```java
   public class Logger {
       // Volatile prevents instruction reordering in multi-thread and other threads can see this change instantly.
   	private static volatile Logger logger;
   	
   	private Logger() {
   	}
   	
       // In fact, synchronization is only required when the first few threads are competing to create an instance, but the synchronized annotation on a method can cause performance degradation even after the instance is created
   	public static synchronized Logger getLogger_1() {
           if (logger == null) {
               logger = new Logger();
           }
           return logger;
   	}
       
       // Use double check locks to improve efficiency
       public static Logger getLogger_2() {
           // 1st check is to avoid unnecerry synchonization
           if (logger == null) {
               synchronized (Logger.class) {
                   // There will be many threads that pass the 1st check, we need to ensure only one thread creates the unique object.
                   if (logger == null) {
                       logger = new Logger();，
                   }
               }
           }
           return logger;
       }
   }
   ```

5. bill pugh: Logger object is initiated when `getLogger()` is invoked. And it is thread-safe because the loading process of static inner class is thread-safe. Furthermore, it doesn’t need extra synchronization mechanism, which means it has a better performance.

   ```java
   public class Logger {
   	private Logger() {
   	}
   	
       // A static inner class is not loaded immediately when an external class loads, but is loaded and initialized the first time it is used (that is, to access its static members or methods)
       private static class LoggerHelper {
       	private static final Logger logger = new Logger();
       }
       
   	public static Logger getLogger() {
       	return LoggerHelper.logger;
   	}
   }
   ```

But you can break singleton by invoking the private constructor reflectively. How to deal with this problem? Use enum!

```java
public enum Logger {
	LOGGER;
    
    private Logger () {
    }
}
```

If you want to use singleton in a distributive system, one effective way is enum, the other way is implementing `Serializable` interface and declare all instance fields `transient` and provide a `readResolve` method.

```java
public class Logger implements Serializable {
	private static final long serialVersionUID = 1L;
    
    private Logger () {
    }
    
    private static class LoggerHelper {
        private static final Logger logger = new Logger();
    }
    
    public static Logger getLogger() {
        return LoggerHelper.logger;
	}
    
    private Object readResolve() {
        return getLogger();
    }
}
```



### Item 4 private constructor

You should make utility class’s constructor private to avoid instantiating this class. Make it abstract is useless for the reason that you can create a sub-class to instantiate the sub-class.

### Item 5 



### Item 6 don’t over-create objects

`String` variable is stored in a string constant pool. So if we declare a String object like:

```
String s1 = “apple”;
```

This “apple” string will be preserved in the constant pool. However, if we declare a string like:

```
String s2 = new String(“monster”);
```

The string “monster” might occupy two memory spaces. One is in pool mentioned before, the other is  in heap space, which is actually re-occupied.

`new` keyword will create a new object in heap space, we need to avoid over creating new object. For example, if we use regular expression to validate an email is valid or not, we can use Pattern to apply the regular expression string, like `\w+@[a-zA-z0-9]{1,}\.com`:

```java
public boolean isValid(String given String){
    return givenString.matches("\\w+@[a-zA-z0-9]{1,}\\.com");
}
```

This method is rather inefficient because if we use this sentence for many times, it may create  countless the same Pattern object because the source code of `Pattern`‘s `compile` will create a new object once it runs:

```java
public static Pattern compile(String regex) {
    return new Pattern(regex, 0);
}
```

The way to avoid creating new object is use static block:

```java
static final Pattern emailPattern;

static {
    emailPattern = Pattern.compile("\\w+@[a-zA-z0-9]{1,}\\.com");
}

public boolean isValid(String given String){
    return emailPattern.matcher(givenString).find();
}
```

You can treat an object as a pointer in C++. So, if you use boxed primitives instead of primitives, when you change the value, the primitives will change it at the original location, but boxed primitives will point t another memory space. 

### Item 7 

When you build your own collection, you need to notice that you should set null to an useless reference so that gc can collect is as a garbage to avoid memory leak.

Here are 4 reference types in Java:

1. Strong Reference: is created by new keyword. It will never be collected by gc if it exists.
2. Soft Reference: When memory space is insufficient seriously, gc will collect this kind of reference to meet the memory requirement.
3. Weak Reference: will be collected whether memory space is sufficient or not.
4. Phantom Reference: we cannot get object by a phantom reference. Phantom reference mainly tracks the status of an object which will be collected by gc.

If you want to build your own cache by `Map`, you can choose `WeakHashMap` because it    is based on weak reference. When a reference no longer points to a key, the entry of this key will be eliminated.

### Item 8 



### Item 9 try-with-resources

If we use try-catch-finally to operate a file, we need an input stream and an output stream. But this will make codes redundant because there will be a lot of try blocks. 

So try-with-resources guarantees a more readable codes than try-catch-finally, and offers more valuable exceptions generated.

## Chapter 3

### Item 10 override “equals”

If you want to override equals in a certain class, you should obey these rules:

1. reflective: `x.equals(x)` is true
2. symmetric: `x.equals(y)` equals `y.equals(x)`
3. transitive: `x.equals(y)` is true, `y.equals(z)` is true, `x.equals(z)` should be true
4. consistent: results of `x.equals(y)` should be the same



### Item 11 override “hashCode”

always override `hashCode` when you override `equals`

### Item 12 override “toString”

We can see what the object contains by `toString`

### Item 13 clone

Must be prudent when we need a copy of an object. Copy constructor and copy factory are two better ways.

### Item 14 comperable

If you want to compare two objects, consider implementing Comparator interface to this certain class. Remember, don’t use the subtraction of two values because it may cause integer overflow and IEEE 754 floating point arithmetic artifacts.

Here are ways to implement comparing logic:

1. Implement Comparable and override `compreTo()`. In `compreTo()`, use boxed class’s static compare methods instead of subtraction.

   ```java
   
   public class Person implements Comparable<Person> {
     String name;
     int age;
     int postalCode;
   
     ...
   
     @Override
     public int compareTo(Person p) {
       int result = String.CASE_INSENSITIVE_ORDER.compare(name, p.name);
       if(result==0) result = Integer.compare(age, p.age);
       if(result==0) result = Integer.compare(postalCode, p.postalCode);
       return result;
     }
   }
   ```

   

2. Use `new Comparator{…}` when invoking sort method. **However** it may burden performance of system.

   ```java
   animals.sort((o1, o2) -> {
       int result = String.CASE_INSENSITIVE_ORDER.compare(o1.name, o2.name);
       if (result == 0) result = Integer.compare(o1.age, o2.age);
       if (result == 0) result = String.CASE_INSENSITIVE_ORDER.compare(o1.zoo, o2.zoo);
       return result;
   });
   ```

   

3. Use Comparator based on Comparator construction method.

   ```java
   @Getter
   @Setter
   public class Person implements Comparable<Person> {
     private static final Comparator<Person> COMPARATOR =  Comparator.comparing(Person::getName).thenComparingInt(Person::getAge).thenComparingInt(Person::getPostalCode);
   
     String name;
     int age;
     int postalCode;
   	...
   
     @Override
     public int compareTo(Person p) {
       return COMPARATOR.compare(this, p);
     }
   }
   ```
   
   

## Chapter 4

### item 15 decouple

You can use these four keywords to decorate an attribute to change the visibility of it.

|                                      | public | protected | default | private |
| ------------------------------------ | ------ | --------- | ------- | ------- |
| in the same class                    | ✅      | ✅         | ✅       | ✅       |
| in the same package                  | ✅      | ✅         | ✅       |         |
| in different packages but extends    | ✅      | ✅         |         |         |
| in differnet package and not extends | ✅      |           |         |         |

But if a class has many private attributes and it implements `Serializable` interface, these private attributes may forfeit their data protection.

```java
public class DemoForfeit {
    static String path = "/.../item_15/";
    public static void main(String[] args) {
        Data d = Data.of("xiaoming_zh", 22, true, "xiaoming_zh@email.com", "zhangxiaoming's id");
        System.out.println(d);
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(path + "data.ser"))) {
            oos.writeObject(d);
            System.out.println("Serializing...");
        } catch (Exception e) {
            e.printStackTrace();
        }
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(path + "data.ser"))) {
            Data deData = (Data) ois.readObject();
            System.out.println("deserialized data: " + deData);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
@lombok.Data
class Data implements Serializable {
    public static final long serialVersionUID = 1L;
    private String name;
    private Integer age;
    private Boolean gender;
    private String email;
    private String id;
    private Data() {
    }
    public static Data of(String name, Integer age, Boolean gender, String email, String id) {
        Data d = new Data();
        d.name = name;
        d.age = age;
        d.gender = gender;
        d.email = email;
        d.id = id;
        return d;
    }
}
```

The result is as below. we can find that, after deserializing, we can get the id of a person, which is not what we want. We must keep id invisible.

```cmd
Data(name=xiaoming_zh, age=22, gender=true, email=xiaoming_zh@email.com, id=zhangxiaoming's id)
Serializing...
deserialized data: Data(name=xiaoming_zh, age=22, gender=true, email=xiaoming_zh@email.com, id=zhangxiaoming's id)
```

So, an effective way is add `transient` to attribute `id`: `private transient String id;`.

The result is as below:

```cmd
Data(name=xiaoming_zh, age=22, gender=true, email=xiaoming_zh@email.com, id=zhangxiaoming's id)
Serializing...
deserialized data: Data(name=xiaoming_zh, age=22, gender=true, email=xiaoming_zh@email.com, id=null)
```



Interface in Java is default `abstract`, so you **don't** need to declare an interface like `public abstract interface MyInterface`. 

Attributes in an interface is default `public static final`, so you can declare an attribute like `int number;` instead of `public static final int number;`.

Methods in an interface is default `public abstract`.



### item 16 In public classes, use accessor methods, not public fields



### item 17 Minimize mutability



### item 18 Favor composition over inheritance



### item 19



### item 20 Prefer interfaces to abstract classes



## Chapter 5



### item 28

Array versus generic list:

- Array is type-safe in runtime but type-unsafe in compile time.
- Generic list is type-safe in compile time but type-unsafe in runtime.

## Chapter 6

## Chapter 7

## Chapter 8

## Chapter 9

## Chapter 10

## Chapter 11

## Chapter 12