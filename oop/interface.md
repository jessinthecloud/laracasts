# Object-Oriented Principles in PHP - Laracast
> [Episode 5 : Handshakes and Interfaces](https://laracasts.com/series/object-oriented-principles-in-php/episodes/5)

## Handshakes and Interfaces 

Let's say users in your application can register for your newsletter. To allow for that you might have a `NewsletterSubscriptionsController` with a `store()` method that would call upon an email `CampaignMonitor`. 

> `CampaignMonitor` would likely need an API key from a configuration file.

```php 
class CampaignMonitor 
{
    public function __construct($apiKey)
    {
        $this->apiKey = $apiKey;
    }

    public function subscribe($email)
    {
        // curl request to API to add user
    }
}

class NewsletterSubscriptionsController
{
    public function store()
    {   
        // assuming your framework has a config file/helper
        $newsletter - new CampaignMonitor(config('services.cm.key'));
        // assuming your framework has auth and users
        $newsletter->subscribe(auth()->user()->email());
    }
}
```

Instead of setting all of this up in the `store()` method, let's just ask for a `CampaignMonitor` instead. The code becomes much cleaner and easier to manage.

```php 
class NewsletterSubscriptionsController
{
    public function store(CampaignMonitor $newsletter)
    {   
        // assuming your framework has auth and users
        $newsletter->subscribe(auth()->user()->email());
    }
}
```

At some point down the road, we might have another newsletter subscription provider, `Drip`, alongside of `CampaignMonitor`. Now we have to go through and update our code, remembering to setup the same methods for the new class.

```php 
class CampaignMonitor 
{
    public function __construct($apiKey)
    {
        $this->apiKey = $apiKey;
    }

    public function subscribe($email)
    {
        // curl request to API to add user
    }
}

class Drip 
{
    public function __construct($apiKey)
    {
        $this->apiKey = $apiKey;
    }

    public function subscribe($email)
    {
        // curl request to API to add user
    }
}

class NewsletterSubscriptionsController
{
    public function store(Drip $newsletter)
    {   
        // assuming your framework has auth and users
        $newsletter->subscribe(auth()->user()->email());
    }
}
```

We know that both `CampaignMonitor` and `Drip` have `subscribe()` methods, as would any other providers added down the road. In this way, having a `subscribe()` method is a bit of an *informal contract*; nothing is explicitly required, it's just a simple **handshake**.

If we were to instantiate `NewsletterSubscriptionsController`, we would be sending an instance of one of the providers. But we would be limited to choosing only one because the `store` method is specifying which we are looking for. 

```php 
class NewsletterSubscriptionsController
{
    public function store(Drip $newsletter)
    {   
        /* ... logic ... */
    }
}

$controller = new NewsletterSubscriptionsController();

$controller->store(new Drip(config('services.cm.key')));
```

However, the controller doesn't care whether it receives a `Drip` implementation or a `CampaignMonitor`, because they both function in the same way. The controller only cares that it can *subscribe someone to a newsletter*. We want our function to work no matter what type of subscription provider it is given. There are a couple options that can help us solve this problem.

### Duck Typing

> "If it walks like a duck and it quacks like a duck, then it must be a duck"

We could leverage what's called **duck typing**. If we remove the typehint from the `store()` method, we are then expecting any `$newsletter` object. As long as the `$newsletter` object has a `subscribe` method, then we're good to go. 

```php 
class NewsletterSubscriptionsController
{
    public function store($newsletter)
    {   
        /* ... logic ... */   
    }
}

$controller = new NewsletterSubscriptionsController();

// now we can use either provider without having to change our controller 
$controller->store(new Drip(config('services.cm.key')));
$controller->store(new CampaignMonitor(config('services.cm.key')));
```

### Interface

If we want something more formal, we can define an **interface** using the `interface` keyword to create a *contract*. In our case, the contract is that we're adding a user to a newsletter.

Think of an interface as a **class without behavior**, only method signatures. In other words, the interface doesn't care *how* you subscribe to a user, it only cares that you expose the behavior allowing you to subscribe. Similar to abstract methods, methods in an interface have no body.

```php 
interface Newsletter 
{
    public function subscribe($email);
}
```

Now we can use the interface to indicate what `NewsletterSubscriptionsController` needs in order to `store` a user's subscription.

```php 
class NewsletterSubscriptionsController
{
    public function store(Newsletter $newsletter)
    {   
        // assuming your framework has auth and users
        $newsletter->subscribe(auth()->user()->email());
    }
}
```

The only problem here is that `Newsletter` is an `interface`, not a `class`. We are expecting an implementation of the `Newsletter` interface, we cannot instantiate it directly. Let's setup `Drip` and `CampaignMonitor` to implement the interface.

```php 
class CampaignMonitor implements Newsletter
{
    /* ... methods ... */
}

class Drip implements Newsletter
{
    /* ... methods ... */
}

$controller = new NewsletterSubscriptionsController();

// still works with either provider without having to change our controller 
$controller->store(new Drip(config('services.cm.key')));
$controller->store(new CampaignMonitor(config('services.cm.key')));
```

Now we no longer have a simple handshake, we've actually written down the terms of the contract and both classes have agreed to conform to the terms. If any of the classes don't conform to the contract, an error will occur.

## Summary

- **Duck typing** is when an object's type is not important as long as it contains the expected methods and/or properties.
- An **interface** is essentially a **contract** that defines required behaviors that a class implementing the interface must have.
- An interface is like a **class without behavior**; it does not (and cannot) have properties or method bodies, only method signatures.
- Interfaces cannot be instantiated

## Addendum - Interface vs Abstract Class

You may notice some similarities between **interfaces** and [**abstract classes**](abstract.md), so here's a general comparison of the differences:

| An interface... | An abstract class...  |
|:---|:---|
| does not contain properties  | can contain properties  |
| contains only method signatures  | can contain both method signatures (abstract methods) and complete methods  |
| cannot contain static methods  | can contain static methods (if they are not abstract)  |
| cannot have access modifiers; everything is assumed public  | can contain access modifiers |
| doesn't contain constructors  | can contain constructors  |
| supports multiple inheritance  | does not support multiple inheritance  |


---

[Episode 4: Abstract Classes < Previous ](abstract.md) &nbsp; | &nbsp; [OOP in PHP](/oop/) &nbsp; | &nbsp; [Next > Episode 6: Encapsulation](encapsulation.md)
