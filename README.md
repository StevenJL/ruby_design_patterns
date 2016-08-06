# GOF Design Patterns for Ruby Synopsis
"Gang of Four" design patterns, applied to Ruby by Russ Olsen, summarized here.

## Design Patterns (the books)

Read Russ Olsen's [Design Patterns in Ruby](https://www.amazon.com/Design-Patterns-Ruby-Russ-Olsen/dp/0321490452), which was inspired by the Gang of Four's ["Design Patterns: Elements of Reusable Object-Oriented Software"](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/B000SEIBB8), which
you should read too.  But if you don't have the time to read, here's a synoposis.  It may be helpful.

### The Meta Pattern

**Don't use a design pattern just for the sake of using a design pattern.  Use a design pattern because it genuinuely adds flexibility and maintainable to your code base.**

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
  def update(delinquent_payment)
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

```ruby
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
  # code that informs finance department of delinquent payment status change
end

delinquent_payment.add_observer do |delinquent_payment|
  # code that informs customer of delinquent payment status change
end
```

### The Command Pattern
**Definition:** Factoring out action code into a "command" class, which runs the action code through public `execute` method. We then pass these command classes into
client classes during instantiation.  This way, we separate out the action logic from the client that calls the action.

**Example:**  E-commerce order UI buttons.

```ruby
# A button class (the client which uses the command classes)
class Button
  attr_accessor :command

  def initialize(command)
    @command = command
  end

  # code related to rendering button

  def on_button_push
    @command.execute
  end
end

# The command classes

class NewOrderCommand
  def execute
    # logic for editing the order
  end
end

class EditOrderCommand
  def execute
    # logic for editing the order
  end
end

class SaveOrderCommand
  def execute
    # logic for saving the order
  end
end

class DeleteOrderCommand
  def execute
    # logic for editing the order
  end
end

new_order_button = Button.new(NewOrderCommand.new)
edit_order_button = Button.new(EditOrderCommand.new)
save_order_button = Button.new(SaveOrderCommand.new)
delete_order_button = Button.new(DeleteOrderCommand.new)

# Note if we didn't use the command pattern and instead opted for one button class
# per type of button, we would have four button classes:
# `NewOrderButton`, `EditOrderButton`, `SaveOrderButton`, `DeleteOrderButton`
# And as we add more types of buttons, we would have to create even more button classes!
```

### The Adapter Pattern

**Definition:** A "client" class wants to invoke a variety of "target" classes.  These target classes have similar functions
but different interfaces.  Adapter classes wrap these target classes so they have a common interface.

**Example:** An OrderShipper class (the client) that calls order classes (targets) through adapters

```ruby
class OrderShipper
  def ship(order)
    weight_lbs = order.shipping_weight_lbs
    cost_in_dollars = order.shipping_cost_dollars
    length_inches = order.length_inches

    # ships the order with the above variables
  end
end

class Order
  attr_reader :shipping_weight_lbs, :shipping_cost_dollars, :length_inches

  # other logic pertaining to orders
end

class EuropeanOrder
  attr_reader :shipping_weight_kg, :shipping_cost_euro, :length_cm
end

class EuropeanOrderAdapter < Order
  def initialize(eo)
    @eo = eo
  end

  def shipping_weight_lbs
    @eo.shipping_weight_kg * KG_TO_LB_CONVERSION
  end

  def shipping_cost_dollars
    @eo.shipping_cost_euro * euro_to_dollar_exchange_rate
  end

  def length_inches
    @eo.length_cm * CM_TO_INCHES_CONVERSION
  end
end

## If its a regular US order, we can just put it in the OrderShipper
order = Order.new
OrderShipper.new.ship(order)

## If its a European order, wrap it in the adapter first
euro_order = EuropeanOrder.new
euro_order_adapter = EuropeanOrderAdapter.new(euro_order)
OrderShipper.new.ship(euro_order_adapter)
```


