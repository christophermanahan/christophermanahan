---
layout: post
title: Functional Exception Handling in Java
---


Handling exceptions in Java requires diligence and strategy in order to determine what to do if our program can recover from failed events and what we should do when we can't. 


Java 8 introduced Functional Java which opens up the world of functional programming techniques to the Java community. 


These techniques can be applied to deal with exceptions in new and interesting ways. So what is an exception, and why should you handle it functionally?


### Exceptions


>An exception is an event, which occurs during the execution of a program, that disrupts the normal flow of the program's instructions.


Exceptions are events that subvert how a program would normally run. They are objects that extend the throwable class. Three types of exceptions exist, checked, unchecked, and errors. This blog post will focus on unchecked exceptions.


For reference, checked exceptions are checked at compile time and any method that may throw an exceptions must include a `throws` clause in it’s signature. This specifies to callers of this method that they must either catch or rethrow this exceptions. Unchecked exceptions are not assessed by the compiler to determine if they have a handler and while they may still be caught and handled, or rethrown, it is not required in order for the code to successfully compile


Exceptions allow methods to exit without returning a value, by handing an exception off to the runtime system.  The runtime system searched the call stack for a called method that can handle the exception. If a method specifies that it can catch the appropriate exception type it will handle the exception according to it’s specification. If no exception handler is thrown, the program will shutdown.
 

### Maybe Type


In order to work with this limitation and specify a technique to handle exceptions without throwing them into the runtime environment we can take advantage of the ability to return Objects, which can act as containers for a variety of state and behavior. This allows us to return objects that contain different state that depends upon whether or not the method caught an exception or successfully executed.


Functional programming provides us with a Maybe type that can either include or not include a value. The maybe type encapsulates this optional value and is a useful construct to deal with nullable values. The maybe type is not just useful for dealing with potential null values however, it also can provide us with a technique to handle recoverable unchecked exceptions gracefully and avoid using exceptions for control flow. Java 8 added the Optional class which is an implementation of the maybe type and includes a variety of chainable methods that allow for a functional style of piped data transformation to be utilized.


In addition we will be discussing the either type which is another paradigm from functional programming that can provide a highly expressive way to implement functional exception handling. While the either type is not included in java we will discuss a small class that can be written using generics to implement this powerful type. However, in certain scenarios we can use an Optional to express whether or not an operation completed without throwing an exception.


### Optional


The way to represent this is simple and clean. If the optional contains data, the method call completed as expected, however if an exception was thrown, the optional will be empty. Why is this advantageous? It gives us the opportunity to handle unchecked exceptions, while making it clear that the method can return different encapsulated values and promotes the caller to handle both the successful (data is present), and failed (empty) scenarios. In addition, the optional class gives us access to the powerful functional interfaces in java, which allow us to easily transform data received from a method that would otherwise have thrown an unchecked exception. The calling class also obtains the opportunity to gracefully exit or recover without handling unnecessary exceptions when it is desired to ignore and recover from the exception.


### Server Example


Let’s look at a simple HTTP server that to demonstrate the elegance enabled by functional exception handling.


#### Procedural Exception Handling
{% highlight java %}
class HTTPServer {
  private Parser parser;
  private Handler handler;
  private Logger logger;

  public HTTPServer(Parser parser, Handler handler) {
    this.parser = parser;
    this.handler = handler;
  }

  public void start(Connection connection) {
    serve(connection);
    close(connection);
  }

  private void serve(connection) {
    try {
      String read = new ConnectionReader(connection).read();
      Request request = parser.parse(request);
      Response response = handler.handle(request);
      connection.send(response);
    } catch (RuntimeException e) {
      close(connection);
    }
  }

  private void close(connection) {
    try {
      connection.close()
    } catch (RuntimeException ignored) {}
  }
}
{% endhighlight %}


This echo server contains a few abstraction.


First, we have a connection that is capable of being read from, written to, and closed. We also have a ConnectionReader that can read a request string in from a connection. This is not injected because it’s behavior is simple and unnecessary too mock in unit tests. In addition we have a parse that can parse data read from the connection into a request. Finally, we have a handler that can handle requests and generate an appropriate response and a logger for well, logging.


