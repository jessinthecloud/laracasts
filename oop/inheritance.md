
# Object-Oriented Principles in PHP - Laracast
> [Episode 3 : Inheritance](https://laracasts.com/series/object-oriented-principles-in-php/episodes/3)

## Inheritance 

Inheritance allows an object to acquire, or *inherit*, the traits and behaviors of another object, in the same way the a child inherits qualities from their parents.

For example, you have a coffee maker and the coffee maker can brew coffee.

```php 
/* a standard coffee maker */
class CoffeeMaker
{
    public function brew()
    {
        echo 'Brewing the coffee';
    }
}

(new CoffeeMaker)->brew();

// output: "Brewing the coffee"
```

But some specialty coffee makers can also brew a latte. It wouldn't make sense for `CoffeeMaker` to be able to `brewLatte()` because standard coffee makers can't do that. So we can create a `SpecialtyCoffeeMaker`.

```php 
/* a fancy coffee maker */
class SpecialtyCoffeeMaker
{
    public function brewLatte()
    {
        echo 'Brewing a latte';
    }
}

(new SpecialtyCoffeeMaker)->brewLatte();

// output: "Brewing a latte"
```

A specialty coffee maker can still brew normal coffee, but we don't want to have to repeat ourselves by copying the function into the additional class. Instead, we can use **inheritance** with the `extends` keyword, to indicate that `SpecialtyCoffeeMaker` "is a" `CoffeeMaker`. This additional class is often called a ***child class*** or a ***subclass***.


```php 
/* a fancy coffee maker */
// SpecialtyCoffeeMaker "is a" CoffeeMaker
class SpecialtyCoffeeMaker extends CoffeeMaker
{
    public function brewLatte()
    {
        echo 'Brewing a latte';
    }
}

(new SpecialtyCoffeeMaker)->brewLatte();
// output: "Brewing a latte"

// SpecialtyCoffeeMaker can still brew(), because it
// inherits the function from the parent (CoffeeMaker)
(new SpecialtyCoffeeMaker)->brew();
// output: "Brewing the coffee"
```

Here's another example; a `Collection` consists of several items. This `$collection` is a `Collection` of `Video`s, and we want to calculate the total runtime for all the videos in the collection.

```php
class Collection
{
    protected array $items;

    public function __construct(array $items)
    {
        $this->items = $items;
    }

    /** calculate the sum of every item in the collection 
    that has this key */
    public function sum($key)
    {
        // create a new array of the values of those keys (array_map())
        // [100, 200, 300]
        // then sum those items in new array (array_sum())
        return array_sum(array_map(function($items) use ($key) {
           return $item->key;
        }, $this->items));

    }
}

class Video
{
    public $title;
    public $length;

    public function __construct($title, $length)
    {
        $this->title = $title;
        $this->length = $length;
    }
}


$collection = new Collection([
    new Video('A Video', 100),
    new Video('Another Video', 200),
    new Video('Additional Video', 300)
]);


echo $collection->sum('length');
// output 600

```

The `sum()` function works, but it isn't very clear; what it's really doing is checking the total `$length` of all of the `Videos`. We can make this more clear with a `length()` function, but length in the way that we want it is specific to a collection of **videos**, so it wouldn't make sense to add a `length()` method to the `Collection` class. That would be too specific to one type of collection. Instead, we can use **inheritance**.

A `VideoCollection` class would give us a place to store specific logic that is only needed by `Video`s. Now, we can use the parent function, but its name is more accurate, and perhaps we could add more details, like a label with units (i.e., 600 minutes).


```php 
class VideoCollection extends Collection
{
    public function length()
    {
        // just use the parent method
        // but now we are able to name it more accurately
        return $this->sum('length');
    }
}

$video = new VideoCollection([
    new Video('A Video', 100),
    new Video('Another Video', 200),
    new Video('Additional Video', 300)
]);


echo $video->length();
// output 600
``` 

We can also refactor the `sum()` function; `array_map()` is simply grabbing all of the keys from each item, which we can do easily with `array_column($array, 'column_name')`

```php 
class Collection
{
    protected array $items;

    public function __construct(array $items)
    {
        $this->items = $items;
    }

    /** calculate the sum of every item in the collection 
    that has this key */
    public function sum($key)
    {
        // get each item from an array where its key=$key 
        // sum the values of those items
        return array_sum(array_column($this->items, $key);
    }
}
```

We can use inheritance with our achievement system example as well. `AchievementType` will hold generic traits and behaviors, and can be extended by more detailed subclasses. 

For instance, every achievement type may have a name, a difficulty, and a points threshold. Defining the default values and behaviors in the parent class makes the child classes much cleaner.

```php 
/** setup defaults */
class AchievementType
{
    public function name()
    {
        // return name based on class name
        // "Achievement Type"
    }

    public function difficulty()
    {
        return 'intermediate';
    }

    public function icon()
    {
        return '/images/'.$this->name.'.png';
    }
}

class FirstThousandPoints extends AchievementType
{
    // will be unique for every achievemnt type
    public function qualifier($user)
    {
        // determine the requirements for getting this achievement
    }
}
```

Now if we need to customize something for the child class, we can **override** the parent's methods. If one of our child classes has a name that does not fit the convention of matching its class name, the child class can have its own `name()` method that will take precedence.

```php 
/** setup defaults */
class AchievementType
{
    public function name()
    {
        // return name based on class name
        // "Achievement Type"
    }

    /* ... other methods ... */
}

class FirstThousandPoints extends AchievementType
{
    /* other methods */

    // override parent when need special behavior
    public function name()
    {
        return "Welcome Aboard!";
    }
}
```

## Summary 

- **Inheritance** allows an object to inherit traits and behaviors from another object, like a **parent** and a **child**
- Child classes can still use the methods of their parents
- Parent classes allow us to define default behaviors and values to keep child classes cleaner 
- Child classes can **override** their parent class's methods when they need custom functionality by defining their own methods of the same name


> You don't need to use inheritance in every situation, but sometimes it makes for a much better API or presentation. This makes your code more readable and easier to use by other developers that may have to work with it.

---

[Episode 2: Objects < Previous](objects.md) &nbsp; | &nbsp; [OOP in PHP](/oop/) &nbsp; | &nbsp; [Next > Episode 4: Abstract Classes](abstract.md)
