# Integrated

> Please note: while all the tests are returning green, this package is still early in development. If you use it and come across any quirks, please do open an issue or PR to let me know.

## Usage

### Step 1: Install

Install through Composer.

```
composer require laracasts/integrated --dev
```

### Step 2: Extend

Within a PHPUnit test class, extend either `Laracasts\Integrated\Extensions\Goutte` for general PHP applications, or `Laracasts\Integrated\Extensions\Laravel`, if you use Laravel.

```php
<?php // tests/ExampleTest.php

use Laracasts\Integrated\Extensions\Laravel as IntegrationTest;

class ExampleTest extends IntegrationTest {}
```

### Step 3: Write Some Integration Tests

That should mostly do it!

If you're using the Goutte extension, you'll of course need to boot up a server. By default, this package will assume a base url of "http://localhost:8888". Should need to modify this (likely the case), set a `$baseUrl` on your unit test class (or a parent class), like so:

```php
class ExampleTest extends IntegrationTest {
  protected $baseUrl = 'http://localhost:1234';
}
```

On the other hand, if you're using the Laravel-specific extension, no server needs to be running. You can ignore the base url. This also comes with the bonus of *very fast tests*!

Here are some examples to get you started:

```php
<?php

use Laracasts\Integrated\Extensions\Laravel as IntegrationTest;

class ExampleTest extends IntegrationTest
{

    /** @test */
   public function it_verifies_that_pages_load_properly()
   {
       $this->visit('/');
   }

    /** @test */
    public function it_verifies_the_current_page()
    {
        $this->visit('/some-page')
             ->seePageIs('some-page');
    }

    /** @test */
   public function it_follows_links()
   {
       $this->visit('/page-1')
            ->click('Follow Me')
            ->andSee('You are on Page 2')
            ->onPage('/page-2');
   }

    /** @test */
   public function it_submits_forms()
   {
       $this->visit('page-with-form')
            ->submitForm('Submit', ['title' => 'Foo Title'])
            ->andSee('You entered Foo Title')
            ->onPage('/page-with-form-results');

        // Another way to write it.
        $this->visit('page-with-form')
             ->type('Foo Title', '#title')
             ->press('Submit')
             ->see('You Entered Foo Title');
   }

    /** @test */
   public function it_verifies_information_in_the_database()
   {
       $this->visit('database-test')
            ->type('Testing', 'name')
            ->press('Save to Database')
            ->verifyInDatabase('things', ['name' => 'Testing']);
   }
}
```

### Step 4: Integration Methods

If you'd like to dig into the API examples from above a bit more, here is what each method call accomplishes.

#### `visit($uri)`

This will perform a `GET` request to the given `$uri`, while also triggering an assertion to guarantee that a 200 status code was returned.

```php
$this->visit('/page');
```

#### `see($text)`

To verify that the current page contains the given text, you'll want to use the `see` method.

```php
$this->visit('/page')
     ->see('Hello World');
```

> Tip: The word "and" may be prepended to any method call to help with readability. As such, if you wish, you may write: `$this->visit('/page')->andSee('Hello World');`.

#### `click($linkText)`

To simulate the behavior of clicking a link on the page, the `click` method is your friend.

```php
$this->visit('/page')
     ->click('Follow Me');
```

Behind the scenes, this package will determine that destination of the link (the "href"), and make a new "GET" request, accordingly.


#### `seePageIs($uri)` and `onPage($uri)`

In many situations, it can prove useful to make an assertion against the current url.

```php
$this->visit('/page')
     ->click('Follow Me')
     ->seePageIs('/next-page');
```

Alternatively, if it offers better readability, you may use the `onPage` method instead. Both are equivalent in functionality. This is especially true when it follows a `see` assertion call.

```php
$this->visit('/page')
     ->click('Follow Me')
     ->andSee('You are on the next page')
     ->onPage('/next-page');
```

#### `type($text, $selector)` or `fill($text, $selector)`

If you need to type something into an input field, one option is to use the `type` method, like so:

```php
$this->visit('search')
     ->type('Total Recall', '#q');
```

Simply provide the value for the input, and a CSS selector for us to hunt down the input that you're looking for. You may pass an id, element name, or an input with the given "name" attribute. The `fill` method is an alias that does the same thing.

#### `press($submitText)`

Not to be confused with `click`, the `press` method is used to submit a form with a submit button that has the given text.

```php
$this->visit('search')
     ->type('Total Recall', '#q')
     ->press('Search Now');
```

When called, this package will handle the process of submitting the form, and following any applicable redirects. This means, we could combine some of previous examples to form a full integration test.

```php
$this->visit('/search')
     ->type('Total Recall', '#q')
     ->press('Search Now')
     ->andSee('Search results for "Total Recall"')
     ->onPage('/search/results');
```

#### `submitForm($submitText, $formData)`

For situations where multiple form inputs must be filled out, you might choose to forego multiple `type()` calls, and instead use the `submitForm` method.

```php
$this->visit('/search')
     ->submitForm('Search Now', ['q' => 'Total Recall']);
```

This method offers a more compact option, which will both populate and submit the form.

Take special note of the second argument,  which is for the form data. You'll want to pass an associative array, where each key refers to the "name" of an input (not the element name, but the "name" attribute). As such, this test would satisfy the following form:

```html
<form method="POST" action="/search/results">
  <input type="text" name="q" placeholder="Search for something...">
  <input type="submit" value="Search Now">
</form>
```

#### `seeInDatabase($table, $data)` or `verifyInDatabase($table, $data)`

For situations when you want to peek inside the database to verify that a certain record/row exists, `seeInDatabase` or its alias `verifyInDatabase` will do the trick nicely.

```php
$data = ['description' => 'Finish documentation'];

$this->visit('/tasks')
     ->submitForm('Create Task', $data)
     ->verifyInDatabase('tasks', $data);
```

When calling `verifyInDatabase`, as the two arguments, provide the name of the table you're interested in, and an array of any attributes for the query.

**Important:** If using the Laravel-specific extension, this package will use your existing database configuration. There's nothing more for you to do. However, if using the Goutte extension for a general PHP project, you'll need to create a `integrated.json` file in your project root, and then specify your database connection string. Here's a couple examples:

** SQLite Config**

```js
{
    "pdo": {
        "connection": "sqlite:storage/database.sqlite",
        "username": "",
        "password": ""
    }
}
```

** MySQL Config**

```js
{
    "pdo": {
        "connection": "mysql:host=localhost;dbname=myDatabase"
        "username": "homestead",
        "password": "secret"
    }
}
```

### TestDummy

To help with RAD, this package includes the "laracasts/testdummy" package out of the box. For integration tests that hit a database, you'll likely want this anyways. Refer to the [TestDummy](https://github.com/laracasts/TestDummy) documentation for a full overview, but, in short, it gives you a very simple way to build up and/or persist your entities (like your Eloquent models), for the purposes of testing.

Think of it as your way of saying, "*Well, assuming that I have these records in my database table*, when I yadayada".

```php
use Laracasts\TestDummy\Factory as TestDummy;

// ...

/** @test */
function it_shows_posts()
{
  TestDummy::create('App\Post', ['title' => 'Example Post']);

  $this->visit('/posts')->andSee('Example Post');
}
```

### FAQ

#### Can I test JavaScript with this method?

No. For client-side interactions, you'll want to use something like Selenium.

