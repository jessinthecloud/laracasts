# Object-Oriented Principles in PHP - Laracast
> [Episode 1 : Classes](https://laracasts.com/series/object-oriented-principles-in-php/episodes/1)

## Classes

A class defines the general structure and behavior for a concept. It is like a blueprint or a template for the concept. As a beginner, the best way to determine your classes is to "find the nouns".

For instance, in project management, a noun would be `Project`. A project might have `Task`s, and those tasks might have `Comment`s on them. These are some of the nouns related to projects that we might use when creating classes, or blueprints, for a project management application. 

Blueprints don't contain any specifics. One `Project` could be about building a house, another could be about writing a book, and each one has a completely different set of details and people related to them.

### Example

Let's say we want to create achievement badges for users that they can earn as they participate with a system. `AchievementBadge` will be our noun. 

```php
/** Defining the blueprint, or template, for what an AchievementBadge is */
class AchievementBadge 
{
   
}
```

We want to plan the general structure and behavior of our `AchievementBadge`. It could have certain attributes, or **properties**:

- a title
- a description 
- points

```php
/** Defining the blueprint, or template, for what an AchievementBadge is */
class AchievementBadge 
{
    // properties of the class
    public $title;
    public $description; 
    public $points;
}
```
> *Prepending `public` to our properties lets us declare their visibilty and whether this property visible to the outside world, or to inherited instances, which we'll go over later.*

As we create **instances**, or objects, of this class, each one will have its own `title`, `description`, and `points`.

In addition to properties, classes can have behaviors. For example, maybe an `AchievementBadge` can be awarded to a user in the system. New behavior can be defined through class **methods** (functions specific to a class).

```php
/** Defining the blueprint, or template, for what an AchievementBadge is */
class AchievementBadge 
{
    // properties of the class
    public $title;
    public $description; 
    public $points;

    /**
     * Award this badge to a specific user
     */
    public function awardTo($user){
        // ... behavior code ...
    }
}
```

## Summary

- **classes** are a *blueprint* for a concept
- *"finding the nouns"* is a great way to figure out what your classes should be 
- **properties** are attributes or values related to the class
- **methods** are functions that define the behaviors of a class
- an *instance* of a class is called an **object**

---

[OOP in PHP](/oop/) &nbsp; | &nbsp; [Next > Episode 2: Objects](objects.md)



