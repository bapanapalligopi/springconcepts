Pre-Requisites: Before diving deep into OOPs, this is to inform you that the complete learning will be based on the Java programming language because of its wide use-cases in the real world. For easier understanding, you can first refer to the following: Java Basics.

Object-Oriented Programming (OOP)
Object-Oriented Programming (OOP) is a programming paradigm (a style of writing code) based on the concept of objects which can contain data and code. The data is represented as fields (often called attributes or properties), and the code is represented as procedures (often called methods). Objects are instances of classes, which act as blueprints for creating objects.

It consists mainly of two things:

Class: A class is a blueprint or template that defines the properties (attributes) and behaviors (methods) common to all objects of its type.
Object: An object is an instance of a class, representing a specific entity with its own unique state (attribute values) and behavior.
Difference between Procedural programming and Object Oriented programming
There are two major programming paradigms which are: Procedural programming and object-oriented programming. Both of these differ from each other in various aspects:

Approach:
Procedural programming focuses on a step-by-step sequence of actions to perform tasks where the control of the program moves in a sequential manner.
OOP focuses on modeling real-world entities as objects, combining data (attributes) and behavior (methods) without a particular flow of control.
Data Handling:
Data is globally accessible in Procedural programming, which can lead to accidental modifications and harder-to-maintain programs.
In OOP, data is encapsulated within objects, and access is restricted through methods. This ensures better data security, as only authorized methods can modify the data.
Code Reusability:
Code reusability is limited in procedural programming. Functions can be reused, but there is no concept of inheritance or polymorphism to extend and customize functionality easily.
Code reusability is high due to features like inheritance (reusing code in child classes) and polymorphism (handling objects in a generic way for flexibility).
Scalability:
Procedural code is often harder to scale for complex projects because adding new functionality requires modifying multiple functions and potentially breaking existing code.
OOP scales better for larger systems. Adding new functionality typically involves creating new classes or modifying existing ones with minimal impact on unrelated parts.
Modularity:
In procedural programming, programs are broken into functions, but the separation of logic and data is less structured, making large programs harder to manage.
In OOP, programs are modularized into classes and objects, ensuring better organization, maintainability, and scalability for large and complex systems.
Real-World Modeling:
Procedural programming is less aligned with real-world systems, as it focuses on actions and lacks an intuitive way to represent real-world entities or relationships.
OOP closely mirrors real-world scenarios by representing entities as objects with attributes and behaviors, making it easier to design and understand complex systems.
Use-Cases of Object Oriented Programming
There are four major factors that make OOP significantly used in the real world. These are as follows:

Modularity: The process of breaking down a complex problem into smaller, manageable, and reusable components (such as classes), enhancing code organization and maintainability.
Example: a banking application with separate classes for Account, Customer, Transaction, etc.
Code Reusability: Refers to the ability to extend and reuse existing functionality, reducing the need to duplicate code and promoting maintainability.
Example: Vehicle class extended by Car and Bike.
Scalability: Refers to the ability to effortlessly add new features or functionality without modifying existing code, ensuring the system can grow and adapt without disruption.
Security: Using OOP, users can protect sensitive data by encapsulating it within objects and exposing only the necessary functionality through controlled access methods, ensuring data integrity and security.
Example: private balance in a Bank Account class.
Real-life Analogy of Object Oriented Programming
Let us consider the codebase of a Bank.

Classes: Represent different entities in the bank, such as Account, Customer, and Transaction.
Objects: Specific instances of those classes, like Raj’s Account, or John’s Transaction.
Attributes: Information associated with each entity, like a customer's name, balance, or account number.
Methods: Actions the bank entities can perform, such as deposit(), withdraw(), and transfer().
Why is OOP better for large-scale applications?
OOP is ideal for large-scale applications because it promotes modularity, where complex systems are broken down into manageable components (classes and objects). It enhances code reusability through inheritance and polymorphism, making it easier to extend functionality without altering existing code.
Additionally, OOP ensures scalability by allowing new features to be added with minimal disruption and provides security by encapsulating sensitive data within objects.

These features make OOP well-suited for building and maintaining large, complex systems.
