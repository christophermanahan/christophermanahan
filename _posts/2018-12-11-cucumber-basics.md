---
layout: post
title: Cucumber Basics
---


Cucumber is an acceptance testing framework. Let's dive in.


### Basics


Cucumber utilizes a hierarchical testing structure to create readable, read-like-a-sentence representations of what your application should do. It reads in plain text files written in a language called Gherkin. Each file represents a _feature_ of your system and contains a variety of _scenarios_ that will be tested. 


These scenarios are defined by _step definitions_ which serve as the connective tissue between the sentence describing the scenario and the runnable _support code_ that tests that the feature has been implemented properly.  This support code is specific to the domain of your system (POSTing to a server, Triggering an onClick event in the UI).


### Features

Every Gherkin file represents a feature and begins with the _feature_ keyword. Features may be followed by an optional description. The description is a deep dive that precisely describes the intent and purpose of a feature and includes any details that are relevant to it's acceptance criteria. 


Features contain several different scenarios which are representative of the different states a system may have to deal with and hopefully provide coverage over edge cases and distinct behavior that must be implemented.

### Scenario

Scenario's represent a specific scenario that describe how the system should behave when presented with a certain situation. In order to get a scenario to pass, the system's behavior must adhere to the scenario's outline.


Scenarios are made up of three steps that are analagous to the _Arrange_, _Act_, _Assert_ paradigm found in unit testing.
1. Manipulate the system to be in a certain state
2. Instruct the system to perform the behavior to be tested
3. Verify the subsequent state of the system


These three steps can be constructed in sentence-like structure using the keywords _Given_, _When_, and _Then_. In addition, _And_ and _But_ can be interwoven into these steps to increase the expressiveness of the scenario.

{% highlight Cucumber %}
Given I ordered a taco through the app yesterday
And I am a tacohort rewards member
When I order a taco today
Then I should get a 10% Discount
{% endhighlight %}

As in any proper testing environment, cucumber scenarios should be independent of one another and not depend on the status of any other scenario.


### Step Definitions


Step definitions serve as the binding between Gherkin scenarios and the concrete implementation of the actions that will be performed in order to test the system. Step definitions describe how your support code should manipulate the system, but do not perform these manipulations themselves. They are kept distinct from the support code that will actually poke and prod the system.


Step definitions bridge this gap between the scenario and support code by using string matching to extract the relevant information from the scenario written in Gherkin and provide them to the support code. This is most commonly executed using regular expressions (regex).


{% highlight ruby %}
Given /^I have bought \$(\d+) worth of tacos (\d+) days ago$/ do |amount, days|
  # TODO: code goes here
End
{% endhighlight %}


While regular expressions are outside of the scope of this post, this presentation by Lea Verou provides a fast-paced wealth of information that covers everything you need to know in order to get started using regex in your Gherkin scenarios!


<a href="http://www.youtube.com/watch?feature=player_embedded&v=EkluES9Rvak&t
" target="_blank"><img src="http://i3.ytimg.com/vi/EkluES9Rvak/maxresdefault.jpg" 
alt="IMAGE ALT TEXT HERE" width="240" height="180" style="display:block;margin:auto" /></a>

The _^_ and _$_ symbols serve as anchors that specify the beginning and end of the scenario snippet.


Another useful feature that allows writing out a series of tests that will all live at the same level of abstraction before coding their concrete support code implementation is _pending_.  Pending allows you to specify what the concrete implementation will do and then move onto writing the next test so that the behavior of the system at this level can be fully specified before implementing the support code.

{% highlight ruby %}
Given /^I have bought \$(\d+) of tacos$/ do |amount| 
  pending("Need to design the purchase history interface") 
end
When /^I buy \$(\d+)$/ worth of tacos do |amount| 
  pending("How do we withdraw funds?") 
end 
{% endhighlight %}


### Expressive Scenarios

Cucumber includes several more key features that allow developers to write expressive scenarios that focus on the important and relevant information about the system under test.


#### Backgrounds


Backgrounds are a section in the feature file that allow you to express steps that are common to the scenarios that will be tested. This is analagous to the beforeEach paradigm in unit testing and serves a similar purpose. Backgrounds isolate setup code so that it must only be changed in one place and allow the reader to focus on the unique and relevant behavior in each scenario. This increases readability but also has the potential to be abused and cause the reader to jump back and forth between a complicated setup routine and the outline in the scenario.

>You want to be in a single responsibility mindset when designing feature tests.

This can be avoided by keeping backgrounds simple and short. If you find your background becoming convoluted it may be a sign that you are testing more than one feature in the file. You want to be in a single responsibility mindset when designing feature tests. In addition, technical details like opening and closing sockets or pinging third party API's belong in your support code and not in your backgrounds.


#### Data Tables

Data tables allow Cucumber users to describe more complex, potentially multi-dimensional data in a format that would not map well to the single lines that _Given_, _When_, and _Then_ require.  They can represent larger data and if you find yourself writing _Given_... _And_... _And_... _And_... then it may be a signal that a data table could increase your test expressiveness.


{% highlight Cucumber %}
Given these tacos:
  | Wrapper | Filling |
  | Hard | Carnitas |
  | Soft | Sofritas |
  | Soft | Beans |
Then the shopping cart should contain:
  | Type | Price |
  | Meat | 4.95 |
  | Tofu | 4.25 |
  | Veggie | 3.55 |
{% endhighlight %}


#### Scenario Outlines


Scenario outlines allow Cucumber users to extract various input values and/or expected outcomes from scenarios that follow a common set of steps.


{% highlight Cucumber %}
Scenario Outline: Buy tacos
  Given I have <Balance> tacopoints
  When I buy <Amount> tacos
  Then I should receive <Additional> tacopoints
  And The balance of my account should be <Updated>
  Examples:
    | Balance | Amount | Additional | Updated |
    | 500 | 1 | 50 | 550 |
    | 1000 | 10 | 550 | 1550 |
{% endhighlight %}


_Placeholders_ are indicated in the scenario outline using angle brackets and serve as variables that can be assigned in the example section of the scenario. You also have the ability to use multiple different tables to test different but related behavior, such as valid and invalid setups.


### Step Definition Internals

A step definition begins by capturing the variable definitions that were defined in the Gherkin test scenarios. Once these variables have been defined they are used to set the system under test into a particular configuration state.  This is often achieved through including libraries in the support code but the system can also be tested using your own custom support code. The step definition can use your support code to poke and prod the system under test. After the captured information has been used to instantiate the support code and the system under tests the relevant methods that will test the system should be called and the support code should be utilized to test the system. A unit testing framework can then be used to verify the results and the acceptance testing cycle for a step is complete.


These steps break down as follows:

1. Capture relevant information from the Gherkin test scenario
2. Use this information to configure the system under test and the support code that will poke the system
3. Capture the results of poking the system
4. Verify these results against the expected outcome using assertions


Examples of support code could include an HTTPRequest that can connect to the server using an HTTP Library or a custom socket connection and return an HTTP Response that could be verified using an xUnit framework against the expected response.


#### Conclusion

Cucumber is a powerful acceptance testing framework that allows plain text language to express powerful and expressive scenarios that can drive the BDD test cycle. 


The Cucumber Book is an excellent resource for further reading and code examples on how to use Cucumber with your next project.


Working with expressive acceptance tests provides a bridge between developers, clients, and business stakeholders and allows effective communication of requirements and acceptance criteria making your development cycle more efficient and less prone to rewrites!
