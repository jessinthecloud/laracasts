# Object-Oriented Principles in PHP - Laracast
> [Episode 9 : Exceptions](https://laracasts.com/series/object-oriented-principles-in-php/episodes/9)

## Exceptions

If at any point your code encounters some abnormal condition that it doesn't know how to handle, you should **throw an *exception***. 

If it helps, think of it very literally; it's a way for your code to say "Hold up! I take *exception* with what you're trying to do, and I will bubble up this exception to my superiors until someone takes care of it (or responds to it)".

Let's start with the most basic of examples; adding two numbers.

```php
function add($one, $two)
{
    return $one + $two;
}

echo add(2, 2);
// output: 4
```

However, what if instead of `2`, we pass an array? 

```php 
echo add(2, []);
// ERROR, expected int but array given
```

We assumed that we would receive the right data, but we didn't and still tried to operator on it. We have a few options to handle this.

**Option 1**: Just... don't do that! Make sure you correct your data before making a call with it.

**Option 2**: Leverage types in the parameters, and the error will be caught sooner.

```php
function add(float $one, float $two)
{
    return $one + $two;
}

echo add(2, []);
// ERROR: Argument 2 must be type float
```

**Option 3**: Guard the data by checking it and throwing an exception if it isn't valid. 

```php
function add($one, $two){
    if(!is_float($one) || !is_float($two)){
        throw new Exception('Please provide a float');
    }
    return $one + $two;
}

echo add(2, []);
// ERROR: Uncaught Exception, Please provide float
```

However, notice that it says "Uncaught Exception". So hear's how to think of this; remember our example of "I take exception to this, and I will bubble up and alert my superiors". Well, if it bubbles up and *nobody* handles it, then the program doesn't know how to respond and it triggers an error.

We could wrap this in a `try` `catch` to define how we would handle the exception. We will `try` to echo out the result of 2 plus an array, but `ctach` an exception and handle it

```php
function add($one, $two){
    if(!is_float($one) || !is_float($two)){
        throw new Exception('Please provide a float');
    }
    return $one + $two;
}

// try the code inside
try {
    echo add(2, []);
} catch (Exception $e) {
    // catch any exceptions encountered and handle them
    echo "Oh Well.";
}
```

How we handle the exception depends on the application. We might perform a redirect, or prepare an error message to the user, or try to recover in a different way.

Even though we can trigger an `Exception` in our `add` function, think of this as the most generic version of an exception. Outside of the message, it doesn't really tell us what the exception was. 

There are many more specific [`Exception`](https://www.php.net/manual/en/spl.exceptions.php) types that we can reference here, in this case, `InvalidArgumentException`. It will still ultimately inherit from the `Exception` class. `InvalidArgumentException` is a type of `LogicException`, which itself extends the global generic `Exception` type.

The advantage here is that we better describe what went wrong. So now if we run the code with a more specific exception, we'll still catch it, because `InvalidArgumentException` is still an `Exception`.


```php
function add($one, $two){
    if(!is_float($one) || !is_float($two)){
        throw new InvalidArgumentException('Please provide a float');
    }
    return $one + $two;
}

// try the code inside
try {
    echo add(2, []);
} catch (Exception $e) {
    // catch any exceptions encountered and handle them
    echo "Oh Well.";
}
```

However, often you'll find in your code, especially if it's bubbling up multiple levels, that you don't want to catch a generic `Exception` because that could be any number of things. 

So if you want to handle a case where *this* specific thing occurred, but for *that* thing that occurred, we handle it in a different way, this is why you'd use specialized exception types.

```php
function add($one, $two){
    if(!is_float($one) || !is_float($two)){
        throw new InvalidArgumentException('Please provide a float');
    }
    return $one + $two;
}

// try the code inside
try {
    echo add(2, []);
} catch (InvalidArgumentException $e) {
    // catch any exceptions encountered and handle them
    echo "Oh Well.";
}
```

Let's do another example. Imagine we have a `User` class with a `download` method. What if the user is in a state in which they're not allowed to download a video? Would we throw an exception, or simply return false, or perform another action altogether?

The general rule is that if this condition is **something you might reasonably expect** as a result of calling the method, then it is **not** exceptional behavior. In which case, return false, or fire an event, or do whatever you need to.

In this case, is it reasonable to request a download if the user is not subscribed? This may be domain specific, in some situations, sure you can do that, in others, no you should never be able to download if you are not subscribed. So that *is* exceptional behavior.

```php
class User
{
    public function download(Video $video)
    {
        if(!$this->subscribed()){
            throw new \InvalidArgumentException("You must be subscribed to download videos.");
        }
    }

    public function subscribed()
    {
        return false;
    }
}
```

But it doesn't really make sense to throw an `InvalidArgumentException` again. We want to be specific, always ask yourself, "*why* am I throwing the exception?". The issue is not with the `Video`, but with the user not qualifying to download the video.

You might look through the [various built-in `Exceptions`](https://www.php.net/manual/en/spl.exceptions.php) PHP has to offer and find that the only one that makes sense is the top level `Exception`, that might be okay. It really depends on how important the requirement is.

```php
class User
{
    public function download(Video $video)
    {
        if(!$this->subscribed()){
            throw new \Exception("You must be subscribed to download videos.");
        }
    }

    public function subscribed()
    {
        return false;
    }
}

// unsubscribed user triggers an Exception
(new User())->download(new Video());
```

Higher up, our system might have a `UserDownloadsController` that tries the download and catches any exceptions.

```php
class User
{
    public function download(Video $video)
    {
        if(!$this->subscribed()){
            throw new \Exception("You must be subscribed to download videos.");
        }
    }

    public function subscribed()
    {
        return false;
    }
}

class UserDownloadsController
{
    public function show()
    {
        try{
            // unsubscribed user triggers an Exception
            (new User())->download(new Video());
        } catch {
            // handle the exception
        }
    }
}
```

Let's say you have a class `Team` and you're going to add a `Member` to it.

```php
class Member
{
    public $name;

    public function __construct($name)
    {
        $this->name = $name;
    }
}

class Team 
{
    protected $members = [];

    public function add(Member $member) 
    {
        $this->members []= $member;
    }

    public function members()
    {
        return $this->members;
    }
}

$team = new Team();
$team->add(new Member("Jane Doe"));
```

However, let's say we have a specific requirement that does not allow the team to have more than 3 members. Currently, we could run the `add` method as many times as we want, so we need to put some more guarding on the method.

```php 
class Team 
{
    protected $members = [];

    public function add(Member $member) 
    {
        // guard our method
        if(count($this->members) === 3){
            throw new Exception("You may not add more than 3 members.");
        }

        $this->members []= $member;
    }

    public function members()
    {
        return $this->members;
    }
}

$team = new Team(); // max of 3 members
$team->add(new Member("Jane Doe"));
$team->add(new Member("John Doe"));
$team->add(new Member("Alex Doe"));
// Exception thrown when adding Sam
$team->add(new Member("Sam Doe"));
```

Now we are correctly throwing an exception if the team tries to get too big. If the exception bubbles all the way up to the controller, you can then catch the exception and turn it into the proper response for the user. Maybe we would prepare a flash message and redirect back to the previous page to let the user know that they did something that wasn't allowed. 

Remember, in situations like this, we preferably want to protect before we even get to this point. Maybe the form doesn't even give the option to add a new team member if they're at the max. 

But there's always situations where they might get around it; they might manually submit a `POST` request, there might be other areas of the application where you call the method without proper validation, etc. We always want to protect ourselves at the lowest level.

### Custom Exception Types

You can imagine, if the exception bubbles all the way up to the controller from a form, there may have been any number of other generic `Exception`s that may have been thrown along the way. 

```php 
class Team 
{
    protected $members = [];

    public function add(Member $member) 
    {
        // guard our method
        if(count($this->members) === 3){
            throw new Exception("You may not add more than 3 members.");
        }

        $this->members []= $member;
    }

    public function members()
    {
        return $this->members;
    }
}

class TeamMemberController
{
    // add a new team memeber
    public function store()
    {
        $team = new Team(); // max of 3 members

        try{
            $team->add(new Member("Jane Doe"));
            $team->add(new Member("John Doe"));
            $team->add(new Member("Alex Doe"));
            // Exception thrown when adding Sam
            $team->add(new Member("Sam Doe"));    

        } catch(Exception $e) {
            // use method from the Exception class to return information
            echo $e->getMessage();
        }
    } 
}

// called via route / URI request
(new TeamMemberController())->store();
```
Right now, the code is simple enough that we immediately know what the exception is. But in a real application, other than the message itself, we often won't know what triggered the `Exception`. In situations like this, where we have an important, **clear** requirement being broken, in those conditions alone, you might consider creating a custom `Exception`. 

> We specify "in those conditions alone", because if you're not careful, you'll end up creating an `Exception` for every single thing that takes place, and it's just no necessary or providing much value.

When naming our custom `Exception` type, we want to consider why we are throwing the exception. In this cases, it is becuase we've reached our maximum members.

```php 
class MaximumMembersReached extends Exception 
{

}
```

> Whether or not you include the suffix "Exception" on your custom type is entirely up to your preference. i.e., `MaximumMembersReached` vs `MaximumMembersReachedException`

Now we can use our custom exception type, and catch our condition specifically when we've reached the maximum number of members for a team.

```php
class Team 
{
    protected $members = [];

    public function add(Member $member) 
    {
        // guard our method
        if(count($this->members) === 3){
            throw new MaximumMembersReached("You may not add more than 3 members.");
        }

        $this->members []= $member;
    }

    /* ... other methods ... */
}

class TeamMemberController
{
    // add a new team memeber
    public function store()
    {
        $team = new Team(); // max of 3 members

        try{
            $team->add(new Member("Jane Doe"));
            $team->add(new Member("John Doe"));
            $team->add(new Member("Alex Doe"));
            // Exception thrown when adding Sam
            $team->add(new Member("Sam Doe"));    

        } catch(MaximumMembersReached $e) {
            // use method from the Exception class to return information
            echo $e->getMessage();
        }
    } 
}
```

We can also have multiple catches, maybe we also want to catch generic `Exception`s, or maybe we want to catch a different `Exception` type related to team members and handle it differently. We can now drill down to the specific `Exception` that was thrown. 

```php 
class TeamMemberController
{
    // add a new team memeber
    public function store()
    {
        $team = new Team(); // max of 3 members

        try{
            $team->add(new Member("Jane Doe"));
            $team->add(new Member("John Doe"));
            $team->add(new Member("Alex Doe"));
            // Exception thrown when adding Sam
            $team->add(new Member("Sam Doe"));    

        } catch(MaximumMembersReached $e) {
            
            // use method from the Exception class to return information
            echo $e->getMessage();

        } catch(SpecialException $e){
            
            // do something special
            
        } catch(Exception $e) {
            
            // do something else
            
        } 
    } 
}
```

If you well be throwing an exception in multiple places, you likely do not want to repeat its message. To solve this, you can store the message on the class itself, or you can setup a static constructor on the custom `Exception` class.

```php 
class MaximumMembersReached extends Exception 
{
    // store message in the class
    protected $message = "You may not add more then 3 members.";
}

class Team 
{
    protected $members = [];

    public function add(Member $member) 
    {
        // guard our method
        if(count($this->members) === 3){
            throw new MaximumMembersReached;
        }

        $this->members []= $member;
    }

    /* ... other methods ... */
}

class TeamMemberController
{
    // add a new team memeber
    public function store()
    {
        $team = new Team(); // max of 3 members

        try{
            $team->add(new Member("Jane Doe"));
            $team->add(new Member("John Doe"));
            $team->add(new Member("Alex Doe"));
            // Exception thrown when adding Sam
            $team->add(new Member("Sam Doe"));    

        } catch(MaximumMembersReached $e) {
            // use method from the Exception class to return information
            echo $e->getMessage();
        }
    } 
}
```

`MaximumMembersReached` is specific enough that we can do this, but something more generic like a `LogicException` could never be hardcoded because it will be depedent on what took place to encounter the exception. 

If we don't want to be so specific, we could have instead created a `TeamException`. Now we can no longer hardcode the message, instead we could again pass the message in the moment the exception is thrown.

```php 
class TeamException extends Exception 
{

}

class Team 
{
    protected $members = [];

    public function add(Member $member) 
    {
        // guard our method
        if(count($this->members) === 3){
            throw new TeamException("You may not add more then 3 members.");
        }

        $this->members []= $member;
    }

    /* ... other methods ... */
}

class TeamMemberController
{
    // add a new team memeber
    public function store()
    {
        $team = new Team(); // max of 3 members

        try{
            $team->add(new Member("Jane Doe"));
            $team->add(new Member("John Doe"));
            $team->add(new Member("Alex Doe"));
            // Exception thrown when adding Sam
            $team->add(new Member("Sam Doe"));    

        } catch(TeamException $e) {
            // use method from the Exception class to return information
            echo $e->getMessage();
        }
    } 
}
```

Now we're back in the situation where we have to type out the message. Not a huge deal, but we can get around it by creating a static constructor. 

```php 
class TeamException extends Exception 
{
    // name indicates what caused the exception to get thsi message
    public static function fromTooManyMembers()
    {
        return new static("You may not add more then 3 members.");
    }
}

class Team 
{
    protected $members = [];

    public function add(Member $member) 
    {
        // guard our method
        if(count($this->members) === 3){
            // use static method related to what happened
            throw new TeamException::fromTooManyMembers();
        }

        $this->members []= $member;
    }

    /* ... other methods ... */
}

class TeamMemberController
{
    // add a new team memeber
    public function store()
    {
        $team = new Team(); // max of 3 members

        try{
            $team->add(new Member("Jane Doe"));
            $team->add(new Member("John Doe"));
            $team->add(new Member("Alex Doe"));
            // Exception thrown when adding Sam
            $team->add(new Member("Sam Doe"));    

        } catch(TeamException $e) {
            // use method from the Exception class to return information
            echo $e->getMessage();
        }
    } 
}
```

## Summary 

- If your code encounters an abnormal condition that it doesn't know how to handle, you should **throw an *exception***
- An **exception** a way for your code to say "Hold up! I take *exception* with what you're trying to do"
- You can wrap code in a `try`/`catch` block to handle the exception when it's thrown
- There are many [native PHP `Exception` types](https://www.php.net/manual/en/spl.exceptions.php) that we can use to **be more specific** about what issue was encountered
- We can also create **custom exception types** for important, specific requirements, by `extending` the `Exception`class
- Custom exceptions let us be more specific and catch only the exception we intend to, instead of a generic `Exception` that may have occurred on the way
- We can set our exception messages by **passing them when thrown**, storing them as a **property** of the class, or putting them into a **static constructor**

---

[Episode 8: Value Objects and Mutability < Previous ](valueobjects.md) &nbsp; | &nbsp; [OOP in PHP](/oop/)
