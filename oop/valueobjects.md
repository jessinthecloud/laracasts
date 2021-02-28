# Object-Oriented Principles in PHP - Laracast
> [Episode 8 : Value Objects and Mutability](https://laracasts.com/series/object-oriented-principles-in-php/episodes/8)

## Value Objects and Mutability

At its core, a **value object** is an object that is **defined according to its value**, or its data, rather than its identity. 

For instance, let's say in order to register a user, we need a name and an age.

```php 
function register(string $name, int $age)
{
    // logic to register the user
}

// valid code
register('John Doe', 35);
// also valid code, but shouldn't be
register('Jane Doe', -35);
```

Passing a negative `$age` is *technically* valid due to its data type (`int`), but we know an age should never be negative. You also would never have an `$age` of 500.

To solve this issue, we could perform basic validation:

```php 
function register(string $name, int $age)
{
    // validate age
    if($age < 0 || $age > 120){
        throw new InvalidArgumentException('The age is not valid');
    }

    // logic to register the user
}

// ERROR
register('Jane Doe', -35);
// ERROR
register('Jane Doe', 500);
```

Now we are ensuring that age is valid, but probably realizing that a type of `int` is not truly accurate; `$age` is not only an integer, it's a specific type of integer (a human age). When you run into these situations it's often and indication that you might want to extract that data into a **value object**.

We can do this by creating a new type, `Age`, and we can do that by creating a class and passing that to `register` instead. Now if we pass an `Age` of 500, we get an error because the `Age` value object is ***protecting its consistency***.

```php 
// Age is a value object
class Age
{
    public $age;

    public function __construct($age)
    {
        // validate age
        if($age < 0 || $age > 120){
            throw new InvalidArgumentException('The age is not valid');
        }

        $this->age = $age;
    }
}

function register(string $name, Age $age)
{

}

// ERROR
register('John Doe', new Age(500));

// valid
register('Jane Doe', new Age(35));

```

Let's take a closer look at our `Age` value object. We've set the `$age` property to `public`, and this could allow the validation to be bypassed altogether by directly setting the value of `$age`.

```php 
// valid
$userAge = new Age(35);

// uh oh
$userAge->age = 500;

// ERROR - $age is now 500
register('Alex Doe', $userAge);
```

Thankfully, we can solve this problem by setting the property to private. What happens if we want to change the value of `$age`? We can do this in the same way we would for an integer; simply increment. 

If you have a **value object** and you want to **change some of its internals**, you must **create a new instance of that object**. 

```php 
$age = new Age(35);

// want age to increase, create a new object
$age = new Age(36);
```

### Mutable vs immutable

A ***mutable object*** is an object whose **internal state CAN be changed**. 

An ***immutable object*** is an object whose **internal state can, or should, NEVER change**. 

For example, imagine we have here a mutable version of `Age` and we can increment without creating a new instance.

```php 
/* mutable object */
class Age
{
    private $age;

    public function __construct($age)
    {
        // validate age
        if($age < 0 || $age > 120){
            throw new InvalidArgumentException('The age is not valid');
        }

        $this->age = $age;
    }

    /* mutable object; the value can change */
    public function increment()
    {
        $this->age+= 1;
    }

    /* getter for our private property */
    public function get()
    {
        return $this->age;
    }
}

$age = new Age(35);

// mutable; change the value
$age->increment();

// same object becomes 36
echo $age->get();
// output: 36
```

`Age` is a **mutable object** because we changed its internal state.

To make `Age` **immutable**, instead we can return a new instance of `Age` from `increment`. 

```php 
/* immutable object */
class Age
{
    private $age;

    public function __construct($age)
    {
        // validate age
        if($age < 0 || $age > 120){
            throw new InvalidArgumentException('The age is not valid');
        }

        $this->age = $age;
    }

    /** immutable object will return another instance
    instead of changing its values */
    public function increment()
    {
        return new self($this->age + 1);
    }

    public function get()
    {
        return $this->age;
    }

}

$age = new Age(35);

// immutable; return a new instance with the new value
$age = $age->increment();

// new object returned with value of 36
echo $age->get();
// output: 36
```

> There are pros and cons to mutability vs immutability. As it relates to PHP specifically, mutability can get a little tricky; things you often think are immutable are actually mutable, and it's a bit higher level than where we are in this series.
>
> As a general rule, often when you have an immutable object it will lead to fewer bugs due to the limit to internal changes.

There are a few benefits to creating a **value object**:

- it helps avoid *primitive obsession*; we started with an integer, but it wasn't really *just* an integer
- avoiding primitive obsession also leads to better **readability**; `Age` is clearer than `int` alone
- performing the validation in the constructor adds **consistency**
- by avoiding *setters* and changing the properties to `private`, we gain the benefits of **immutability**

### Use cases

Be careful not to overuse value objects. It does not always make sense to specifically type every object, for instance: 

```php 
function register(FirstName $first, LastName $last, Age $age, EmailAddress $email, Password $password)
```

Sure, you might have a good use case for a `Password` type, but if not, you could simply perform validation in your controller and make sure you don't set the password until after validation
 
`EmailAddress` has an even more legitimate case for a value object if you want to validate the email, but `FirstName` and `LastName` really shouldn't be types. Mapping every primitive to a custom type is rarely worth it; only reach for a value object when it clearly offers a benefit.

### Connected data sets

Let's take a look at this set of data. `$x` and `$y` are connected, as are `$x1` and `$y1`, and `$x2` and `$y2`. You can't have one without the other, or the data becomes invalid.

```php 
function pin($x, $y){

}

function distance($x1, $y1, $x2, $y2)
{

}
```

In situations when data is connected and dependent, such as coordinates like this, we can give it a proper type. We can make the properties public because we aren't very concerned about protecting the integrity, we really just want to group the data.

```php 
/* valie object */
class Coordinates
{
    public $x;
    public $y;

    public function __construct($x, $y)
    {
        $this->x = $x;
        $this->y = $y;
    }
}

function pin(Coordinates $coordinates)
{
    $x = $coordinates->x;
    $y = $coordinates->y;
}

function distance(Coordinates $begin, Coordinates $end)
{
    $x1 = $begin->x;
    $y1 = $begin->y;

    $x2 = $end->x;
    $y2 = $end->y;
}
```

There are many different situations where value objects would make sense, especially with money. If you're constantly passing through a currency type and the amount ($5 USD) passed together, why not create a new type to house both the amount and the currency.

```php 
class Money
{
    private $currency;
    private $amount;

    public function __construct($currency, $amount)
    {
        $this->currency = $currency;
        $this->amount = $amount;
    }
}
```

### Value vs Identity

There may be more than one person in the world named "Jeffrey Way", but they don't share the same **identity**. 

On the other hand, if three five dollar bills are on a table, there is no identity defining them, they are defined by their **value**, which is identical for each one.

## Summary 

- A **value object** is an object that is **defined according to its value**, or its data, rather than its identity
- A ***mutable object*** is an object whose **internal state CAN be changed**
- An ***immutable object*** is an object whose **internal state can, or should, NEVER change**
- A value object ***protects its consistency*** by performing validation in its constructor, avoiding *setters*, and keeping properties private (it is immutable)
- value objects help avoid *primitive obsession*; we started with an integer, but it wasn't really *just* an integer
- Avoiding primitive obsession also leads to better **readability**; `Age` is clearer than `int` alone
- Not all primitives need value objects or custom types 
- Connected or dependent data can greatly benefit from custom types

---

[Episode 7: Object Composition and Abstractions < Previous ](composition.md) &nbsp; | &nbsp; [OOP in PHP](/oop/) &nbsp; | &nbsp; [Next > Episode 9: Exceptions](exceptions.md)
