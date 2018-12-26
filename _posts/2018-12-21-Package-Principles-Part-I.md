---
layout: post
title: Package Principles, Part I
---

>“The design of large systems depends critically on good component design”


Packages allow teams to focus on and deploy isolated pieces of a system rather than the system as a whole. Grouping classes into packages allows developers to reason about higher level design at a new level of abstraction.


However, just as dependencies between classes can become tangled and coupled, so too can dependencies between packages if left unmanaged as an application grows and requirements change.


Two groups of six package principles have been developed and refined over the years to help create clean, robust, and maintainable packages that can be depended on by developers and stand the test of time. This blog post is about the first group, the __Principles of Package Cohesion__.

### The Reuse/Release Equivalence Principle


>“The granule of reuse is the granule of release”


If developers want to be able to reuse software packages, they cannot do so unless a release mechanism for that software exists. If the granule of release was simply at the source code level, we would be forced to copy and paste code, and any updates to that code or bug fixes would not be received by developers reusing the code. Therefore, the granule of release must be at the level of reusable packages in order to ensure that the client code and source code are synchronized and do not diverge.


This signifies that packages are the granule of release, and offers hints as to how to structure our code to allow it to be reusable and releasable


#### All or None


If a package is to be reused, all of the classes within it must also be capable of being reused. 


We cannot have classes that are coupled to other packages as this would leave the package dependent and it’s true granule of reuse would have to include the package it is coupled too.


#### Domain Independence


We must structure our code so that the domain of the package is independent of other domains. A package must follow the single responsibility principle to encourage maximum reusability


If we couple two responsibilities together, such as an electronic payment library and a shopping cart, clients can no longer utilize the functionality of one responsibility without the other, and the package has become less able to be reused in a wider variety of scenarios.


By adhering to these two sub-principles we can ensure that our packages can be reused by other developers, and that our packages are modular.


### The Common Reuse Principle


>“The classes in a component are reused together. If you reuse one of the classes in a component, you reuse them all.”


The CRP is the analogue of the Interface Segregation Principle for packages.


>"No client should be forced to depend on methods it does not use" -ISP


The ISP guides us to only depend on interfaces if we depend on the entirety of the interface. If we are depending on fragments of an interface in one class, and other fragments in another, it is a signal that our interface may have more than one responsibility and should be segregated.


The CRP makes a similar assertion for packages. If you are going to reuse a packaged class, you must depend on the entire package. Just as the ISP provides a signal as to what methods belong in an interface, and which do not, the CRP tells us which classes do not belong within a package. The CRP says more about which classes _do not_ belong in a package rather than those that do. If a class is not tightly coupled to other classes within a package, it is a signal that it does not belong. 


A single change to a class inside a package, whether or not a client of the package cares about that change or not, requires an entire re-release and re-deployment of that package. This is costly and should only occur if clients of the package actually care about the update. Therefore packaging together classes that do not change together increases the likelihood of costly and unnecessary re-releases.


### The Common Closure Principle


>“The classes in a component should be closed together against the same kinds of changes. A change that affects a component affects all the classes in that component and no other components.”


The CCP is the Single Responsibility Principle Analogue for packages.


>"We want to increase coupling between classes that change for the same reasons, and separate those that change for different reasons" -SRP


When changes inevitably occur, we want to isolate the number of places they occur in. The best way to do this is to keep classes that are likely to influence each other, to depend on each other, in the same packages.


By making sure a package has one and only one responsibility, we help reinforce that changes to code within the package are isolated within that package and do not require changes to other classes in other packages.


CCP attempts to minimize the number of changes that occur across packages when requirements change by providing a common closure around classes that all work together to implement a single abstraction.


### Conclusion

This blog post covers the first three of the package principles, the __Principles of Package Cohesion__. Part II will cover the second three, the __Principles of Package Coupling__.














