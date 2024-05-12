# Effective Java 3rd Notes

## Chapter 2

### Item 1 static factory

Assume that there’s a class called `Person`. You can get an object by `new Person()` or static factory method like `PersonFactory.getOnePerson()`.

Why do we choose this static factory method? Here are some advantages and disadvantages:

advantages:

1. Static factory methods can offer clear guidance. In Java, constructors can be overloaded, but they cannot have identical parameter lists. So this sentence means that we can change the order of a sequence of parameters to get an object of the same attributes. However, these constructors may confuse us because they are similar and we can’t know the differences directly by different orders of parameters.
2. `new Person()` creates a new Person object when it is called. But static factory method can avoid this circumstance when we don’t want to create an object every time we call them.
   - The target of `Flyweight pattern`: to reduce the occupation of memory and the cost of creating object so that to enhance the performance.

3. The combination of static factory method and factory method pattern can let users acquire any actual class of an abstract class.
4. Different input parameters determine different returned types that are mentioned in the 3rd advantage.
5. When you are writing the class of `PersonFactory`, you don't need to know what object you will get finally. Just like you may get a Male or Female object, however, in the static factory method `getOnePerson()`, the return type is simply `Person`. Of course, this advantage also have a strong relationship with 3rd advantage.
   - `Bridge pattern`:
     1. `Service provider framework` is a set of some functions. These functions can be called as `service Interface`.
     2. Provider use `provider registration API` to implement these abstract functions and can also add some new functions.
     3. Client, when using this framework, can acquire a richer functions than the original functions by `service access API`. 


disadvantages:

1. The factory class like `PersonFactory` cannot have sub-class, because its constructors are not public or protected(are private).
2. Static factory methods are hard to find, compared to constructors.

### Item 2 builder

### Item 3 singleton

### Item 4 

### Item 5 dependency injection

### Item 6 don’t over-create objects

### Item 7 

### Item 8 

### Item 9 try-with-resources

## Chapter 3

## Chapter 4

## Chapter 5

## Chapter 6

## Chapter 7

## Chapter 8

## Chapter 9

## Chapter 10

## Chapter 11

## Chapter 12