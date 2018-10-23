---
layout: post
title: Techniques to Manage Dependencies and Build Flexible Interfaces
---

Seemingly innocuous code can often be hiding design decisions that will only rear their ugly head as a project morphs and grows over time. When working on a class that represents apartments in a building, it is quite easy to lock your code into a myriad of strangling dependencies and a rigid interface.

We begin our journey into flexing good design techniques with a gets-the-job-done building class.

{% highlight ruby %}
class Building
  attr_reader :apartments, :location

  def initialize(stories, sizes, prices, floors, location)
    @apartments = Array.new(stories) { [] }
    floors.each_with_index do |floor, room|
      @apartments[floor] << Apartment.new(sizes[room], prices[room])
    end
    @location = location
  end

  def get_apartments()
    @apartments
  end

  def get_apartments_by_floor(floor)
    @apartments[convert_index(floor)]
  end

  def get_apartments_by_max_price(price)
    @apartments.flatten.select { |apartment| apartment.price <= price }
  end

  def get_aparments_by_min_size(size)
    @apartments.flatten.select { |apartment| apartment.size >= size }
  end

  def convert_index(index)
    index.to_i - 1
  end
end

class Apartment
  attr_reader :size, :price

  def initialize(size, price)
    @price = price
    @size = size
  end
end
{% endhighlight %}

### Managing Dependencies

This code allows clients of building to get information on apartments in the building using a variety of methods. Building contains an array of apartments that have certain attributes like price and size and it is here that we encounter our first embedded dependency. Whenever you are calling a constructor in a class it should activate your sense of smell. Due to this constructor call our building is now bound to the implementation of the apartment class and any changes to the apartment class will reverberate to all of its dependents including board. 

#### Dependency Injection

>If you are mindful of dependencies and develop a habit of routinely injecting them, your classes will naturally be loosely coupled.
__Sandi Metz__

If we look at building's methods we can see that we only care if an apartment has a size and a price. Maybe we would also like to own buildings that can rent out storage rooms or music practice rooms. In this scenario buiding doesn't care if a room is an instance of apartment or any other type of room as long as we can access it's price and size. 

We can resolve this issue by utilizing dependency injection. This is a complicated sounding technique that boils down to passing instances of classes into the constructor of a class dependent on those instances. As long as those instances adhere to the interface required by the class they are injected into, the type of that class no longer matters to us.

{% highlight ruby %}
class Building
  attr_reader :apartments, :location

  def initialize(apartments, location)
    @apartments = apartments
    @location = location
  end
  ...
end
{% endhighlight %}

We can now utilize a factory to prepare our apartments for injection into building as below.

{% highlight ruby %}
class ApartmentsFactory
  def initialize()
    @apartments = []
  end

  def build(stories, sizes, prices, floors)
    stories.times do
      @apartments << []
    end

    floors.each_with_index do |floor, room|
      @apartments[floor] << build_apartment(sizes[room], prices[room])
    end

    @apartments
  end

  def build_apartment(size, price)
    Apartment.new(size, price)
  end
end

apartment_factory = ApartmentFactory.new
apartments = apartment_factory(1, [1000, 1250, 1100], [1500, 2000, 1300])
Buildings.new(apartments, "120 5th Avenue, NY, NY")
{% endhighlight %}

Our apartments factory is very similar to the original building constructor but we have decoupled building from its functional dependency on apartments by utilizing dependency injection.

#### Structural Dependency

Our next dependency is implied within the building's methods. They are only functional if building is instantiated with a 2 dimensional array in a very specific format (apartments[floor][room].) This is not ideal and makes changing the data structure full of apartments we pass to building impossible. It also has ramifications for any client of building who will not be able to see what floor they live on after using one of building search methods. 

We can begin to decouple this unmanaged dependency by constructing apartments that know what floor they are on and injecting a simpler data structure into building.

