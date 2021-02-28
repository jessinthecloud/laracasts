# Object-Oriented Principles in PHP - Laracast
> [Episode 6 : Encapsulation](https://laracasts.com/series/object-oriented-principles-in-php/episodes/6)

## Encapsulation

> To enclose within a capsule

The **restriction of access** to an object's internals (hiding). Imagine yourself as a person in the outside world; there is certain information that you expose to the world, but also information that you would like to keep private.

```php 
class Person
{
    public $name;

    public function __construct($name)
    {
        $this->name = $name;
    }

    // something we share with everyone 
    public function favoriteTeam()
    {
        return "NY Liberty";
    }

    // not something we want to share
    public function innerTurmoil()
    {
        return "I'm terrified because we're all going to die someday.";
    }
}

$alex = new Person('Alex');

echo $alex->favoriteTeam();
// output: NY Liberty

// everything in the class is declared public, 
// so this is also exposed
echo $alex->innerTurmoil();
// output: I'm terrified because we're all going to die someday.
```

We can change visibility by changing the keywords in front of our properties and methods (`public`, `protected`, `private`). If you don't set any visibility, it will default to public.

```php 
class Person
{
    public $name;

    /* ... public methods ... */

    // not something we want to share
    private function innerTurmoil()
    {
        return "I'm terrified because we're all going to die someday.";
    }
}

$alex = new Person('Alex');

echo $alex->innerTurmoil();
// output: ERROR
```

When the `innerTurmoil` method is set to `private`, calling it from outside of the class will cause an error. 

Even though we can "restrict" access using keywords, there are still ways to get around these settings; it is not a security measure, but indicating to the rest of the code that it doesn't need to worry about these behaviors or values.

> Native to PHP, the `ReflectionMethod` class reports information about a method. We can use it to access private methods of another class -- [Learn More](https://www.php.net/manual/en/class.reflectionmethod.php)

```php
// we can still access the method using Reflection classes
$method = new \ReflectionMethod(Person::class, 'innerTurmoil');
$method->setAccessible(true);

$alex = new Person('Alex');

$method->invoke($alex);
```

One of the purposes of encapsulation to **signal** to the outside world that an **object's internal state should not be changed**. In other words, we should never be able to force an object into an invalid state. For example, because `$name` is `public`, anyone could change `$alex`'s name to whatever they want, including `null`. 

When you hide internals by assigning visibility, you are **protecting the integrity** of the object.

> Most developers will default their methods to private or protected

As an additional indicator, if a method is public specifically for use externally, many developers will add a tag to better signal that these methods are "safe" to use outside of the class.

```php 
class TennisMatch
{
    /** @api */
    public function score()
    {
        // is there a winner
        // does someone have advantage
        // are they in deuce
        // etc
    }

    protected function hasAdvantage()
    {
        
    }
}
```

There are situations in which a property may be `protected`, but you still need to access it. In situations like this, you can add a *getter*, or accessor, method that will simply return the internal state that you need.

```php 
class TennisMatch
{
    protected $playerOne;

    // getter for $playerOne
    public function playerOne()
    {
        return $this->playerOne;
    }
}
```

> **Note**: Some developers are of the opinion that getters should never be used and are indicative of bad code. However, sometime it's impractical to avoid them. 
> 
> Just be wary when using getters as it is possible to break encapsulation if you're not careful.

Let's look at one more example:

```php 
namespace Laracats\Teams;

use Mail;
use Laracasts\Teams\Team;
use Illuminate\Database\Eloquent\Model;
use Laracasts\Mail\teams\MemberAddedToTeam;

class Invitation extands Model
{
    protected $fillable = ['code', 'canceled', 'team_id', 'recipient'];

    public static function generate(Team $team)
    {
        return new static([
            'code' => 'foo-code',
            'team_id' => $team->id
        ]);
    }

    public function cancel()
    {
        // logic to cancel invite
    }

    public function team()
    {
        // logic to get a team
        
    }

    public function sendTo($recipient)
    {
        // logic to send invite
    }

    protected function deletePastInvitationTo($recipient)
    {
        // logic to delete invite
    }

    protected function email()
    {
        // logic to send email
    }
}
```

The methods `deletePastInvitationTo` and `email` are `protected` because they're only relevant to things that the class needs to do its job, but not things the outside world would need to use or be concerned with. For example, we don't want others to be calling `email` directly, that is what `sendTo` is for.


## Summary

- **Encapsulation** is the restriction, or hiding, of access to an objects properties and methods from outside of its class
- The purpose of encapsulation is to allow an object ot protect its integriy and signal to the outside world
- The keywords `public`, `protected`, and `private` let us set access, or visibility
- Setting visibility is a signal to the outside world as to which values should not be changed
- "Getters" are public methods that can be used to access `private` or `protected` properties, but use them with caution

| Keyword | Behavior |
|:--------|:---------|
| public | open to the world to get/set/use<BR>is the default when no visibility is set |
| protected | cannot be accessed directly from outside of the class<BR> can be accessed by subclasses <BR> a signal to others that it probably should not be changed |
| private | cannot be accessed directly from outside of the class <BR> a signal to others that it should not be changed |

---

[Episode 5: Handshakes and Interfaces < Previous](interface.md) &nbsp; | &nbsp; [OOP in PHP](/oop/) &nbsp; | &nbsp; [Next > Episode 7: Object Composition and Abstractions](composition.md)
