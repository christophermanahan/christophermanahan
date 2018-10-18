---
layout: post
title: Experimenting with the Single Responsibility Principle
---

When we want to probe the real world to understand how it works we often rely on instruments that can measure with a precision that far exceeds our own subjective capabilities. We use thermometers to ascertain a numeric representation of what we would term hot or cold. When developing software we have a tool that allows us to probe our code to understand how it works. This tool is unit testing.

### Experimental Control

Making good, reliable measurements requires a certain level of control over the environment in which the measurement is made. When we use a thermometer to measure the temperature of some system we expect that if the quantity we are measuring has no reason to change that our instrument should give us the same results repeatedly. In this scenario we rely on the systems ability to hold other factors like the pressure and volume constant, so that we can be sure that our result is true and replicatable.

- ##### Quick Aside: Temperature, Pressure, and Volume are seperate quantities that nonetheless affect on one another.

However, if our system was not designed with the single responsibility of managing heat but was also responsible for managing pressure, painful implications become apparent if we ever decide to change how the system manages these quantities. If we asked our system to accept and manage a certain amount of heat, and we have left the mechanisms by which it does so intact, we would expect a measurement now to be the same as a measurement later. However, if without our knowledge someone (or a naive version of us at a different time) changed the mechanisms that manage the pressure of the system in between measurements our thermometer would show us two completely different temperatures. This crisis of measurement has multiple implications that are important to consider when designing systems.

### Unit Tests and SRP

>Changes to one responsibility may impair or inhibit the class’ ability to meet the others.
__Sandi Metz__

A major problem that has been noted by many, notably by Uncle Bob in his article on [The Single Responsibility Principle](https://8thlight.com/blog/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html) shows us how coupling different responsibilities that are likely to change in a single class leads us to a rigid, static codebase. We are fearful that changes to one responsibility in a class could collapse the operation of other responsibilities and this fear prevents useful change. However, there is another potential and notable outcome. If our thermometer tells us two different measurements and we have not altered the mechanisms that manage heat, we lose faith in our thermometer. We may give up on the tool's ability to probe the behavior of the system and simply tinker with the system until it generally does what we want. When working on software, this is the moment where we abandon our testing suite and just throw code at the wall until the system 'works.'  

Part of the reason that the SRP is a cornerstone of good system design is that it allows us to place faith in our tests. It isolates behavior in well defined modules where our thermometer only returns different results when we make visible changes to how the system manages heat. And if we want to tinker with the system and build a more efficient heat manager, we can trust our thermometer when it tells us that our system stayed the same temperature. The SRP provides confidence in our testing suite.

### Well Defined Interfaces

This isolation of behavior also provides an additional benefit. If we have designed two seperate mechanisms in our system for managing heat and pressure, the mechanisms can provide a clear signal to each other when we change how they operate. The ability for the various modules in our system to communicate how they will operate to one another is the basis of an interface. The SRP proves to be a guiding light when we want to build well defined interfaces that only change when we want to alter a modules single responsibility. The SRP enables confidence when we probe behavior (unit test) and helps us build well defined modules that have a clear intent when we communicate between them (interfaces.)

### Seperate Concerns

>We want to increase the cohesion between things that change for the same reasons, and we want to decrease the coupling between those things that change for different reasons.
__Uncle Bob__

Learning to isolate change provided humanity with the scientific revolution when applied to create the experimental method. We now had a powerful tool to probe the behavior of nature and be confident that we could _replicate_ our results. The SRP teaches us the same lesson when applied to our code; we can test with confidence and communicate changes clearly. We just have to be responsible.
