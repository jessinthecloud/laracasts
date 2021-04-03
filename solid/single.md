# SOLID Principles in PHP - Laracast
> [Episode 1 : Single Responsibility Principle](https://laracasts.com/series/solid-principles-in-php/episodes/1)

## Single Responsibility

***A class should have one, and only one, reason to change.***

In the following `SalesReporter` class, we have violated the single responsibility principle several times:

```php
namespace Acme\Reporting;

use Auth, DB, Exception;

class SalesReporter
{
    public function between($startDate, $endDate)
    {
        // perform authentication
        if( ! Auth::check()) throw new Exception('Authentication required for reporting.');

        // get sales from db
        $sales = $this->queryDBForSalesBetween($startDate, $endDate);

        // format and return the results
        return $this->format($sales);
    }

    protected function queryDBForSalesBetween($startDate, $endDate)
    {
        return DB::table('sales')->whereBetween('created_at', [$start_date, $endDate])->sum('charge') / 100;
    }

    protected function format($sales)
    {
       return "<h1>Sales: $sales</h1>";
    }
}
```

In order to use the `SalesReporter` (in Laravel) we would do the following:

```php 
Route::get('/', function(){
    $report = new Acme\Reporting\SalesReporter();

    $begin = Carbon\Carbon::now()->subDays(10);
    $end = Carbon\Carbon::now();

    return $report->between($begin, $end);
})
```

Admittedly, it does work, but it's not very conducive to change, which is what the single responsibility principle is about; an object should only have a single reason to change, a class should only have one job, or, ultimately, ***a class should have a single responsibility***. 

If a class is responsible for too many things, then it is doing too much and some of these actions should be extracted into their own classes.

### What's Wrong

What exactly is wrong with `SalesReporter`:

#### Authenticating

1. ***Why should `SalesReporter` have any interest at all in the authentication of a user?***

    ```php
    // perform authentication
    if( ! Auth::check()) throw new Exception('Authentication required for reporting.');
    ```

    This is application logic that doesn't belong here. Could we consume the `SalesReporter` class without having and authenticated user? Does it come from the web or some kind of service or API? 

    This isn't something for the `SalesReporter` to worry about, we can remove it entirely and instead we could extract it to the controller.

    ```php
    namespace Acme\Reporting;

    use Auth, DB, Exception;

    class SalesReporter
    {
        public function between($startDate, $endDate)
        {
            // removed authentication and put it into a controller instead
            
            // get sales from db
            $sales = $this->queryDBForSalesBetween($startDate, $endDate);

            // format and return the results
            return $this->format($sales);
        }

        protected function queryDBForSalesBetween($startDate, $endDate)
        {
            return DB::table('sales')->whereBetween('created_at', [$start_date, $endDate])->sum('charge') / 100;
        }

        protected function format($sales)
        {
           return "<h1>Sales: $sales</h1>";
        }
    }
    ```

#### Querying

2. ***We have a database query***

    ```php
    // get sales from db
    $sales = $this->queryDBForSalesBetween($startDate, $endDate);
    ```

    We are querying a table, fetching all rows that match a start and end date, getting the sum of those rows based on the charge, and then dividing by 100 to convert cents to dollars. Once again, `SalesReporter` has too many responsibilities, or reasons to change. 

    Another way to think of it is; there are too many consumers of this class. For example, if our persistence layer (database access) were to change in the future, we would have to update this class. It is not necessarily this class's responsibility to understand what our persistence layer is, or how to fetch this information.

    Instead, we can inject this through the constructor, perhaps with something like a `SalesRepository`. 

    > Ideally, we would actually typehint an Interface, maybe `SalesRepositoryInterface`, but we'll save that for a [future lesson]()

    We need to create the class, and probably change the method name to something more friendly:

    ```php
    // in the Acme\Repositiories\ folder
    namespace Acme\Repositories;

    use DB;

    // responsible for database specific interaction    
    class SalesRepository
    {
        protected function between($startDate, $endDate)
        {
            return DB::table('sales')->whereBetween('created_at', [$start_date, $endDate])->sum('charge') / 100;
        }
    }
    ```

    Now, inside our original `SalesReporter`, we can use the `SalesRepository` to query the db

    ```php
    namespace Acme\Reporting;

    // use new repo
    use Acme\Repositories\SalesRepository;
    use Auth, DB, Exception;

    class SalesReporter
    {
        private $repo;

        // inject the SalesRepository
        public function __construct(SalesRepository $repo)
        {
            $this->repo = $repo;
        }

        public function between($startDate, $endDate)
        {
            // get sales from db using repo class
            $sales = $this->repo->between($startDate, $endDate);

            // format and return the results
            return $this->format($sales);
        }

        protected function format($sales)
        {
           return "<h1>Sales: $sales</h1>";
        }
    }
    ```

    Now `SalesReporter` has one less responsibility, as a result it's easier to test and is more maintainable.

#### Formatting / Output

3. ***We have a `format()` method returning output.***

    ```php
    protected function format($sales)
    {
       return "<h1>Sales: $sales</h1>";
    }
    ```

    Again, why should `SalesReporter` care, or why should it be this class's responsibility to output, format, or print the results? Is it possible that we will want the output in a different format? We are assuming HTML, but what if we want JSON instead? Any time we want to change that, this class would have to be updated.

    One option would be to leave the formatting to the consumer of this class, in which case we would simply return the data.

    If you do want class-based formatting, how could we allow for that while still giving flexibility to the formatting? How do we know which format will be desired up front?

    Another option, is to extract the `format()` method and use an **interface**. Let's create an interface to describe how something can be formatted or printed for the user.

    ```php
    namespace Acme\Reporting;

    interface SalesOutputInterface
    {
        public function output();
    }
    ```

    And that's it! Just a simple contract that any implementation has to adhere to. Let's create our first implementation.

    ```php
    namespace Acme\Reporting;

    use Acme\Reporting\SalesOutputInterface;

    class HtmlOutput implements SalesOutputInterface
    {
        public function output($sales)
        {
            return "<h1>Sales: $sales</h1>";
        }
    }
    ```

    We have to also allow the `output()` method in the interface to accept the `$sales` parameter.

    ```php
    namespace Acme\Reporting;

    interface SalesOutputInterface
    {
        public function output($sales);
    }
    ```

    Now if we want to format it in a different way, simply create a new implementation (`JsonOutput implements SalesOutputInterface`, etc).

    In `SalesReporter`, we could add another helper method that accepts an implementation of the `SalesOutputInterface`, or add an argument to our existing method `between()`.

    ```php
    namespace Acme\Reporting;

    // use new repo
    use Acme\Repositories\SalesRepository;
    use Auth, DB, Exception;

    class SalesReporter
    {
        private $repo;

        // inject the SalesRepository
        public function __construct(SalesRepository $repo)
        {
            $this->repo = $repo;
        }

        public function between($startDate, $endDate, SalesOutputInterface $formatter)
        {
            // get sales from db using repo class
            $sales = $this->repo->between($startDate, $endDate);

            // format with the formatter provided and return the results
            return $formatter->output($sales);
        }
    }
    ```

    We can pass any implementation of `SalesOutputInterface` as a formatter, and because we know that any object provided will adhere to the contract of the interface and have a `format()` method we can use.

    Now `SalesReporter` doesn't care how we format the data, all it cares about is that no matter what implementation we use, we adhere to the contract and successfully call an `output()` method.

---


Notice how much cleaner this class is; it has a **single responsibility**.

If we were to go back to our routes file, in order to use the `SalesReporter` (in Laravel) we need to:

1. Pass in the `SalesRepository`
2. Pass in the `HtmlOutput` formatter

```php 
Route::get('/', function(){
    $report = new Acme\Reporting\SalesReporter(new \Acme\Repositories\SalesRepository());

    $begin = Carbon\Carbon::now()->subDays(10);
    $end = Carbon\Carbon::now();

    return $report->between($begin, $end, new Acme\Reporting\HtmlOutput());
})
```

Now if we want to change the output, all we need to do is swap out the formatter provided with another implementation (after creating the new implementation class)

```php
return $report->between($begin, $end, new Acme\Reporting\JsonOutput());
```

## Summary

- A class should only have one job
- Makes classes easier to maintain 
- Makes classes easier to test

---

[SOLID Principles in PHP](/solid/) &nbsp; | &nbsp; [Next > Episode 2: Open-Closed](openclose.md)
