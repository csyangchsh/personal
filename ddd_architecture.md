# DDD Architecture

This is a copy of DDD sample application notes. The orignal post is here: http://dddsample.sourceforge.net/architecture.html.

## Architecture

![](img/layers.jpg)

### Interfaces

This layer holds everything that interacts with other systems, such as web services, RMI interfaces or web applications, and batch processing frontends. 

It handles interpretation, validation and translation of incoming data. 

It also handles serialization of outgoing data, such as HTML or XML across HTTP to web browsers or web service clients, or DTO classes and distributed facade interfaces for remote Java clients.

### Application

The application layer is responsible for driving the workflow of the application, matching the use cases at hand. These operatios are interface-independent and can be both synchronous or message-driven. This layer is well suited for **spanning transactions**, **high-level logging and security**.

The application layer is thin in terms of domain logic - it merely coordinates the domain layer objects to perform the actual work.

### Domain

The domain layer is the heart of the software, and this is where the interesting stuff happens. **There is one package per aggregate, and to each aggregate belongs entities, value objects, domain events, a repository interface and sometimes factories.**

The core of the business logic belongs in here. The structure and naming of aggregates, classes and methods in the domain layer should follow the ubiquitous language, and you should be able to explain to a domain expert how this part of the software works by drawing a few simple diagrams and using the actual class and method names of the source code.

### Infrastructure

In addition to the three vertical layers, there is also the infrastructure. As the the picture shows, it supports all of the three layers in different ways, facilitating communication between the layers. In simple terms, the infrastructure consists of everything that exists **independently** of our application: external libraries, database engine, application server, messaging backend and so on.

Also, we consider code and configuration files that glues the other layers to the infrastructure as part of the infrastructure layer. Looking for example at the persistence aspect, the database schema definition, Hibernate configuration and mapping files and implementations of the repository interfaces are part of the infrastructure layer.

While it can be tricky to give a solid definition of what kind of code belongs to the infrastructure layer for any given situation, it should be possible to completely stub out the infrastructure in pure Java unit/scenario tests and still be able to use the domain layer and possibly the application layer to work out the core business problems.



