# Object-Oriented Principles in PHP - Laracast
> [Episode 2 : Objects](https://laracasts.com/series/object-oriented-principles-in-php/episodes/2)

## Objects

Let's say we have a class for managing teams. When describing a team, we might say that each one:

- has a name 
- has members
- can add a member 
- can be canceled
- has a manager

```php 
class Team 
{
    protected $name;
    protected $members = []; // an empty array to store the members

    // for getting the team's name
    public function name() { }

    // for getting the team's members
    public function members() { }

    public function add() { }

    public function cancel() { }

    public function manager() { }
}
```

Now we know the basic behavior for any team we create in our system, and we can use this blueprint to construct our `Team` *instances*, which we'll call **objects**. 

We can create a new `Team` object and pass it a `$name` in the constructor arguments.

> *A **constructor** is a special method that is called automatically whenever the class is **instantiated** (when an object is created from the class). This method is named `__construct`. We need to define it to take a name first, so let's do that*

```php 
class Team 
{
    protected $name;
    protected $members = []; // an empty array to store the members

    public function __construct($name)
    {
        $this->name = $name;
    }

    /* ... our other methods ... */
}
```

>*`$this->name` in the constructor is setting the `$name` property of "this" object being created.*

> *`$this` is a [pseudo-variable](https://www.php.net/manual/en/language.oop5.basic.php) that represents the current object. When referencing properties and methods inside the class, you need to use `$this` and the `->` object operator: `$this->property` for properties, or `$this->method()` for methods.*

> *If you are referencing in `static` context, you would use `self::` instead of `$this->`, but don't worry about that for now.*

Now we can create the `Team` object and inject the `$name` into the constructor.

```php 
$acme = new Team('Acme');
```

In case you haven't noticed, `$name` is a `protected` property, so it is not visible from outside of the class. This means that if we want to access the `$name` of the `Team` object, we'll need to use the `name()` method. This kind of method is called an **accessor** or a **getter**. `$members` is also a protected property, so let's setup its method as well.

```php 
class Team 
{
    protected $name;
    protected $members = [];

    public function __construct($name)
    {
        $this->name = $name;
    }

    // for getting the team's name
    public function name()
    {
        return $this->name;
    }

    // for getting the team's members
    public function members()
    {
        return $this->members;
    }

    /* ... our other methods ... */
}
```

Now we can access the name of our `Team`:

```php 
$acme = new Team('Acme');

echo "The team's name is ".$acme->name();
```

> If the `$name` property of `Team` were `public` instead, we could acces it directly with `$acme->name;`


Our team has no members right now, so let's add some, and first let's make sure our `add()` method is going to do this for us.

```php 
class Team 
{
    protected $name;
    protected $members = [];

    public function __construct($name)
    {
        $this->name = $name;
    }

    /* ... our getter methods ... */

    public function add($name)
    {
        $this->members[] = $name;
    }

    /* ... our other methods ... */
}
```

Now we can add team members and retrieve the list of members from the `Team`.

```php 
// create a new team named "Acme"
$acme = new Team('Acme');

// add new team members
$acme->add('John Doe');
$acme->add('Jane Doe');

// get the current team members
var_dump($acme->members());

/* will output:

array(2) {
    [0]=>
    string(8) "John Doe"
    [1]=>
    string(8) "Jane Doe"
} 
*/
```

Although... It would be easier to add members to a `Team` when it's created, like we are able to define its name. Let's update the constructor so that we can pass through the initial members when we instantiate `Team`. 

We can let the members be optional when creating a team by assigning it a value in the parameters for the constructor.

```php 
class Team 
{
    protected $name;
    protected $members = [];

    public function __construct($name, $members = [])
    {
        $this->name = $name;
        $this->members = $members;
    }

    /* ... our other methods ... */
}
```

Now when we create a `Team` we can begin with some members

```php 
// create a new team named "Acme", with team members
$acme = new Team('Acme', [
    'John Doe',
    'Jane Doe'
]);

// still returns the team members
var_dump($acme->members());

/* will output:

array(2) {
    [0]=>
    string(8) "John Doe"
    [1]=>
    string(8) "Jane Doe"
} 
*/

// We can still add members
$acme->add('Alex Doe');

var_dump($acme->members());

/* will output:

array(2) {
    [0]=>
    string(8) "John Doe"
    [1]=>
    string(8) "Jane Doe"
    [2]=>
    string(8) "Alex Doe"
} 
*/
```

### Static

We've been instantiating our `Team` using the `new` keyword, but what about the `static` keyword I mentioned earlier? 

A `static` constructor allows us to write code that may be more in line with how we would naturally refer to concepts when speaking about them. For instance, we could `start` a new `Team`. Let's create a static constructor called `start`.

> *`public static` methods are essentially global; they're **accessible anywhere in the system**. When you call a static method that changes some part of your system, be cautious.*

```php 
class Team 
{
    protected $name;
    protected $members = [];

    public function __construct($name, $members = [])
    {
        $this->name = $name;
        $this->members = $members;
    }

    /**
     * create a new instance of the class
     * otherwise, is doing the same as constructor
     *
     * public static methods are global
     *
     * @return object   static instance of the class
     */
    public static function start($name, $members = [])
    {
        // returns a new instance of this class
        return new static($name, $members);
    }

    /* ... our other methods ... */
}
```

Now we can instantiate our class in a way that may be more readable

```php 
// create a new team named "Acme", with team members
$acme = Team::start('Acme', [
    'John Doe',
    'Jane Doe'
]);
```

### Refactor

We're starting to repeat ourselves a bit in the code. `__construct` and `start` both have the same parameters, and they should always match that way. If we want to pass another value through `start` to the constructor, we'll have to remember to add the parameter in both places. 

Ideally, we want the original `__construct` to be the single source instead. Let's refactor to keep our code a little more [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)

```php 
class Team 
{
    protected $name;
    protected $members = [];

    public function __construct($name, $members = [])
    {
        $this->name = $name;
        $this->members = $members;
    }

    /**
     * create a new instance of the class
     * otherwise, is doing the same as constructor
     *
     * public static methods are global
     *
     * @param  ...$params   accept a variable number of arguments 
     *
     * @return object       static instance of the class
     */
    public static function start(...$params)
    {
        // unpacks the $params array into its values to pass through to the constructor
        return new static(...$params);
    }

    /* ... our other methods ... */
}
```

The code `start(...$params)` will accept any number of arguments and pass those through to the constructor. Now if more parameters are defined, we don't have to worry about updating both methods.

> `...`, or the "[splat operator](https://stackoverflow.com/questions/41124015/meaning-of-three-dot-in-php)", allows for a variable number of function parameters. All the arguments passed to the `start` method will be "packed" into the `$params` array. 
>
> Using the splat operator again when passing `$params` to the static constructor means they are "unpacked" into their separate values inside of it.

```php 
$acme = Team::start("Acme", ['Jane', 'John']);

// inside the start method, "Acme" and ['Jane', 'John'] are packed into the $params array:
//      it's like sending ["Acme", ['Jane', 'John']]

// we pass $params to static(...$params); which then unpacks it
//      it's like calling static("Acme", ['Jane', 'John']);

```

We've been using strings to represent team members, but what happens when we want to know more about a member? When did they sign up? What's their e-mail address? etc. We can't represent all that information in a single string. 

Member is another noun related to `Team`, so let's pull it out into its own class.

```php 
class Member
{
    protected $name;

    public function __construct($name)
    {
        $this->name = $name;
    }
}
```

Now we have a place to store logic and information about a member, and we can use `Member` when creating a `Team`.

```php 
// create a new team named "Acme", with team members
$acme = Team::start('Acme', [
    new Member('John Doe'),
    new Member('Jane Doe')
]);

// still returns the team members, but now they're objects
var_dump($acme->members());

/* will output:

array(2) {
    [0]=>
    object(Member)#1 (1) {
        ["name":protected]=>
        string(8) "John Doe"
    }
    [1]=>
    object(Member)#2 (1) {
        ["name":protected]=>
        string(8) "Jane Doe"
    }
} 
*/

```

## Summary

- if a class is like a blueprint, then an **object** is an **implementation of that blueprint**
- if you need access to `protected` properties, you can use methods call **accessors** or **getters** to return their value
- **`static`** constructors create a nice readable way to make a new object

---

[Episode 1: Classes < Previous ](classes.md) &nbsp; | &nbsp; [OOP in PHP](/oop/) &nbsp; | &nbsp; [Next > Episode 3: Inheritance](inheritance.md)
