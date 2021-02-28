# Object-Oriented Principles in PHP - Laracast
> [Episode 7: Object Composition and Abstractions](https://laracasts.com/series/object-oriented-principles-in-php/episodes/5)

## Object Composition and Abstractions

Imagine you have a class called `Subscription`, which would likely have behaviors to; `create` a subscription, `cancel` a subscription, `swap` a subscription (annual to monthly), etc. The `cancel` behavior is complex and requires more functionality than some of these other behaviors. It would likely make an API request, maybe to Stripe, find the Stripe customer, find subscriptions by customer, etc.

### Option 1 - Add behavior directly to the class

```php 
class Subscription
{
    public function create()
    {
        
    }

    public function cancel()
    {
        // call Stripe API
        // find Stripe customer
        // find subscriptions by customer 
    }

    protected function findStripeCustomer()
    {

    }

    protected function findStripeSubscriptionByCustomer()
    {

    }
}
```

This is our first option for adding the billing functionality to the class. However, `findStripeCustomer` and `findStripeSubscriptionByCustomer` aren't really related to `Subscription`, they're related to a billing provider, and a specific provider at that. It's also very likely that we'll want this billing functionality elsewhere in the system and not just in the `Subscription` class.

### Option 2 - Use inheritance

Instead, another option would be to use [**inheritance**](inheritance.md); `BillableSubscription` could extend `Subscription`.

```php 
class Subscription
{
    public function create()
    {
        
    }

    public function cancel()
    {
        // call Stripe API
        // find Stripe customer
        // find subscriptions by customer 
    }
}

class BillableSubscription extends Subscription
{

    protected function findStripeCustomer()
    {

    }

    protected function findStripeSubscriptionByCustomer()
    {

    }
}
```

Now our `Subscription` class is clean, and anything related to billing is in our `BillableSubscription` class. But we're still encountering the problem that our billing methods aren't related to a subscription, they're related to a specific gateway to a billing provider. We also still might want to use these functions elsewhere in our system and we want to avoid duplication.

### Option 3 - Composability

**Object composition** means **combining types** to build a **more complex object**, or when **one class has a pointer to another class** where behavior is located and deferred to. (Pointer meaning, some property on the object).

Instead of inheritance, we can move our protected methods and make them public methods of another class and use it to **compose** our `Subscription` class by using **constructor injection**.

```php 
class StripeGateway
{
    public function findStripeCustomer()
    {

    }

    public function findStripeSubscription()
    {

    }
}

/* Subscription is COMPOSED with another class (StripeGateway) */
class Subscription
{
    // @var \StripeGateway
    protected StripeGateway $gateway; // pointer to another class

    public function __construct(StripeGateway $gateway)
    {
        $this->gateway = $gateway;
    }

    public function create()
    {
        
    }

    public function cancel()
    {
        // call Stripe API
        // find Stripe customer
        $customer = $this->gateway->findStripeCustomer();
        // find subscriptions by customer 
    }
}
```

Now `Subscription` is ***composed*** of a `StriperGateway`. `$gateway` is a pointer to the class in which we have our deferred behavior. Then we can simply use the `$gateway` to find the Stripe customer.

Depending on the size of your application, it may not be a problem to compose your `Subscription` with a specific gateway like `StripeGateway`; it may be the only provider you ever use for your project. However, you never know what else you may need, maybe Braintree offers better prices so in the future you want to switch. 

### Abstraction

It's also a bit strange that our generic `Subscription` knows specifics about our billing provider. `Subscription` doesn't care about what gateway, it only cares that it has one.

We can remove the specifics by ***depending upon abstractions*** with a new generic `Gateway` **interface**.

```php 
// define the functions required for billing gateways
interface Gateway
{
    // rename to generic customer and subscription
    public function findCustomer();
    public function findSubscription();
}

class StripeGateway implements Gateway
{
    // must use methods from contract "signed" 
    // by implementing the interface
    public function findCustomer()
    {

    }

    public function findSubscription()
    {

    }
}

class BraintreeGateway implements Gateway
{
    // must use methods from contract "signed" 
    // by implementing the interface
    public function findCustomer()
    {

    }

    public function findSubscription()
    {

    }
}
```

Now we can abstract away the details by composing `Subscription` with our new generic `Gateway`. Then we can pass it whichever gateway provider we need, without having to change our existing classes.

```php

class Subscription
{
    /**
     * use an interface to create abstraction
     * @var \Gateway
     */
    protected Gateway $gateway;

    public function __construct(Gateway $gateway)
    {
        $this->gateway = $gateway;
    }

    public function create()
    {

    }

    public function cancel()
    {
        // find stripe customer
        $customer = $this->gateway->findCustomer();
        // find stripe subscription
    }
}

// we can pass it any kind of gateway that adheres to 
// the Gateway interface contract
$subscription = new Subscription(new StripeGateway());
// OR 
$subscription = new Subscription(new BraintreeGateway());
```

## Summary 

- **Object composition** means **one class has a pointer, or property, to another class** where behavior is located and deferred to.
- Objects are often composed using *constructor injection*, as seen in our `Subscription` constructor.
- We can remove unnecessary specifics and prevent needing future changes by ***depending upon abstractions***

---

[Episode 6: Encapsulation < Previous ](encapsulation.md) &nbsp; | &nbsp; [OOP in PHP](/oop/) &nbsp; | &nbsp; [Next > Episode 8: Value Objects and Mutability](valueobjects.md)