Unfortunately, as reading from, writing to, and closing a socket are operations that throw IOExceptions, we need to utilize a try catch block to handle unexpected behavior. We will have the connection rethrow these exceptions as RuntimeExceptions which is an unchecked exception type so that we may choose how to handle the thrown exceptions. For this example we have to make sure the connection gets closed if an exception is thrown so we assert that the connection closes in the catch clause.


This setup is legible and descriptive but the connection must be closed if an exception is caught.  Let’s now assume that instead of a String, the ConnectionReader returns an Optional of the read in request string and see how we can leverage the functional interfaces in Java to produce cleaner code.


#### Optional Exception Handling
{% highlight java %}
class HTTPServer {
  private Parser parser;
  private Handler handler;
  private Logger logger;

  public HTTPServer(Parser parser, Handler handler) {
    this.parser = parser;
    this.handler = handler;
  }

  public void start(Connection connection) {
    serve(connection);
    close(connection);
  }

  private void serve(connection) {
    new ConnectionReader(connection).read
      .map(parser::parse)
      .map(handler::handle)
      .ifPresentOrElse(connection::send, connection::close);
  }

  private void close(connection) {
    try {
      connection.close()
    } catch (RuntimeException ignored) {}
  }
}
{% endhighlight %}


We can see that the method is now a clean chain of calls that closes the connection if an exception is thrown and the Optional is empty using `ifPresentOrElse`. This is more expressive and adding further data transformation capability into the server is as simple as adding another `.map()` into the chain.


Finally, I would like to take a look at the Either class and how we can implement it in Java to obtain another technique to handle exceptions functionally.


#### Either Exception Handling
{% highlight java %}
class HTTPServer {
  private Parser parser;
  private Handler handler;
  private Logger logger;

  public HTTPServer(Parser parser, Handler handler, Logger logger) {
    this.parser = parser;
    this.handler = handler;
    this.logger = logger;
  }

  public void start(Connection connection) {
    serve(connection);
    close(connection);
  }

  private void serve(connection) {
    new ConnectionReader(connection).read
      .map(parser::parse, logger::log)
      .map(handler::handle, logger::log)
      .ifSuccessOrElse(connection::send, connection::close);
  }

  private void close(connection) {
    try {
      connection.close()
    } catch (RuntimeException ignored) {}
  }
}
{% endhighlight %}


Now we obtain the ability to have Either wrap a value if the call succeeded, or wrap an exception if a failure occurred. We can then pipe these into the appropriate functions. In this example we will simply log the exception.


Below is an example implementation of the Either type in Java that utilizes generics and static constructors.


{% highlight java %}
public class Either<S,F> {
  private final S success;
  private final F failure;

  public Either(F f, S s) {
    this.failure = f;
    this.success = s;
  }

  public static <S,F> Either<S,F> success(S s) {
    return new Either<>(null, s);
  }

  public S success() {
    return success;
  }

  public boolean isSuccess() {
    return success != null;
  }

  public static <S,F> Either<S,F> failure(F f) {
    return new Either<>(f, null);
  }

  public F failure() {
    return failure;
  }

  public boolean isFailure() {
    return failure != null;
  }

  public <S,F> Either<S,F> map(Function<S, Either<>> successMapper, Function<F, Either<>> failureMapper) {
    if (isSuccess()) {
      return success(successMapper.apply(success))
    } else {
      return failure(failureMapper.apply(failure));
    }
  }

  public void ifSuccessOrElse(Consumer<S> consumer, Runnable runnable) {
    if (isSuccess()) {
      consumer.accept(success);
    } else {
      runnable.run();
    }
  }
}
{% endhighlight %}


This is a small API that provides the necessary methods but this could be expanded to include the analogies to the full breadth of methods included in Java's Optional type. 


Those methods can be viewed here: 
<a href="https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Consumer.html">Java 11 Optional Class</a>


Java 8 brought a host of powerful new capabilities to Java and we can learn from the paths blazed by the functional programmers before us. Java 8 adds new options to handle exceptions in Java and with some thought and respect, we can write cleaner, better Java code and handle exceptions in new and interesting ways!





