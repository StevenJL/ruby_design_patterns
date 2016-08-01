# GOF Design Patterns for Ruby Synopsis
"Gang of Four" design patterns, applied to Ruby by Russ Olsen, summarized here.

## Design Patterns (the books)

Read [Design Patterns in Ruby](https://www.amazon.com/Design-Patterns-Ruby-Russ-Olsen/dp/0321490452), which was inspired by the Gang of Four's ["Design Patterns: Elements of Reusable Object-Oriented Software"](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/B000SEIBB8), which
you should read too.  But if you don't have the time to read, here's a synoposis.  It may be helpful.

### The Meta Pattern

**Don't use a design pattern just for the sake of using a design pattern.  Use a design pattern because it genuinue adds flexibility and maintainable to your code base.**

### Template Pattern

**Definition:** Implement a base class, which holds the flow logic, but leaves out actual implementation details.  Concrete classes inherit from this base class with
the implementation details.

**Example:** The code fulfills orders to both domestic and international customers.  The steps to fulfilling their orders are the same, but the implementation details are different.

```ruby
class Order
  def fulfill
    charge_customer
    email_customer
    ship_to_customer
  end
end

class DomesticOrder < Order
  def ship_to_customer
    # ship to customer using domestic logic
  end
end

class InternationalOrder < Order
  def ship_to_customer
    # ship to customer using international logic
  end
end
```

### The Strategy Pattern

**Definition:** A 'strategy' is an object that fulfills a particular task.  A 'context' is an object which holds the strategy as one of its attributes and invokes it.  The
context swaps in the right strategy for the right situation.  Using the strategy pattern is an example of the "composition over inheritance" philosophy.

**Example:** Ships orders to local, domestic, and international customers.

```ruby
class OrderFulfiller
  def initialize(order, shipper)
    @order = order
    @shipper = shipper
  end

  def perform
    ...
    @shipper.ship(order)
  end
end

class BaseShipper
  # logic common to all shippers
end

class LocalShipper < BaseShipper
  def ship(order)
    # Ships order to local customers using bike messengers
  end
end

class DomesticShipper < BaseShipper
  def ship(order)
    # Ships order to domestic customers through USPS
  end
end

class InternationShipper < BaseShipper
  def ship(order)
    # Ships order to international customers through DHL
  end
end

# You can swap in the type of shipping you want during the fulfillment of the order
local_shipper = LocalShipper.new
OrderFulfiller.new(order, local_shipper).perform
```


##### The Strategy Pattern (with blocks)

Just like the strategy pattern above, but if the strategies are fairly simple, just pass them in as blocks.  This avoids creating additional
strategy classes.

```ruby

class OrderFulfiller
  def initialize(order, &block)
    @order = order
    @shipper = block
  end

  def perform
    ...
    @shipper.call(order)
  end
end

# The invocation of OrderFulfiller takes in a block, which has contains shipping logic.
OrderFulfiller.new(order) do |order|
  # Ships order to international customers through DHL
end
```

### The Observer Pattern

**Definition:** A class called the 'subject' has state which changes. Other classes called 'observers' would like to informed of these changes.

**Example:**  When delinquent payment changes states, inform the customer and the financial department.

```ruby
class DelinquentPayment
  def intialize
    @observers = []
  end

  def add_observer(observer)
    @observers << observer
  end

  def delete_observer(observer)
    @observers.delete(observer)
  end

  def notify_observers
    @observers.each do |observer|
      observer.update(self)
    end
  end

  def change_status(status)
    @status = status
    notify_observers
  end
end

class Customer
  def update(delinquent_payment)
    # Informs customer of delinquent payment status change
  end
end

class Finance
  def update(order)
    # Informs finance department of delinquent payment status change
  end
end

# Observer pattern in action
delinque_payment = DelinquentPayment.new
customer = Customer.new
finance = Finance.new

delinquent_payment.add_observer(customer)
delinquent_payment.add_observer(finance)

delinquent_payment.change_status(:paid)
```

##### The Observer Pattern (using Ruby's build in observer module)

The observer pattern is so cool that Ruby's standard library actually comes with a prebuilt `Observable` module so we don't have to write
the `add_observer`, `remove_observer`, and `notify_observers` boiler-plate code.

```ruby
require "observer"

class DelinquentPayment
  include Observable

  def change_status(status)
    @status = status
    changed # we need to invoke this before notify_observers when using `Observable`
    notify_observers(self)
  end
end
```

##### The Observer Pattern using blocks

Again, we can use blocks if we don't want to instantiate observer objects.

```
class DelinquentPayment
  def add_obsever(&observer)
    @observers << observer
  end

  def notify_observers
    @observers.each do |observer|
      observer.call(self)
    end
  end

end

delinquent_payment.add_observer do |delinquent_payment|
  # code informs finance department of delinquent payment status change
end

delinquent_payment.add_observer do |delinquent_payment|
  # code informs customer of delinquent payment status change
end
```


