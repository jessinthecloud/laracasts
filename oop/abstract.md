# Object-Oriented Principles in PHP - Laracast
> [Episode 4 : Abstract Classes](https://laracasts.com/series/object-oriented-principles-in-php/episodes/4)

## Abstract Classes 

An abstract class provides a template, or base, for a subclass. Let's recall our achievement system. If we have many different kinds of achievements that have similiar properties and behaviors, we may end up repeating ourselves quite a bit.

```php 
class FirstThousandPoints
{
    public function name()
    {
        return "First Thousand Points";
    }

    public function icon()
    {
        return 'first-thousand-points.png';
    }

    public function qualifier($user)
    {

    }
}

// this class is extremely similar to the first
// we're repeating methods that probably don't need to be unique to each class
class FirstBestAnswer
{
    public function name()
    {
        return "First Best Answer";
    }

    public function icon()
    {
        return 'first-best-answer.png';
    }

    public function qualifier($user)
    {
        
    }
}
```

We'd be far better off with a **parent class** to start off with a template or a base that our classes can build off of. Since the values returned by the methods are simply versions of each classes name, we can move these methods into the parent and determine their values dynamically instead. 

> Native to PHP, `ReflectionClass` reports information about a class. We can use it to determine the current class name -- [Learn More](https://www.php.net/manual/en/class.reflectionclass.php)


```php  
class AchievementType
{
    /* set defaults by using class name to set values */
    public function name()
    {
        // short name is name of class by itself without namespace prefix
        $class = (new ReflectionClass($this))->getShortName();

        // convert literal class name to readable string
        // FirstThousandPoints --> First Thousand Points
        // take capital letter and replace with a space and itself
        return trim(preg_replace('/[A-Z]/', ' $0', $class));
    }

    public function icon()
    {
        // 'first-thousand-points.png'
        return strtolower(str_replace(' ', '-', $this->name())).'.png';
    }
}

class FirstThousandPoints extends AchievementType
{
    public function qualifier($user)
    {
        // qualifier logic
    }
}


class FirstBestAnswer extends AchievementType
{
    public function qualifier($user)
    {
        // qualifier logic        
    }
}

$achievement = new AchievementType();

echo $achievement->name();
// output: "Achievement Type"
echo $achievement->icon();
// output: "achievement-type.png"


$achievement = new FirstBestAnswer();

echo $achievement->name();
// output: "First Best Answer"
echo $achievement->icon();
// output: "first-best-answer.png" 

```

Now we can simply extend the parent class and use its methods instead of repeating them in each class. But something seems odd; there's no reason for`AchievementType` to ever be instantiated, its only purpose is to provide a base for the subclasses. 

We can make `AchievementType` an **abstract class** instead. Abstract classes *cannot* be instantiated, so this solves our problem of never wanting to create `AchievementType` objects directly.

```php  
abstract class AchievementType
{
    public function name()
    {
        $class = (new ReflectionClass($this))->getShortName();
        return trim(preg_replace('/[A-Z]/', ' $0', $class));
    }

    public function icon()
    {
        return strtolower(str_replace(' ', '-', $this->name())).'.png';
    }
}

class FirstThousandPoints extends AchievementType
{
    public function qualifier($user)
    {
        // qualifier logic
    }
}

class ReachTop50 extends AchievementType
{

}

// we can't do this anymore, it will cause a PHP error 
// $achievement = new AchievementType();

$achievement = new ReachTop50();

echo $achievement->name();
// output: "Reach Top 50"
echo $achievement->icon();
// output: "reach-top-50.png" 
```

However, there's more that the `abstract` keyword can do for us. Notice that we forgot to define a `qualifier()` method for `ReachTop50`? We can't really define the whole method in `AchievementType` because it's going to behave differently in each class. What we can do is define an **abstract method**.

**Abstract methods** are not implemented directly, they need specifics from a subclass, But they let us define a requirement for the subclasses. If an abstract method is defined in the parent class, the subclass ***must*** define its own version of that method. This way we can more easily guarantee that subclasses are doing everything they're supposed to.

```php 
abstract class AchievementType
{
    public function name()
    {
        $class = (new ReflectionClass($this))->getShortName();
        return trim(preg_replace('/[A-Z]/', ' $0', $class));
    }

    public function icon()
    {
        return strtolower(str_replace(' ', '-', $this->name())).'.png';
    }

    // no body allowed, just defining a requirement for subclasses
    abstract public function qualifier($user);
}

class FirstThousandPoints extends AchievementType
{
    public function qualifier($user)
    {
        // qualifier logic
    }
}

class ReachTop50 extends AchievementType
{
// we can't accidentally forget, because an error would occur right away
    public function qualifier($user)
    {
        // qualifier logic
    }
} 
```

## Summary 
- **abstract classes** let us define a template or *base class*
- abstract classes can't be instantiated directly, they must be *extended* by subclasses
- abstract classes can still include behavior and functionality (unlike *interfaces*)
- abstract classes can declare **abstract methods** to define methods that are **required** to be implemented by subclasses
- abstract methods cannot be implemented directly (there is no function body, it needs a definition in the subclass)


---

[Episode 3: Inheritance < Previous ](objects.md) &nbsp; | &nbsp; [OOP in PHP](/oop/) &nbsp; | &nbsp; [Next > Episode 5: Handshakes and Interfaces](interface.md)
