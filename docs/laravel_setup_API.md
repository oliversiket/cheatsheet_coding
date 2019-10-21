# Creating an API

with Laravel

---

## Setup

We'll need to install the Laravel installer package:

```shell
composer global require laravel/installer
```

This will allow us to easily create new Laravel projects. You'll only need to run this once (or if you get a new computer).

---

## Creating a Project

To create a new Laravel project run:

```shell
laravel new blog-api
```

**Note**: `blog-api` is just the project name, you could choose anything.

We're going to use Vagrant. Laravel has a prebuilt box called **Homestead**.

In the newly created `blog-api` directory, run:

```shell
composer require laravel/homestead --dev
```

Next, we need to setup Homestead:

```shell
vendor/bin/homestead make
```

Next, change the second line of `Homestead.yaml`:

```
memory: 512
```

Finally, we can run:

```shell
vagrant up
```


---

# Meanwhile...

---

## What is Laravel?

- Created by Taylor Otwell: previously a .NET developer
- A modern PHP framework
- Written using modern best practices
- Minimal core: uses Composer packages where possible
- Built on [Symfony](https://symfony.com)
- Great ecosystem

---

## Laravel Features

- **Homestead** Vagrant configuration
- **Eloquent ORM**
- CLI tool **artisan**
- **Database migrations**
- Scheduling
- Job Queues
- Good documentation
- Active community: [Laracasts](https://laracasts.com)
- [Laravel Forge](https://forge.laravel.com): easily host Laravel apps

---

## Getting Started

Once Vagrant has finished loading visit `http://homestead.test` (or `http://localhost:8000` in Windows)

---
## Edit the .env file

```php
DB_CONNECTION = mysql
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_DATABASE = homestead
DB_USERNAME = homestead
DB_PASSWORD = secret
```

## Routing

We're already familiar with **routing** from when we did React. The concept is very similar in Laravel: we need to take a URL and use it to determine what we want to do.

First, let's add a route to handle `POST` events sent to `/articles`. This will point to code that handles creating a new article.

Add a `POST` route for `/articles` to `routes/api.php`:

```php
// use the post method
// when the use request /articles - don't need the forward slash
// which will call the store method of the Articles controller
$router->post("articles", "Articles@store");
```

---

## Controllers

In Laravel routes point to **controllers**. It is the controller's job to deal with the request and return a response.

Run the following inside your Vagrant box<sup>†</sup> to create an Articles controller:

```shell
artisan make:controller Articles --api
```

Laravel has added a `store` method to `Articles`:

```php
public function store(Request $request)
{
    // handle post request
}
```

<p class="footnote">† <i>All</i> commands should be run inside Vagrant</p>

---

## Database Migrations

We'll need to store the articles in a database so that the data is persisted.

Laravel makes it really easy to deal with database structuring using **database migrations**.

Database migrations are bits of code that tell the database how it should be structured. These are run by Laravel when we run the `artisan migrate` command. This means anyone with our codebase can easily get their database to have the right structure for the app.

### Creating a Model/Migration

On your Vagrant box run:

```shell
artisan make:model Article -m
```

This will create an Article model class (in `app/Article.php`) as well as a database migration (in the `database/migrations` directory).


---

## Database Migrations

We'll need to update the migration file to create the structure we'll need to store an article:

```php
public function up()
{
    Schema::create('articles', function (Blueprint $table) {
        $table->increments('id');
        $table->string("title", 100);
        $table->text("article");
        $table->timestamps();
    });
}
```

Finally, we need to run the migrations:

```shell
artisan migrate
```

We should now have an `articles` table in the database.

**Note**: if you make a mistake in your migration file, you can run `artisan migrate:rollback` to undo the last set of migrations that you ran.

---

## Models

The `Article` model **extends** the Eloquent ORM model. This allows us to use the model to interact with the data from the database.

For example, to add an article to the database we can use the `Article::create()` method. And to get all the articles out of the database we can use `Article::all()`.

Once we have an `Article` object we can access the data from the database. For example:

```php
$article->title; // would give us the article title
$article->title = "Blah blah blah"; // would set the article title
```

If you change the values of a model object, make sure you run `save()` on it:

```php
$article->save();
```

You can read more above models on the [Eloquent ORM Documentation](http://laravel.com/docs/5.6/eloquent#introduction).

---

## Creating

Now we've got somewhere to store the articles, we need to update our controller so that we can create an Article.

First, we need to tell the controller to use the `Article` model that we created using `artisan`:

```php
// make sure you add this near the top, undereath the namespace declaration
use App\Article;
```

Then we need to update our `store` function to create an article using the data from the request:

```php
public function store(Request $request)
{
    // get post request data for title and article
    $data = $request->only(["title", "article"]);

    // create article with data and store in DB
    $article = Article::create($data);

    // return the article along with a 201 status code
    return response($article, 201);
}
```

Now we can try doing a `POST` request with Postman.

---

## Mass Assignment Vulnerability

If we're not careful we might accidentally allow users to update fields that they should have access to. Laravel guards against this by default.

We can update the `Article` model to tell it which fields to expect.

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    // Only allow the title and article field to get updated via mass assignment
    protected $fillable = ["title", "article"];
}
```

Now try doing a `POST` request with Postman.

---

## Listing

Next let's setup the `GET` request to `/articles`.

First, add the route:

```php
$router->get("articles", "Articles@index");
```

And update the `index` method in the `Articles` controller:

```php
public function index()
{
    // get all the articles
    return Article::all();
}
```

That's all there is to it: Laravel does all the hard work for us once we've setup our model.

---

## Reading

Next we want to add a route for `GET` to something like `/articles/1`.

First, add a route:

```php
// we can group all our articles routes together
$router->group(["prefix" => "articles"], function ($router) {
    // ...previous routes...

    // {article} is a url parameter representing the id we want
    $router->get("{article}", "Articles@show");
});
```

And update the `show` method in `Articles`:

```php
// the id gets passed in for us
public function show($id)
{
    return Article::find($id);
}
```

---

## Editing

Next we'll add editing. We'll need a `PUT` route for `/articles/{article}`.

Add a route:

```php
$router->group(["prefix" => "articles"], function ($router) {
    // ...previous routes...

    $router->put("{article}", "Articles@update");
});
```

And update the `update` method in `Articles`:

```php
public function update(Request $request, $id)
{
    // find the current article
    $article = Article::find($id);

    // get the request data
    $data = $request->only(["title", "article"]);

    // update the article
    $article->fill($data)->save();

    // return the updated version
    return $article;
}
```

---

## Deleting

Finally, let's add a `DELETE` route.

Add a route:

```php
$router->group(["prefix" => "articles"], function ($router) {
    // ...
    $router->delete("{article}", "Articles@destroy");
});
```

And update the `destroy` method in `Articles`:

```php
public function destroy($id)
{
    $article = Article::find($id);
    $article->delete();

    // use a 204 code as there is no content in the response
    return response(null, 204);
}
```

---

## Key Terms

- **ORM**: Object Relational Mapper - allows us to access data from a database using standard objects
- **Controller**: a piece of code that is run for a specific route, whose job it is to get/update the relevant data and return a response to the user

---

## Additional Reading

- [Routing](https://laravel.com/docs/5.6/routing)
- [Controllers](http://laravel.com/docs/5.6/controllers)
- [Requests](https://laravel.com/docs/5.6/requests)
- [Responses](https://laravel.com/docs/5.6/responses)
- [Database Migrations](http://laravel.com/docs/5.6/migrations)
- [Eloquent](http://laravel.com/docs/5.6/eloquent)
