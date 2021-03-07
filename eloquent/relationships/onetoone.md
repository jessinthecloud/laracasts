# Eloquent Relationships - Laracast
> [Episode 1 : One to One](https://laracasts.com/series/eloquent-relationships/episodes/1)

## One to One

A user might have **many** blog posts, but what is something they have **one** of? A user might have one name, one address, or one profile. 

Laravel and Eloquent include all of these base relationships as readable methods.

```php
class User {
    
    public function profile() 
    {
        $this->hasOne(Profile::class);
    }    
}
```

`Profile` doesn't exist yet, so let's create it and include the migration with `-m`

```sh 
php artisan make:model Profile -m
```

> In Laravel 8, you can find the Model in your /app/Models/Profile.php and migration in /database/migrations/

Let's build up the user's profile

```php 
// ...profiles table migration

/**
 * Run the migrations.
 *
 * @return void
 */
public function up()
{
    Schema::create('profiles', function (Blueprint $table) {
        $table->id();
        $table->string('website_url');
        $table->string('github_url');
        $table->string('twitter_url');
        $table->timestamps();
    });
}    
```

You may be thinking, why not put this on the `users` table instead? The answer is; maybe. Consider whether you are getting enough bang for your buck by moving these columns to their own table, or do they make more sense directly tied to the user?

Now that we have a new `profiles` table, we still don't have a link between the `Profile` and the `User`, this is what basic foreign keys are for. Let's setup the foreign key to the `User` table and then run the migration

```php 
// ...profiles table migration

/**
 * Run the migrations.
 *
 * @return void
 */
public function up()
{
    Schema::create('profiles', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id');
        $table->string('website_url');
        $table->string('github_url');
        $table->string('twitter_url');
        $table->timestamps();
    });
}    
```

```sh
# run the migration
php artisan migrate
```

Now we have a `users` and a `profiles` table. You can enter some test data to associate a user with a profile.

Using basic SQL, you can fetch all users, all profiles, or the profile associated to a user, and we can use Eloquent to reproduce these queries.

```sql
SELECT * FROM users; # get all users
SELECT * FROM profiles; # get all profiles
SELECT * FROM profiles WHERE user_id=1; # get profile for user 1
```

When we set up relationships between tables & Models like this, eloquent will allow you to dynamically access, in this case the `Profile`, as if it were a property. Which means we can say "give me the user profile" (`$user->profile`), and instantly it will be as if we have an instance of that given `Profile`.

```php
class User {
    
    public function profile() 
    {
        $this->hasOne(Profile::class);
    }    
}

// is like getting a new Profile() with its data filled out for this user
$profile = $user->profile;
```

We can then access the related model's methods and data 

```php 
$github = $user->profile->github_url;
```

When we are calling `hasOne`, it is dictating what foreign key (id) we are using and what SQL query will be run behind the scenes. Laravel and eloquent will run the query, fill out the attributes and then make them available to you. 

If you want to see this in better detail, we can use the `DB` Facade in tinker to see what Laravel is doing behind the scenes for us.

```sh 
php artisan tinker 
```
```php
DB::listen(function($sql) { var_dump($sql->sql, $sql->bindings); });

$user = App\Models\User::first();

$user->profile;

```

What `$user->profile` is really saying is 

```sql 
SELECT * FROM profiles WHERE profiles.user_id = ? and profiles.user_id IS NOT NULL LIMIT 1;
```

And filling in the current user for the `user_id`

We could also see what Laravel is doing another way

```sh
php artisan tinker 
```
```php
DB::enableQueryLog();

$user->App\Models\User::first();

$user->profile;
```

Tinker isn't printing out the info for us right now, but we know that we've run two queries, so let's get the query log.

```php 
DB::getQueryLog();

[
    // get one user 
    [
        "query" => "SELECT * FROM users LIMIT 1",
        "bindings" => [],
        "time" => 2.08
    ],
    // get profile associated with user
    [
        "query" => "SELECT * FROM profiles WHERE profiles.user_id = ? and profiles.user_id IS NOT NULL LIMIT 1",
        "bindings" => [
            1,
        ],
        "time" => 0.62
    ]
]
```

Now we can see an array of all the queries we made. If you're ever getting confused in this series, just enable the query log, run the code, and check out what is happening in the SQL.

Let's do one more example. Users can gain experience through interactions, so let's add this method and model.

```sh 
php artisan make:model Experience -m
```
```php 
class User {
    
    public function experience() 
    {
        $this->hasOne(Experience::class);
    }    
}

// ...experience migration
public function up()
{
    Schema::create('experiences', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id');
        $table->unsignedInteger('points');
        $table->timestamps();
    });
}
```

```php 
$user = App\Models\User::first();

$user->experience->points;
```

Let's view these queries

```sql 
SELECT * FROM experiences WHERE expereiences.user_id = ? AND experiences.user_id IS NOT NULL LIMIT 1;
```

This is fine, but if points is the only thing we'll ever have in the experiences table, we may want to reconsider and simply add it as a column on the `users` table.

```php 
// ...users migration 
 public function up()
{
    Schema::create('users', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->string('email')->unique();
        $table->timestamp('email_verified_at')->nullable();
        $table->string('password');
        $table->unsignedInteger('experience_points');
        $table->rememberToken();
        $table->timestamps();
    });
}
```

Now we can store experience points directly on the `User`.


---

[Eloquent Relationships](/eloquent/relationships/) &nbsp; | &nbsp; [Next > Episode 2: One to Many](onetomany.md)
