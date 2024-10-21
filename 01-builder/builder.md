# Builder Pattern

The Builder Pattern is a creational design pattern used to construct complex objects step by step. Unlike other patterns that create objects in a single step (such as the Factory Pattern), the Builder Pattern focuses on creating a complex object by specifying its type and content, followed by assembling it in a controlled manner.

The Builder Pattern is particularly useful when:
- An object requires numerous parameters for its construction.
- Some of the parameters might be optional.
- The construction of the object involves complex processes.

The pattern separates the construction of an object from its representation, allowing the same construction process to create different types or representations of the object.

### Structure of the Builder Pattern
1. **Builder Interface**: Specifies methods for constructing the parts of the complex object.
2. **Concrete Builder**: Implements the Builder interface and constructs the object step by step.
3. **Director**: An optional class responsible for constructing the object using the builder.
4. **Product**: The final complex object that is being built.

### Example Without Builder Pattern
Suppose we are creating a House struct that requires many parameters, including some optional ones. Without the Builder Pattern, we would typically pass all parameters directly to the constructor or use multiple setters:

```go
package main

import "fmt"

// House represents a complex object with multiple parameters
type House struct {
    windows int
    doors   int
    hasGarden bool
    hasGarage bool
    hasSwimmingPool bool
}

func NewHouse(windows int, doors int, hasGarden bool, hasGarage bool, hasSwimmingPool bool) *House {
    return &House{
        windows: windows,
        doors: doors,
        hasGarden: hasGarden,
        hasGarage: hasGarage,
        hasSwimmingPool: hasSwimmingPool,
    }
}

func main() {
    // Creating a house with multiple parameters
    house := NewHouse(4, 2, true, true, false)
    fmt.Printf("House: %+v\n", house)
}
```

### Problem:
- The constructor becomes difficult to maintain as more parameters are added.
- It’s easy to get lost in the order of parameters (leading to errors like passing incorrect values).
- Some parameters might be optional, and it's not clear which ones are mandatory.

### Refactoring Using Builder Pattern

Using the Builder Pattern, we can solve these issues by constructing the object step by step in a more readable and flexible way.

```go
package main

import "fmt"

// House is the complex object we're building
type House struct {
    windows        int
    doors          int
    hasGarden      bool
    hasGarage      bool
    hasSwimmingPool bool
}

// HouseBuilder is the interface for building a house
type HouseBuilder interface {
    SetWindows(windows int) HouseBuilder
    SetDoors(doors int) HouseBuilder
    SetGarden(hasGarden bool) HouseBuilder
    SetGarage(hasGarage bool) HouseBuilder
    SetSwimmingPool(hasSwimmingPool bool) HouseBuilder
    Build() House
}

// ConcreteHouseBuilder implements the HouseBuilder interface
type ConcreteHouseBuilder struct {
    house House
}

func NewHouseBuilder() *ConcreteHouseBuilder {
    return &ConcreteHouseBuilder{}
}

func (b *ConcreteHouseBuilder) SetWindows(windows int) HouseBuilder {
    b.house.windows = windows
    return b
}

func (b *ConcreteHouseBuilder) SetDoors(doors int) HouseBuilder {
    b.house.doors = doors
    return b
}

func (b *ConcreteHouseBuilder) SetGarden(hasGarden bool) HouseBuilder {
    b.house.hasGarden = hasGarden
    return b
}

func (b *ConcreteHouseBuilder) SetGarage(hasGarage bool) HouseBuilder {
    b.house.hasGarage = hasGarage
    return b
}

func (b *ConcreteHouseBuilder) SetSwimmingPool(hasSwimmingPool bool) HouseBuilder {
    b.house.hasSwimmingPool = hasSwimmingPool
    return b
}

func (b *ConcreteHouseBuilder) Build() House {
    return b.house
}

func main() {
    builder := NewHouseBuilder()

    // Step-by-step construction of the house
    house := builder.
        SetWindows(4).
        SetDoors(2).
        SetGarden(true).
        SetGarage(true).
        SetSwimmingPool(false).
        Build()

    fmt.Printf("House: %+v\n", house)
}
```
### Explanation:
1. House: This is the complex object we want to build.
2. HouseBuilder Interface: This defines the methods to set various properties of the House object (e.g., SetWindows, SetDoors, etc.).
3. ConcreteHouseBuilder: Implements the HouseBuilder interface. It provides methods that return the builder itself (HouseBuilder), allowing method chaining to progressively build the House object.
4. Build() Method: The Build method is the final step, returning the fully constructed House object.

### Benefits of the Builder Pattern:
- Readable Code: The construction of an object is done step-by-step, making the code more readable, especially when there are many parameters.
- Flexibility: You can choose which parts of the object to build, making it easy to handle optional parameters.
- Immutability: Once the Build method is called, the object is ready and can be used in an immutable way (i.e., no further changes are made).
- Separation of Concerns: The client doesn’t need to know how the object is constructed internally, only how to specify what it needs.

### Real-World Use Cases for Builder Pattern:
- Configuration objects: When you have configuration objects with many parameters, some of which are optional or have default values, the builder pattern provides an elegant solution for constructing them.
- UI libraries: Constructing complex UI elements where each part can be customized (e.g., buttons, dialog boxes).
- Complex document generation: In scenarios like PDF or HTML generation, the builder pattern can be used to progressively add sections or elements to the document.