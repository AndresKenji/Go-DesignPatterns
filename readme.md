## Table of content

1. [Solid Design Principles](#solid-design-principles)
2. [SRP](#s---single-responsibility-principle-srp)
3. [OCP](#o---openclosed-principle-ocp)
4. [LSP](#l---liskov-substitution-principle-lsp)
5. [ISP](#i---interface-segregation-principle-isp)
6. [DIP](#d---dependency-inversion-principle-dip)
7. [Builder](./01-builder/builder.md)
8. [Factory](./02-factory/readme.md)

# Solid Design Principles

The SOLID design principles are a set of five design guidelines introduced to promote maintainable, scalable, and robust software architecture. 
These principles help developers create systems that are easier to understand, extend, and modify over time, reducing the chances of introducing bugs when making changes. 
The SOLID principles were introduced by **Robert C. Martin** (often known as Uncle Bob) in his 2000 paper titled **Design Principles and Design Patterns**.


## S - Single Responsibility Principle (SRP):

The Single Responsibility Principle (SRP), states that a class or module should have only one reason to change. In simpler terms, each module, function, or class should have one specific responsibility or role. This makes the code more modular, easier to understand, and easier to maintain.

By limiting a class or module to a single responsibility, we reduce coupling, improve separation of concerns, and simplify testing.

### Applying SRP in Go
In Go, SRP can be applied to structs and functions. Each struct should have a clear, well-defined purpose, and each function or method should do one thing.

Let’s explore an example.

- Example Without SRP
Suppose we have a program that handles file operations and also sends reports. Initially, both tasks are handled by a single Report struct:

```go
package main

import (
    "fmt"
    "os"
)

// Report handles generating a report and saving it to a file
type Report struct {
    Title string
    Body  string
}

// SaveToFile saves the report to a file
func (r Report) SaveToFile(filename string) error {
    file, err := os.Create(filename)
    if err != nil {
        return err
    }
    defer file.Close()

    _, err = file.WriteString(r.Title + "\n" + r.Body)
    return err
}

// SendByEmail sends the report via email (simplified for the example)
func (r Report) SendByEmail(email string) {
    fmt.Printf("Sending report to %s...\n", email)
    // Simulating sending an email...
}

func main() {
    report := Report{Title: "Monthly Report", Body: "This is the report content"}

    err := report.SaveToFile("report.txt")
    if err != nil {
        fmt.Println("Error saving file:", err)
    }

    report.SendByEmail("example@example.com")
}
```
### Problem
Here, the Report struct is responsible for two different tasks:

- Saving the report to a file (SaveToFile)
- Sending the report via email (SendByEmail)

This violates SRP because if you need to change how reports are saved to a file or sent via email, you would have to modify the Report class for both operations. This creates unnecessary coupling between responsibilities.

### Refactoring Using SRP
To follow SRP, we should split these responsibilities. We can introduce separate structs or functions that handle saving to a file and sending via email.

```go
package main

import (
    "fmt"
    "os"
)

// Report holds the content of the report
type Report struct {
    Title string
    Body  string
}

// FileSaver is responsible for saving reports to files
type FileSaver struct{}

// SaveToFile saves the report to a file
func (fs FileSaver) SaveToFile(r Report, filename string) error {
    file, err := os.Create(filename)
    if err != nil {
        return err
    }
    defer file.Close()

    _, err = file.WriteString(r.Title + "\n" + r.Body)
    return err
}

// EmailSender is responsible for sending reports via email
type EmailSender struct{}

// SendByEmail sends the report via email (simplified for the example)
func (es EmailSender) SendByEmail(r Report, email string) {
    fmt.Printf("Sending report to %s...\n", email)
    // Simulating sending an email...
}

func main() {
    report := Report{Title: "Monthly Report", Body: "This is the report content"}

    fileSaver := FileSaver{}
    err := fileSaver.SaveToFile(report, "report.txt")
    if err != nil {
        fmt.Println("Error saving file:", err)
    }

    emailSender := EmailSender{}
    emailSender.SendByEmail(report, "example@example.com")
}
```
### Explanation of the Refactor:
1. **Single Responsibility:**

- The Report struct now only holds data related to the report.
- The FileSaver struct is responsible for saving reports to files.
- The EmailSender struct is responsible for sending reports via email.
2. **Separation of Concerns:** Each struct has a single, well-defined purpose. If we need to change how we save files or send emails, we only need to modify the corresponding struct, not the entire system.

3. **Reusability:** Now that responsibilities are separated, we can reuse the FileSaver or EmailSender independently in other parts of the application.

4 **Testing:** Testing individual responsibilities is easier. We can test the file-saving and email-sending logic separately without involving the other functionality.

### Benefits of SRP:
- Better Maintainability: When a class has one responsibility, it is easier to understand, modify, and debug.
- Increased Modularity: When responsibilities are separated, each component can evolve independently.
- Improved Testability: Each responsibility can be tested in isolation, making testing simpler and more effective.
- Scalability: As the system grows, you can extend functionality without bloating classes or modules with multiple responsibilities.

By applying SRP, you end up with cleaner, more modular code that is easier to work with and extend over time.




## **O** - Open/Closed Principle **(OCP)**:
The Open/Closed Principle (OCP), part of the SOLID design principles, states that software entities like classes, modules, or functions should be open for extension but closed for modification. In simpler terms, you should be able to add new functionality to a system without changing the existing code, thereby avoiding unintended side effects.
This principle promotes extensibility by ensuring that new features can be added through well-defined extensions (like new classes or functions) without needing to modify existing, tested, and stable code.

### How to apply OCP in Go: 
In Go, we often use **interfaces** to apply OCP. By defining behavior through interfaces, we can extend functionality by adding new types that implement the interface, instead of altering the existing code. Let’s walk through an example to clarify this.

- Example: Let’s say we are building a system that calculates the area of different shapes. Initially, we have a simple Rectangle struct and a function to calculate its area:

```go
package main

import "fmt"

// Rectangle is a shape that has width and height
type Rectangle struct {
    Width  float64
    Height float64
}

// Area calculates the area of the rectangle
func Area(rect Rectangle) float64 {
    return rect.Width * rect.Height
}

func main() {
    rect := Rectangle{Width: 5, Height: 10}
    fmt.Printf("Area of rectangle: %f\n", Area(rect))
}
```

This works fine, but what happens if we want to add support for calculating the area of a circle or other shapes? According to OCP, we shouldn’t modify the existing Area function to add new shape-specific logic. Instead, we should extend the system by introducing abstractions, allowing us to calculate the area for any shape.

- Refactoring with OCP in Go
We can refactor the design using an interface to represent the concept of a "shape". By doing so, we can easily extend the program to support new shapes without modifying the existing code:

```go
package main

import (
    "fmt"
    "math"
)

// Shape is an interface that any shape must implement
type Shape interface {
    Area() float64
}

// Rectangle implements the Shape interface
type Rectangle struct {
    Width  float64
    Height float64
}

// Area calculates the area of the rectangle
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Circle implements the Shape interface
type Circle struct {
    Radius float64
}

// Area calculates the area of the circle
func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

// CalculateArea takes any Shape and prints its area
func CalculateArea(s Shape) {
    fmt.Printf("Area: %f\n", s.Area())
}

func main() {
    rect := Rectangle{Width: 5, Height: 10}
    circle := Circle{Radius: 7}

    CalculateArea(rect)   // Output: Area of rectangle: 50.000000
    CalculateArea(circle) // Output: Area of circle: 153.938040
}
```

- Explanation of the Refactor:
    - Open for Extension: We can add new shapes like Circle or any other shape in the future, just by implementing the Shape interface. The existing system doesn’t need to be modified. For instance, we added a Circle type that implements the Area() method. Similarly, we could add other shapes like Triangle, Square, etc.

    - Closed for Modification: The CalculateArea function works with any Shape and does not need to be modified when new shapes are introduced. The same applies to the Rectangle struct — its original implementation remains untouched.

- Benefits of Applying OCP
    - Extensibility: New functionality (like support for new shapes) can be added without altering the existing code.
    - Maintainability: Since we don’t need to modify tested and existing code, the risk of introducing bugs is reduced.
    - Flexibility: The code is more flexible because it relies on abstractions (Shape interface) rather than concrete types.

By following the Open/Closed Principle, you ensure that your codebase can grow without turning into an unmanageable mess of conditional statements or tightly coupled logic. Instead, you create a flexible system that's easy to extend.

## **L** - Liskov Substitution Principle **(LSP)**:
Objects of a superclass should be replaceable with objects of a subclass without affecting the correctness of the program. Essentially, derived classes must be substitutable for their base classes without altering the desired behavior of the program.

## **I** - Interface Segregation Principle **(ISP)**:
The Interface Segregation Principle (ISP) is the fourth principle in the SOLID principles. It states that no client should be forced to depend on methods it does not use. In other words, rather than having large, general-purpose interfaces, it's better to create smaller, more specific interfaces that are focused on a particular set of actions.

ISP encourages designing interfaces that are tailored to the specific needs of clients, avoiding "fat" or overly broad interfaces that may force clients to implement methods they don’t need. This leads to better decoupling and increased flexibility.

### Example Without ISP
Suppose you have a program where various machines (like printers and scanners) can perform operations such as printing, scanning, and faxing. You create an interface Machine that includes all possible operations:

```go
package main

import "fmt"

// Machine interface that combines multiple operations
type Machine interface {
    Print(document string)
    Scan(document string)
    Fax(document string)
}

// Printer implements all methods (print, scan, fax)
type Printer struct{}

func (p Printer) Print(document string) {
    fmt.Println("Printing:", document)
}

func (p Printer) Scan(document string) {
    fmt.Println("Scanning:", document)
}

func (p Printer) Fax(document string) {
    fmt.Println("Faxing:", document)
}

func main() {
    doc := "Document"
    printer := Printer{}
    
    printer.Print(doc)  // Printer can print
    printer.Scan(doc)   // Printer can scan
    printer.Fax(doc)    // Printer can fax
}
```
### Problem:
Not all machines have all functionalities. For example, you may have a SimplePrinter that can only print but not scan or fax. However, due to the large Machine interface, the SimplePrinter must implement methods it doesn’t need:

```go
package main

import "fmt"

// SimplePrinter can only print but must implement all methods
type SimplePrinter struct{}

func (sp SimplePrinter) Print(document string) {
    fmt.Println("Printing:", document)
}

func (sp SimplePrinter) Scan(document string) {
    // Does nothing, since SimplePrinter can't scan
}

func (sp SimplePrinter) Fax(document string) {
    // Does nothing, since SimplePrinter can't fax
}

func main() {
    doc := "Document"
    simplePrinter := SimplePrinter{}
    
    simplePrinter.Print(doc)  // This works
    // But Scan and Fax methods are irrelevant for SimplePrinter
}
```
Problem Recap:
The SimplePrinter is forced to implement Scan and Fax methods, even though it doesn’t use them.
This creates "fat" interfaces that violate ISP, as clients (like SimplePrinter) are forced to depend on methods they don’t need.
Refactoring Using ISP
To fix this and follow ISP, we can break down the Machine interface into smaller, more specific interfaces, each representing a single functionality. This way, a class or struct will only implement the interfaces it needs.

```go
package main

import "fmt"

// Printer interface for printing functionality
type Printer interface {
    Print(document string)
}

// Scanner interface for scanning functionality
type Scanner interface {
    Scan(document string)
}

// Fax interface for faxing functionality
type Fax interface {
    Fax(document string)
}

// MultiFunctionPrinter implements all interfaces
type MultiFunctionPrinter struct{}

func (mfp MultiFunctionPrinter) Print(document string) {
    fmt.Println("Printing:", document)
}

func (mfp MultiFunctionPrinter) Scan(document string) {
    fmt.Println("Scanning:", document)
}

func (mfp MultiFunctionPrinter) Fax(document string) {
    fmt.Println("Faxing:", document)
}

// SimplePrinter only implements the Printer interface
type SimplePrinter struct{}

func (sp SimplePrinter) Print(document string) {
    fmt.Println("Printing:", document)
}

func main() {
    doc := "Document"

    // MultiFunctionPrinter can print, scan, and fax
    mfp := MultiFunctionPrinter{}
    mfp.Print(doc)
    mfp.Scan(doc)
    mfp.Fax(doc)

    // SimplePrinter only prints
    sp := SimplePrinter{}
    sp.Print(doc)

    // Now SimplePrinter is not forced to implement Scan or Fax
}
```
### Explanation of the Refactor:
Separate Interfaces: Instead of having a large Machine interface, we break it down into smaller interfaces:
- Printer: Only defines the Print method.
- Scanner: Only defines the Scan method.
- Fax: Only defines the Fax method.
- MultiFunctionPrinter: This type implements all three interfaces (Printer, Scanner, and Fax), as it has the capability to perform all these operations.
- SimplePrinter: This type only implements the Printer interface, as it can only print. It is no longer forced to implement the Scan and Fax methods, thus adhering to ISP.

### Benefits of ISP:
- No Unnecessary Dependencies: By splitting large interfaces into smaller ones, clients (like SimplePrinter) are not forced to depend on methods they don’t need.
- Decoupling: Different responsibilities are decoupled into separate interfaces, making the system more modular and flexible.
- Better Maintenance: Smaller, more focused interfaces are easier to maintain and extend. If a new operation is added, only the relevant interface needs to be updated.
- Easier Testing: Testing becomes simpler, as each interface can be mocked independently, and you don’t need to mock unused methods.


## **D** - Dependency Inversion Principle **(DIP)**:
The Dependency Inversion Principle (DIP) is the fifth and final principle in the SOLID design principles. It states that high-level modules should not depend on low-level modules; both should depend on abstractions. Additionally, abstractions should not depend on details; details should depend on abstractions.

The idea behind DIP is to reduce tight coupling between the high-level logic (business logic) and low-level details (such as how inputs are obtained or outputs are generated). Instead of directly depending on concrete implementations, modules should depend on interfaces or abstract classes. This way, the details of how the system works can be swapped or modified without changing the high-level logic.

### Example Without DIP
Imagine we have a Report class that depends directly on a FileLogger to log its information. This creates a direct dependency on a concrete implementation:

```go
package main

import "fmt"

// FileLogger is responsible for logging to a file
type FileLogger struct{}

func (fl FileLogger) Log(message string) {
    fmt.Println("Logging to file:", message)
}

// Report depends on FileLogger directly
type Report struct {
    logger FileLogger
}

func (r Report) Generate() {
    // Report generation logic
    fmt.Println("Generating report...")

    // Log the report generation
    r.logger.Log("Report generated")
}

func main() {
    logger := FileLogger{}
    report := Report{logger: logger}
    report.Generate()
}
```
### Problem:
- The Report struct depends directly on the FileLogger. If we wanted to change the logging mechanism (e.g., to log to a database or a network service), we would need to modify the Report class.
- This violates DIP because the high-level module (Report) is dependent on a low-level module (FileLogger), and changing the logging implementation would require changes in the Report module.

### Refactoring Using DIP

To adhere to DIP, we introduce an abstraction (interface) for logging, which **Report** will depend on, rather than directly on **FileLogger**. This way, we can easily swap different logging implementations without affecting the **Report** logic.

```go
package main

import "fmt"

// Logger is an interface that both high-level and low-level modules depend on
type Logger interface {
    Log(message string)
}

// FileLogger is a low-level module that implements Logger
type FileLogger struct{}

func (fl FileLogger) Log(message string) {
    fmt.Println("Logging to file:", message)
}

// ConsoleLogger is another low-level module that implements Logger
type ConsoleLogger struct{}

func (cl ConsoleLogger) Log(message string) {
    fmt.Println("Logging to console:", message)
}

// Report depends on the Logger interface, not on a concrete implementation
type Report struct {
    logger Logger
}

func (r Report) Generate() {
    fmt.Println("Generating report...")

    // Log the report generation
    r.logger.Log("Report generated")
}

func main() {
    // File logging
    fileLogger := FileLogger{}
    report1 := Report{logger: fileLogger}
    report1.Generate()

    // Console logging
    consoleLogger := ConsoleLogger{}
    report2 := Report{logger: consoleLogger}
    report2.Generate()
}
```
### Explanation of the Refactor:
1. Logger Interface: We introduce a Logger interface that has a Log method. Both the FileLogger and ConsoleLogger structs implement this interface.

2. Dependency on Abstraction: The Report struct now depends on the Logger interface, not on any specific logger implementation. This makes the Report class flexible, as it can work with any logger that implements the Logger interface.

3. Swappable Implementations: We can easily swap out the FileLogger for a ConsoleLogger, or even create new loggers (e.g., DatabaseLogger), without modifying the Report class. This adheres to DIP by ensuring that high-level modules (like Report) depend on abstractions, and low-level details (like FileLogger and ConsoleLogger) depend on those same abstractions.

### Benefits of DIP:
- Decoupling: High-level modules are decoupled from low-level modules. This means changes in low-level modules don’t affect high-level modules.
- Flexibility: We can swap out different implementations of the abstraction (interface) without affecting the client code.
- Testability: By depending on abstractions, it becomes easier to mock or stub out dependencies for testing. For example, you can create a MockLogger for testing the Report class without needing an actual file or console logger.
- Maintainability: The code becomes easier to maintain and extend, as new implementations can be added without modifying existing high-level logic.

