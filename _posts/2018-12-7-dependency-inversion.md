---
layout: post
title: Dependency Inversion
---

* High-level modules should not depend on low-level modules. Both should depend on abstractions.
  
* Abstractions should not depend upon details. Details should depend upon abstractions.

___

Dependency inversion begins with these two statements. When we speak of high level modules we are talking about the business logic of an application. High level modules contain the identity of the application; they are responsible for the behavior of the system.

### Depending on Lower Levels

When you have modules that define the identity of your application, you would like these modules to only have to change if the identity of the application changes. They should not have to respond to changes deeper down in the implementation details. 

When the identity of the application depends on lower level modules any changes to these low level modules will result in rewrites to the business logic of the application. 

_This is diametrically opposed to the desired behavior._

High level modules should function independent of the implementation of lower level modules. Those details should be abstracted away from the high level module so that they may be reused just as low level modules are reused to implement commonly desired functionality. By applying dependency inversion we can isolate high level modules from changes in the implementation details of lower level concrete classes. In other words, we can make our application flexible and extensible.

### Layers of Abstraction

Well defined object oriented programs have clearly defined layers of abstraction. These layers allow a developer to build an application the way a mechanic builds a car. Once we place an engine in the car we do not worry about the pistons and cylinders that drive revolution. We design the car around the engines ability to rotate a driveshaft. This is the 'engine abstraction' and it represents one layer in the development of a car. This same principle helps us build software. 

However these layers have the potential to be abused and can lead to transitive dependencies. 

Transitive dependencies result when a naive layering scheme is applied. Having high level modules depend on lower level modules, who in turn depend on even lower level modules results in a structure where a change to a low level module can ‘propagate up’ the dependency structure and cause reverberations that require change to all of the higher level modules that depend upon it. 

How can we avoid this chain of changes?

In order to avoid transitive dependencies the high level module should define abstract interfaces for the modules that will implement behavior they require.

### Clients Define Interfaces

Interfaces are a powerful tool that tell developers to rely on behavior and not implementation. As long as interfaces are implemented, a higher level module does not care about how the implementer of an interface performs its duty, as long as it gets it done efficiently and correctly.

By having higher level modules define the interfaces they need, we have also applied the dependency inversion principle to the interfaces themselves. The high level module now ‘owns’ the interface, because it defines the behavior it requires. Low level modules simply implement that interface, but no longer own the functionality they will provide. They do not own the behavior.

Not only does this inversion of ownership break the dependence of higher level modules on changes to lower level modules, it also breaks the transitive dependency by depending on abstractions. The interface lives within the same package as the high level modules, forcing the lower level module to adhere to its desired structure.

>Depend on abstractions 
-Uncle Bob

It is inevitable that the code we write is volatile, that the interfaces we create will be subject to change. 

This is why it is so important for the client to define the interface it needs. If the interface lives with the concrete class, and the interface of the concrete class changes, this will propagate up to the dependent class and require rewriting the business logic of the application. However, if the client owns the interface, the only time the interface will change is when the client needs it to change, and the dependence of higher level modules on lower level modules has been broken.

### Defining Higher Level Interfaces

How do we define these higher level interfaces?

>The truths that do not vary when the details are changed 
-Uncle Bob

Interfaces should define generic abstractions that can be wielded by modules that identify the business logic of the application. These interfaces should be implemented by more specific, lower level concrete classes that depend on the interface defined by the higher level module.

The target object of the higher level module does not matter. The way the behavior of the interface is implemented does not matter. The only thing that matters is that the higher level policy can assume it has access to efficient and correct behavior.

Breaking the dependence on the identity of the lower level module has another desirable effect. It allows the higher level modules to manipulate and control objects that have not yet been written into existence.

### Conclusion

The proper application of dependency inversion is necessary when designing reusable applications.

It is a primary design principle that allows the design of reusable, resilient, and flexible code. It breaks transitive dependencies and prevents change from propagating through an application requiring the developer to alter everything that it touches.

Change is inevitable. Having change reverberate thorough our applications is not. 

