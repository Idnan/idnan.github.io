---
layout: post
title: Understanding the SOLID Principles: A Beginner's Guide
comments: false
---

SOLID is an acronym that stands for five design principles of object-oriented programming (OOP). These principles help in designing software systems that are more maintainable, flexible, and scalable. The SOLID principles are:

## Single Responsibility Principle:
This principle states that a class should have only one reason to change. This means that a class should have only one responsibility or job, and should not have multiple responsibilities that can be separated out into separate classes.

In other words, a class should do one thing and do it well. This helps to keep code organized, maintainable, and easy to modify. When a class has too many responsibilities, it can become difficult to maintain and modify over time.

### Example:
Let's say you have a class called `Car`. The responsibility of this class is to represent a car and provide some basic functionality related to cars, like `startEngine()`, `stopEngine()`, and `drive()`.

However, let's say you also want to log some data about the car, like how many times the engine was started, or how far the car has driven.
If you put the logging functionality in the same `Car` class, you violate the Single Responsibility Principle. This is because the `Car` class now has two responsibilities: representing a car and logging data.

A better approach would be to create a separate `Logger` class that is responsible for logging data. Then, you can use an instance of the `Logger` class in the `Car` class to log data as needed.

This way, the `Car` class only has one responsibility, which is to represent a car and provide basic car functionality. The `Logger` class is responsible for logging data, and can be reused in other classes that need logging functionality.

```php
class Car {
    private $engineStartCount;
    private $distanceDriven;
    private $logger;

    public function __construct(Logger $logger) {
        $this->logger = $logger;
    }

    public function startEngine() {
        // Code to start the engine
        $this->engineStartCount++;
        $this->logger->log("Engine started. Total starts: " . $this->engineStartCount);
    }

    public function stopEngine() {
        // Code to stop the engine
        $this->logger->log("Engine stopped.");
    }

    public function drive($distance) {
        // Code to drive the car
        $this->distanceDriven += $distance;
        $this->logger->log("Distance driven: " . $this->distanceDriven);
    }
}

class Logger {
    public function log($message) {
        // Code to log the message to a file or database
        echo $message . PHP_EOL;
    }
}
```

## Open/Closed Principle:
This principle states that software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification. This means that a software entity should be easily extendable without modifying its source code.

In simpler terms, the Open/Closed Principle suggests that you should write code in a way that allows you to add new functionality without changing existing code. Instead of modifying existing code, you should create new code that extends the existing code.

This principle helps to make your code more maintainable and robust, as it reduces the risk of introducing bugs or unintended consequences when modifying existing code.

### Example:
Let's say we have a class `DiscountCalculator` that calculates the discount on an order. The current implementation of the `DiscountCalculator` class applies a fixed discount of 10% to the total order amount. Now let's say we want to introduce a new type of discount that applies a fixed amount discount of $5 on each item in the order.

To implement this new functionality, we could modify the existing `DiscountCalculator` class to support both types of discounts. However, this would violate the Open/Closed Principle, as we would be modifying an existing class instead of extending it.

Instead, we can create a new class `ItemDiscountCalculator` that extends the `DiscountCalculator` class and implements the new type of discount. This way, the `DiscountCalculator` class remains unchanged, and we can use either the original discount or the new item discount as needed.

```php
class DiscountCalculator {
  public function calculateDiscount($total) {
    return $total * 0.1; // 10% discount
  }
}

class ItemDiscountCalculator extends DiscountCalculator {
  public function calculateDiscount($total) {
    $itemDiscount = 0;
    foreach ($this->order->items as $item) {
      $itemDiscount += 5; // $5 discount per item
    }
    return $itemDiscount;
  }
}
```

## Liskov Substitution Principle:
The Liskov Substitution Principle (LSP) states that objects of a superclass should be able to be replaced with objects of its subclass without causing any problems or unexpected behavior in the program.

### Example
Let's say you have a class called `Shape` with a method called "getArea()". This method returns the area of the shape. Now, you create two subclasses of `Shape` called `Rectangle` and `Triangle`.

