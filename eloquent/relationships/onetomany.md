# Eloquent Relationships - Laracast
> [Episode 2 : One to Many](https://laracasts.com/series/eloquent-relationships/episodes/2)

## One to Many

A user can have many: blog posts, jobs, achievements. 
A post can have many: comments.
A project can have many: tasks.
etc.

All of these are examples of one-to-many relationships.

```php 
class User {
    
    public function posts() 
    {
        $this->hasMany(Post::class);
    }    
}
```

If a user `hasMany` posts, then a single post `belongsTo` a user.

---

[Previous < Episode 1: One to One](onetoone.md) &nbsp; | &nbsp; [Eloquent Relationships](/eloquent/relationships/) &nbsp; | &nbsp; [Next > Episode 3: Many to Many](manytomany.md)
