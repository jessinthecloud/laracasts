# Eloquent Relationships - Laracast
> [Episode 4 : Has Many Through](https://laracasts.com/series/eloquent-relationships/episodes/4)

## Has Many Through
This allows us to access distant associations. After we add `affiliation_id` to the `users` table:

```php
class User 
{
    // user has many posts
    // post has one (belongs to)  user 
    public function posts() 
    {
        return $this->hasMany(Post::class);
    }    

    public function affiliation()
    {
        return $this->belongsTo(Affiliation::class);
    }
}
class Affiliation 
{
    public function posts()
    {
        // has many posts through the user table
        return $this->hasManyThrough(Post::class, User::class);
    }
}
```

An affiliation has many posts *through* the `users` table


---

[Previous < Episode 3: Many to Many](manytomany.md) &nbsp; | &nbsp; [Eloquent Relationships](/eloquent/relationships/) &nbsp; | &nbsp; [Next > Episode 5: Polymorphic Relations](polymorphic.md)
