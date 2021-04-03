# SOLID Principles in PHP - Laracast
> [Episode 2 : Open-Closed Principle](https://laracasts.com/series/solid-principles-in-php/episodes/2)

## Open-Closed

***Entities (class, method, function) should be open for extension, but closed for modification***

"***Open for extension***" means it should be **simple** to change the behavior of an entity (or a class).

"***Closed for modification***" means change the behavior of the entity (class) without modifying source code (by using ***extension***)
> This is a **goal** to strive for, it's very difficult to follow perfectly

### Why?

**Avoid code rot** -- If we could somehow manage to conform to this principle, then it's possible to change the behavior of your code without ever touching the original source.

### Examples

Imagine your boss wants you to prepare a shape, so we have a class called `Square`

```php
namespace Acme;

class Square 
{
    public $width;
    public $height;

    public function __construct($width, $height)
    {
        $this->width = $width;
        $this->height = $height;
    }
}
```

Next your boss tells you we need some way to calculate the area of a set of squares.

Following the [single responsibility principle](single.md), we can create a class dedicated to calculating area

```php
namespace Acme;

class AreaCalculator {
    
    public function calculate($squares)
    {
        $area = 0;
        foreach($squares as $square){
            $area += $square->width * $square->height;
        }
        return $area;
    }
}
```

Alternatively we could calculate by creating an array of areas:

```php
public function calculate($squares)
{
    foreach($squares as $square){
        $area []= $square->width * $square->height;
    }
    return array_sum($area);
}
```

At this point, everything is great, but then your boss tells you we also need circles. No problem, we'll add a `Circle` class with `$radius` instead of `$width` and `$height`.

```php
namespace Acme;

class Circle {
    public $radius;

    public function __construct($radius)
    {
        $this->radius = $radius;
    }
}
```

However, now we're again told we need to support calculating the area of a `Circle`. We've called everything `Square` inside the `AreaCalculator`, so now we have to modify the class code and realize we have **broken the open-closed principle**.

Let's continue down that path, changing `$square` to `$shape`

```php 
class AreaCalculator {
    
    public function calculate($shapes)
    {
        foreach($shapes as $shape){
            $area []= $shape->width * $shape->height;
        }
        return array_sum($area);
    }
}
```

Okay, that's a little more flexible, but the forumla for the area of a circle is not width * height, it's radius squared * pi, so how do we do this? 

Here is a glaringly obvious symptom of breaking the open-closed principle: We'd have to check the type of shape before calculating.

```php
class AreaCalculator {
    
    public function calculate($shapes)
    {
        foreach($shapes as $shape){
            if(is_a($shape, 'Square')){
                $area []= $shape->width * $shape->height;
            }
            else{
                $area []= $shape->radius * $shape->radius * pi();
            }
        }
        return array_sum($area);
    }
}
```

Clearly, this class has broken the open-closed principle; it was open for modification when it should have been closed.

You may think, *"Eh, it's not a big deal, add a simple check and move on. Why worry about these principles."* 

But what about when your boss returns needing support for a `Triangle` class and its area? Should we really go back and add another check every time we need a change?

This is the type of thing that leads to code rot and breakage.

### What is the solution?

When you have a module that you want to extend without modifying, ***separate the extensible behavior behind an interface, and then flip the dependencies around***.

1. Separate the extensible behavior behind an interface

Let's add an interface `Shape`

```php
namespace Acme;

/**
 * interface is a contract that any implementation must adhere to
 */
interface Shape {
    // we just need some way to determine what the area is
    public function area();
}
```

And implement the interface for each concrete class

```php
class Square implements Shape
{
    public $width;
    public $height;

    public function __construct($width, $height)
    {
        $this->width = $width;
        $this->height = $height;
    }

    /**
     * function from interface
     * by implementing the Shape interface, we "signed a contract"
     * to use its methods in this class
     * 
     * @return [float] the area of the square
     */
    public function area()
    {
        return $this->width * $this->height;
    }
}

class Circle implements Shape {
    public $radius;

    public function __construct($radius)
    {
        $this->radius = $radius;
    }

    /**
     * @return [float] the area of the circle
     */
    public function area()
    {
        return $this->radius * $this->radius * pi();
    }
}
```

2. Flip the dependencies around

    If we go back to `AreaCalculator`, instead of doing the specific checks, we can simply call the method that is guaranteed by the interface "contract".

    ```php
    class AreaCalculator {
   
        public function calculate($shapes)
        {
            foreach($shapes as $shape){
                // we know this is callable because each shape class uses the shape interface
                // guaranteeing that the method is defined
                $area []= $shape->area();
            }
            return array_sum($area);
        }
    }
    ```

Now when the boss says they need a `Triangle` and triangle area, we never have to update the existing code, we simply:

1. create the `Triangle` class
2. make sure `Triangle` implements the common `Shape` interface
3. define its `area()` method and unique formula
4. and then are able to pass that in to `AreaCalculator`

Coding to a common interface means we were able to flip things around and avoid future changes.

You may notice these design principles sort of have each others' backs. There are a lot of similarities between them.

> 10 minutes in


## Summary

-

---

[Episode 1: Single Responsibility < Previous](single.md) &nbsp; | &nbsp; [SOLID Principles in PHP](/solid/) &nbsp; | &nbsp; [Next > Episode 3: Liskov Substitution](liskov.md)
