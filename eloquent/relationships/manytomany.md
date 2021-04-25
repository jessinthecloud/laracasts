# Eloquent Relationships - Laracast
> [Episode 3 : Many to Many](https://laracasts.com/series/eloquent-relationships/episodes/3)

## Many to Many

### Inverse of One to Many

A post has many comments, and a comment can only belong to one post. But a post can also have tags, and those tags can also belong to other posts.

Many posts belong to many tags, and many tags belong to many posts.

```php
class Post 
{
    // one to many
    public function comments() 
    {
        $this->hasMany(Comment::class);
    }

    // many to many
    public function tags()
    {
        return $this->belongsToMany(Tag::class);
    }    
}

class Tag
{
    public function posts()
    {
        return $this->belongsToMany(Post::class);
    }
}
```

### Database - Pivot Table
Along with the `posts` table and `tags` table, the table that will connect the two is called a ***pivot table***. The pivt table here will be `post_tag`

> Pivot table's naming convention is to concatenate the singular name of the two tables with an underscore in alphabetical order

> To custom name your pivot table, pass the name in the relationship assignment method in each class
> `$this->belongsToMany(Tag::class, 'custom_name');`

The pivot table will consist of columns to store the `id`s from each table.

```php
/**
 * pivot table to map together the posts and tags tables
 */
Schema::create('post_tag', function (Blueprint $table) {
    $table->id();

    // references id on posts table, delete this record if post is deleted
    $table->foreignId('post_id')->constrained()->onDelete('cascade'); 
    
    // references id on tags table, delete this record if post is deleted
    $table->foreignId('tag_id')->constrained()->onDelete('cascade'); 
    $table->timestamps();
});
```

Let's get all posts related to a tag

```php
// after data is seeded into the table
$post = App\Models\Post::find(1);
// get all tags on this post
$tags = $post->tags; // returns Eloquent Collection
```

In reverse, let's get all posts related with a tag

```php
// after data is seeded into the table
$tag = App\Models\Tag::find(1);
// get all posts related to this tag
$posts = $tag->posts; // returns Eloquent Collection
```

## Relating specific table records 
To add a new tag to a post, we want to `attach()` it. This will insert the corresponding values into the pivot table `post_tag`

```php
$post = App\Models\Post::find(1);
// add via tag id
$post->tags()->attach(2);
```

The `post_tag` table will look like 

| id | post_id | tag_id |
|---|----|----|
| 1 | 1 | 2 |

You can do the same in reverse

```php
$tag = App\Models\Tag::first();
// add via tag id
$tag->posts()->attach(2);
```

| id | post_id | tag_id |
|---|----|----|
| 1 | 1 | 2 |
| 2 | 2 | 1 |

### Remove
To remove, use `detach()`
```php
$post = App\Models\Post::first();
// remove via tag id
$post->tags()->detach(2);
```

### Add multiple
To attach multiple, send an array of ids
```php
$post = App\Models\Post::first();
// add via tag id
$post->tags()->attach([2,5,7]);
```

### Add existing `Model` object
To attach an existing `Model` instance, pass the instance
```php
$post = App\Models\Post::first();
$tag = App\Models\Tag::first();
// add via existing model
$post->tags()->attach($tag);
```

### Timestamps
If you want timestamps to be populated on your pivot table, opt in `withTimestamps()`
```php
class Tag
{
    public function posts()
    {
        return $this->belongsToMany(Post::class)->withTimestamps();
    }
}
```

### Avoiding Duplicates
If we try to add the same data many times in a row, we can prevent duplicate records inserted by

1. Setting the pivot table primary key to a composite key
    ```php
    /**
     * pivot table to map together the posts and tags tables
     */
    Schema::create('post_tag', function (Blueprint $table) {
        // composite primary key
        $table->primary(['post_id', 'tag_id']);
        // references id on posts table, delete this record if post is deleted
        $table->foreignId('post_id')->constrained()->onDelete('cascade'); 
        // references id on tags table, delete this record if post is deleted
        $table->foreignId('tag_id')->constrained()->onDelete('cascade'); 
        $table->timestamps();
    });
    ```
2. Set a composite unique index
    ```php
    /**
     * pivot table to map together the posts and tags tables
     */
    Schema::create('post_tag', function (Blueprint $table) {
        $table->id();
        // references id on posts table, delete this record if post is deleted
        $table->foreignId('post_id')->constrained()->onDelete('cascade'); 
        // references id on tags table, delete this record if post is deleted
        $table->foreignId('tag_id')->constrained()->onDelete('cascade'); 
        $table->timestamps();

        // the combo of these 2 columns MUST be unique
        $table->unique(['post_id', 'tag_id']);
    });
    ```

Now an exception would be thrown on a duplicate insert attempt



---

[Previous < Episode 2: One to Many](onetomany.md) &nbsp; | &nbsp; [Eloquent Relationships](/eloquent/relationships/) &nbsp; | &nbsp; [Next > Episode 4: Has Many Through](hasmany.md)