{% highlight ruby %}
class Apartment
  attr_reader ..., :floor

  def initialize(size, price, floor)
    @floor = floor
    ...
  end
end

class Building
  def get_apartments_by_floor(floor)
    @apartments.select { |apartment| apartment.floor == floor}
  end

  def get_apartments_by_max_price(price)
    @apartments.select { |apartment| apartment.price <= price }
  end

  def get_aparments_by_min_size(size)
    @apartments.select { |apartment| apartment.size >= size }
  end
  ...
end

class ApartmentsFactory
  def build(sizes, prices, floors)
    rooms = sizes.count
    rooms.times do |i|
      @apartments << build_apartment(sizes[i], prices[i], floors[i])
    end
    @apartment
  end
  ...
end
{% endhighlight %}

Now as long as we inject an enumerable that adheres to the correct interface into building its search methods will be fully functional.

#### Order Dependency

Another dependency reveals itself if we look closely at apartments factory that has been plaguing us in past iterations as well. This is the dependency on argument order that exists in the build method. Unless we pass these arrays in the specified order our factory will produce incorrect apartments. A quick fix in Ruby is to utilize named parameters to prevent the order dependency.

{% highlight ruby %}
class ApartmentsFactory
  def build(sizes:, prices:, floors:)
  ...
end

sizes = [1000, 1500]
prices = [2000, 3000]
floors = [1, 2]
apartments = ApartmentFactory.new.build(
  sizes: sizes
  prices: prices
  floors: floors
)
{% endhighlight %}

### Creating Flexible Interfaces

We want building to have a flexible interface with stable public methods and incoming messages that focus on asking for "what" instead of telling "how." 

#### Well Defined Public Interface

Our first version of building included a convert_input method in it's public interface which implied that clients of building could send that message when in reality it would only be used internally. A quick, small improvement to this version of building is achieved by making this method private. We can also encapsulate apartments in the private interface to encourage access through buildings search methods.

{% highlight ruby %}
class Building
  ...
  private
  attr_reader :apartments

  def convert_index(index)
    index.to_i - 1
  end
end
{% endhighlight %}

#### Inversion of Control

The next question to ask ourselves is whether building's interface is flexible and pliable to changes. A key factor is revealed if we think about what would happen if we wanted to add a new field to our apartments. If our client now required that they would like to know whether an apartment was furnished or not we could add this new field to the apartment class and modify the factory and call it a done deal. 

However, our building class has not been updated to include this new information and clients have no way to search through apartments that are furnished or vice versa. We would have to add yet another specific search method to building and if the fragility of this interface was not obvious before it is now.

A powerful technique we can use to solve this problem is to invert control of the search functionality. Instead of programming a bulky and ever-changing public interface into building that needs to be modified for every new requirement we can utilize Ruby blocks to give control back to the client. The public interface of building exposes an enumerable of apartments that enables the client to utilize a variety of powerful functionality.

{% highlight ruby %}
class Building
  def initialize(apartments)
    @apartments = apartments
  end

  def list
    apartments.to_enum(:each)
  end
  ...
end

# Check if an entire building is furnished
all_rooms_furnished? = building.list.all?(&:furnished)

# Find all apartments bigger than 3000 sq feet
bigger_than_3000_sq_ft = building.list.select do |apartment|
  apartment.size > 3000
end
{% endhighlight %}

Through inversion of control we have exposed every method that can act on an enumerable to clients of building and the public interface of apartment provides documentation for the parameters that are available. This has drastically reduced the size of the public interface and has allowed clients to send messages to building that specify how _they_ want to receive the target apartments instead of having to deal with a predefined response.

These techniques can be applied to transfrom rigid classes with hidden internal dependencies into flexible, modular classes that can be passed into applications without worry. Managing dependencies and crafting flexible interfaces are two key design perspectives that will prevent an application from crystallizing into a codebase that is resistant or even impossible to change. Our software should be open to extension and closed to modification and flexible, modular code defines a path toward this goal.
