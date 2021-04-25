# Eloquent Relationships - Laracast
> [Episode 2 : One to Many](https://laracasts.com/series/eloquent-relationships/episodes/2)

## One to Many

A user can have many: blog posts, jobs, achievements. 
A post can have many: comments.
A project can have many: tasks.
etc.

All of these are examples of one-to-many relationships.

If a user `hasMany` posts, then a single post `belongsTo` a user.

```php 
class User 
{
    // user has many posts
    // post has one (belongs to)  user 
    public function posts() 
    {
        $this->hasMany(Post::class);
    }    
}

class Post 
{
    // user has many posts
    // post has one (belongsto) user
    public function user() 
    {
        $this->belongsTo(User::class);
    }    
}
```

If the table foreign key is not `tablesingular_id`, you can override the foreign key Eloquent will look for

```php
class Post {
    // if posts table has author_id instead of user_id as key to users table
    public function user() 
    {
        $this->belongsTo(User::class, 'author_id');
    }    
}
```
```


---

[Previous < Episode 1: One to One](onetoone.md) &nbsp; | &nbsp; [Eloquent Relationships](/eloquent/relationships/) &nbsp; | &nbsp; [Next > Episode 3: Many to Many](manytomany.md)
