[index](../readme.md)
# Factory Pattern

The Factory Pattern is a creational design pattern that provides an interface for creating objects in a superclass, but allows subclasses to alter the type of objects that will be created. Essentially, instead of creating objects directly using constructors, the Factory Pattern uses a factory (a separate method or class) to handle the creation logic. This helps in promoting loose coupling, making the system more flexible to changes.

There are two main types of factory patterns:

1. Simple Factory Pattern: It provides a single method for creating different types of objects based on some parameter or input.
2. Factory Method Pattern: It defines an interface for creating an object but allows subclasses to decide which class to instantiate.

### Use Cases for Factory Pattern:
- When the exact type of object to be created is determined at runtime.
- When you want to isolate the creation of objects from their actual use.
- When you want to introduce flexibility by allowing object creation to evolve over time without modifying existing code.

### Simple Factory Pattern Example in Go
In this example, imagine we want to create different types of Animal objects (Dog, Cat) based on some input:

```go
package main

import "fmt"

// Animal is the interface for all animals
type Animal interface {
    Speak() string
}

// Dog is a concrete implementation of Animal
type Dog struct{}

func (d Dog) Speak() string {
    return "Woof!"
}

// Cat is another concrete implementation of Animal
type Cat struct{}

func (c Cat) Speak() string {
    return "Meow!"
}

// SimpleAnimalFactory is a factory that creates animals based on input
type SimpleAnimalFactory struct{}

func (s SimpleAnimalFactory) CreateAnimal(animalType string) Animal {
    switch animalType {
    case "dog":
        return Dog{}
    case "cat":
        return Cat{}
    default:
        return nil
    }
}

func main() {
    factory := SimpleAnimalFactory{}

    // Create a dog
    dog := factory.CreateAnimal("dog")
    fmt.Println("Dog says:", dog.Speak())

    // Create a cat
    cat := factory.CreateAnimal("cat")
    fmt.Println("Cat says:", cat.Speak())
}
```
### Explanation:
1. Animal Interface: This defines a Speak method that all animals must implement.
2. Dog and Cat Structs: These are concrete implementations of the Animal interface.
3. SimpleAnimalFactory: This is our factory that has a method CreateAnimal. It takes in a string (animalType) and returns the corresponding Animal object.
4. main Function: In this function, we use the factory to create instances of Dog and Cat without directly using their constructors.

### Benefits of the Simple Factory Pattern:
- Encapsulation of Object Creation: The factory encapsulates the logic of creating objects, making the client code cleaner and easier to maintain.
- Single Responsibility: The factory has a single responsibility: creating objects.
- Flexibility: The factory can be modified to return different implementations without changing the client code.

### Factory Method Pattern Example in Go
In the Factory Method Pattern, the actual creation logic is deferred to subclasses, which means that we create an abstract factory and concrete factories that handle the creation.

Let’s extend the previous example, where we define an abstract factory for creating animals and let the concrete factories (DogFactory, CatFactory) handle the instantiation.

```go
package main

import "fmt"

// Animal is the interface for all animals
type Animal interface {
    Speak() string
}

// Dog is a concrete implementation of Animal
type Dog struct{}

func (d Dog) Speak() string {
    return "Woof!"
}

// Cat is another concrete implementation of Animal
type Cat struct{}

func (c Cat) Speak() string {
    return "Meow!"
}

// AnimalFactory is an interface that defines a method to create animals
type AnimalFactory interface {
    CreateAnimal() Animal
}

// DogFactory is a factory that creates Dog objects
type DogFactory struct{}

func (d DogFactory) CreateAnimal() Animal {
    return Dog{}
}

// CatFactory is a factory that creates Cat objects
type CatFactory struct{}

func (c CatFactory) CreateAnimal() Animal {
    return Cat{}
}

func main() {
    // Create a dog factory and produce a dog
    var dogFactory AnimalFactory = DogFactory{}
    dog := dogFactory.CreateAnimal()
    fmt.Println("Dog says:", dog.Speak())

    // Create a cat factory and produce a cat
    var catFactory AnimalFactory = CatFactory{}
    cat := catFactory.CreateAnimal()
    fmt.Println("Cat says:", cat.Speak())
}
```
### Explanation:
1. Animal Interface: This is the same as before; it defines the common behavior (Speak) for animals.
2. Dog and Cat Structs: These are the concrete implementations of the Animal interface, like before.
3. AnimalFactory Interface: This interface declares the CreateAnimal method, which returns an Animal. It’s an abstract factory that will be implemented by concrete factories.
4. DogFactory and CatFactory: These are concrete factories that implement the AnimalFactory interface. Each factory is responsible for creating a specific type of animal (Dog or Cat).
5. main Function: The DogFactory and CatFactory are used to create Dog and Cat objects, respectively, without directly instantiating them in the client code.

### Benefits of the Factory Method Pattern:

- Flexibility: New types of products can be added easily by creating new factory classes.
- Code Reusability: The client code does not need to change when a new factory is added, promoting reusability and flexibility.
- Separation of Concerns: The client code is decoupled from the actual creation logic, following the Dependency Inversion Principle.

### Factory Pattern in Real Life:

- Loggers: Depending on the configuration, a factory could return a FileLogger, ConsoleLogger, or DatabaseLogger without the client knowing which one it is using.
- Database Connections: Depending on the environment (development, production), a factory could return a connection to an in-memory database, an SQL database, or a cloud-based database.
- UI Widgets: Factories are used to create UI elements like buttons, text fields, or dropdown menus depending on the platform or theme.

