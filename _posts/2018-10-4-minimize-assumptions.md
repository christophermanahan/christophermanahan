---
layout: post
title: Minimize Assumptions - Simple Design & SOLID
---

---
This blog post assumes familiarity with Conway's Game of Life.

While reading The Four Rules of Simple Design (FRSD) a pattern emerges across several of the examples that seemed to sit at a crossroads between simple design and SOLID principles. I will refer to this pattern as 'minimize assumptions' and I believe it is an important sub-component of writing easily extensible code. 

### Duplication of Knowledge about Topology

The first encounter that displays how assumptions can harm our code quality appears in the "Duplication of Knowledge about Topology" example in FRSD. Here, a set_living_at method has been implemented on the World class to set a living cell to a specific coordinate in the game world.

{% highlight ruby %}
class World
  def set_living_at(x, y) # Better get flat
    #...
  end 
end
{% endhighlight %}

Unfortunately, we discover that the coordinates dimensionality has been hard coded into the method as well as all cell types that need access to said coordinates.

{% highlight ruby %}
class LivingCell 
  attr_reader :x, :y # 'Cause there ain't no going back
end
class DeadCell
  attr_reader :x, :y
end
{% endhighlight %}

What immediately follows is a limitation on our ability to extend the code in an interesting and perhaps desirable manner.

By making a strong assumption about the dimensionality of our world's representation, we have removed the ability to easily extend our code to handle not just 3-dimensions (which could be extremely visually interesting) but any number of higher dimensions. This assumption would have spread like a virus throughout our codebase as well, with all cell types making the same strong assumption and producing a codebase that disregards the Open Closed principle and disallows interesting extensions to our project without significant modification. This virus-like decision could potentially be deeply embedded within our code, affecting everything from how we check a location's neighbors to how we define the parameters that govern the logic of the game. 

{% highlight ruby %}
class ExampleCell 
  attr_reader :x, :y

  def neighbors
    for i in y-1..y+1 do
      for j in x-1..x+1 do
        # Have fun modifying this method
      end
    end
  end
end
{% endhighlight %}

By removing this assumption we can avoid situations such as values like '2' and '3' being hardcoded into the logic that dictates the neighboring cell rules for life and death and allow these values to be dynamically decided by the client when they choose their desired game dimensionality. 

Haines suggests a solution that generalizes x and y coordinates to the concept of a class that is readily swappable with locations classes of different dimensionality.

{% highlight ruby %}
class Location 
  attr_reader :x, :y 
  def neighbors
    # Calculate neighbor stuffs
  end 
end

class World
  def set_living_at(location) # Ayy I could be any dimensionality!
    #...
  end
end

class ExampleCell 
  attr_reader :location # Virus eradicated
end
{% endhighlight %}

This immediately creates a richer, more extensible codebase that can be used to build all new Games' of Life with different behavior, parameters, and potential visuals when rendered.

### Don’t Have Tests Depend on Previous Tests

The second encounter with problematic assumptions occurs in the "Don’t Have Tests Depend on Previous Tests" example. This example is meant to elucidate how creating dependencies between our tests can lead to a fragile testing suite, but I believe it also illustrates the broader potential mistake of making an assumption. The issue begins in an initial test to check if a new world is empty when constructed.

{% highlight ruby %}
def test_a_new_world_is_empty 
  assert_true World.new.empty? # I declare new worlds to be empty, peasants.
end
{% endhighlight %}

A subsequent behavioral test is then implemented to check that after the game state is updated a single time, an empty world stays empty. The crux of the issue is that this second test creates a new world, updates the game state, and then checks if the world is still empty. 

{% highlight ruby %}
def test_an_empty_world_stays_empty_after_a_tick
  world = World.new # Says who?
  next_world = world.tick
  assert_true next_world.empty?
end
{% endhighlight %}

And therein lies the problem; an assumption has been made that a new world is created empty. Should we choose to alter this design in the future and initially construct non-empty worlds, this test will mysteriously fail and display the fragile test coverage our unnecessary assumption has caused. By minimizing assumptions and realizing that we can drive our code with TDD, we can employ a builder pattern that provides a method for creating and returning an empty world without any dependencies on our initial configuration, thus strengthening our testing suite.