According to the Liskov Substitution Principle, you should be able to replace an instance of the `Shape` class with an instance of either the `Rectangle` or `Triangle` classes without causing any problems. This means that if you call the `getArea()` method on an instance of `Rectangle` or `Triangle`, it should return the correct area for that shape.

To ensure that the Liskov Substitution Principle is followed, you need to make sure that the subclass does not change the behavior of the parent class. In other words, the subclass should only add new functionality or behaviors, not change the existing ones.

For example, in our `Shape` class, the `getArea()` method should be implemented in such a way that it works correctly for any shape. If we create a subclass of `Shape` called `Circle`, we should be able to replace an instance of `Shape` with an instance of `Circle` without any problems, as long as the `getArea()` method of the `Circle` class returns the area of the circle correctly.

```php
class Shape {
  public function getArea() {
    // implementation for calculating the area of the shape
  }
}

class Rectangle extends Shape {
  public function getArea() {
    // implementation for calculating the area of the rectangle
  }
}

class Triangle extends Shape {
  public function getArea() {
    // implementation for calculating the area of the triangle
  }
}

class Circle extends Shape {
  public function getArea() {
    // implementation for calculating the area of the circle
  }
}
```

## Interface Segregation Principle:
This principle states that a client should not be forced to depend on methods it does not use. In other words, you should create interfaces that are specific to the needs of their clients.

### Example
Suppose we have an interface `Animal` that defines a method `run()` and `swim()`. However, not all animals can swim. Implementing the `swim()` method in all classes that implement `Animal` would be unnecessary and might even result in incorrect behavior.

To follow the Interface Segregation Principle, we should split the `Animal` interface into smaller interfaces based on the specific functionality they provide. In this case, we could create two interfaces: `Runnable` and `Swimmable`.

Here's an example code:
```php
interface Runnable {
    public function run();
}

interface Swimmable {
    public function swim();
}

class Dog implements Runnable {
    public function run() {
        echo "Dog is running.";
    }
}

class Fish implements Swimmable {
    public function swim() {
        echo "Fish is swimming.";
    }
}

class Cat implements Runnable {
    public function run() {
        echo "Cat is running.";
    }
}
```
In this example, we have split the `Animal` interface into two smaller interfaces: `Runnable` and `Swimmable`. The `Dog` and `Cat` classes implement the `Runnable` interface, which only defines the `run()` method. The `Fish` class implements the `Swimmable` interface, which only defines the `swim()` method. This way, each class only implements the methods that are relevant to it, and we avoid having unnecessary or irrelevant methods in a class.

## Dependency Inversion Principle:
This principle states that high-level modules should not depend on low-level modules. Instead, both should depend on abstractions. 

### Example
Suppose you have a class called `UserController` that is responsible for handling user-related tasks in your application. It has a method called `createUser` that creates a new user object and saves it to the database.
```php
class UserController {
  private $db;

  public function __construct(Database $db) {
    $this->db = $db;
  }

  public function createUser($name, $email, $password) {
    $user = new User($name, $email, $password);
    $this->db->save($user);
  }
}
```
In this example, the `UserController` class has a dependency on the `Database` class, which is tightly coupled to it. If you want to change the database implementation or use a different database system, you would need to modify the `UserController` class, which violates the Dependency Inversion Principle.

To solve this problem, you can use dependency injection to invert the dependency. Instead of creating an instance of the `Database` class inside the `UserController` class, you can inject an instance of the `Database` class into the constructor.

```php
interface DatabaseInterface {
  public function save($data);
}

class Database implements DatabaseInterface {
  public function save($data) {
    // implementation
  }
}

class UserController {
  private $db;

  public function __construct(DatabaseInterface $db) {
    $this->db = $db;
  }

  public function createUser($name, $email, $password) {
    $user = new User($name, $email, $password);
    $this->db->save($user);
  }
}
```

In this example, we've created an interface called `DatabaseInterface` that defines the `save` method, which the `Database` class implements. We've also updated the `UserController` class to accept an instance of `DatabaseInterface` instead of `Database`. This way, we can inject any class that implements the `DatabaseInterface` into the `UserController` class, without modifying its code.

By using dependency injection and programming to interfaces, we have decoupled the `UserController` class from the `Database` class, making it more flexible and easier to maintain.