{% highlight ruby %}
def test_an_empty_world_stays_empty_after_a_tick
  world = World.empty # No worldly assumptions up in here
  next_world = world.tick
  assert_true next_world.empty?
end
{% endhighlight %}

### Procedural Polymorphism

The third and final encounter I discovered that elucidates how minimizing assumptions can create higher quality, more extensible code occurs in the "Procedural Polymorphism" example. Up until this point, a cell class has been implemented that uses an if else branch to execute different behavior based on the 'alive' status of the cell.

{% highlight ruby %}
class Cell 
  # ...
  def alive_in_next_generation? 
    if alive # You're either alive. Or you ain't. Make up your mind!
      stable_neighborhood?
    else
      genetically_fertile_neighborhood?
    end 
  end
end
{% endhighlight %}

And once again this code makes a strong assumption about the nature of the game that has the same effect of limiting our ability to extend the code with interesting and potentially desirable features. 

Haines initial attempt to remedy this is to use a state property that can take on values of 'alive' or 'dead' but this leads us to the same limitations that hard coding coordinates into our world and cells hit earlier.

{% highlight ruby %}
class Cell # ...
  def alive_in_next_generation? 
    if state == ALIVE # I said it once. I won't say it again.
      stable_neighborhood?
    elsif state == DEAD
      genetically_fertile_neighborhood?
    end 
  end
end
{% endhighlight %}

By utilizing hard coded strings we have eliminated the potential to easily extend our game with new and potentially game altering cell types that we may want to implement to create a richer, more varied Game of Life. Haines uses the Zombie Cell as an example but I leave it to the imagination to think of how introducing new cell types with their own sets of rules could create novel forms of the Game of Life that could produce potentially beautiful animations and static constructs. 

{% highlight ruby %}
class AsexualCell
  def alive_in_next_generation?
    # I don't need no friends, they're all in my head.
  end
end
{% endhighlight %}

By utilizing the idea of inverted composition introduced in the "Inverted Composition as a Replacement for Inheritance" example, we can create a codebase that is readily extensible so that we may experiment with a variety of different cells and their emergent interactions.

{% highlight ruby %}
class Location
  attr_reader :x, :y 
  attr_reader :cell
end

class LivingCell
  def stays_alive?(number_of_neighbors)
    number_of_neighbors == 2 ||
    number_of_neighbors == 3
  end
end

class DeadCell
  def comes_to_life?(number_of_neighbors)
    number_of_neighbors == 3
  end
end
{% endhighlight %}

### IRL: Tic Tac Toe

The inadvertant decision to minimize assumptions naturally appeared when writing my first Tic Tac Toe application. Early on I made the decision to not include any explicit hard coded boards into my source code. This decision lead me to use for loops with different starting and ending points, and increment values to check for horizontal, diagonal, and vertical wins. 

{% highlight ruby %}
class Board
  def checkLoop(board, symbol, loopStart, loopEnd, increment)
    # Yay abstraction, rows, columns, and diagonals can all use this method
    (loopStart..loopEnd).step(increment) do |position|
      return false if board[position] != symbol
    end
    true
  end

  def checkRowsForWin(board, size, symbol)
    start = 0
    (start..board.length).step(size) do |rowStart|
      return true if checkLoop(board, symbol, rowStart, rowStart+size, 1)
    end
    false
  end
  # ...
end
{% endhighlight %}

Without knowing it at the time, I had minimized the assumptions I made about the games format. Later on, when tempted out of curiosity to add the option to play 4x4 and 5x5 tic tac toe boards to flex the alpha beta algorithm a bit (which failed until the magic of memoization, another post for another time,) I found that because my game logic was decoupled from assumptions about its format, implementing this feature was as simple as asking the user to choose a board size. 

> Simple design allows small changes to implement powerful new features without modifying existing code.

And that's the whole point. By following the Four Rules of Simple Design from the start and minimizing assumptions between classes, extending the codebase became easier. Simple design allows small changes to implement powerful new features without modifying existing code. Solid and simple were best friends all along!
