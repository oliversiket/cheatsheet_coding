# Building an API

with Laravel

---

# Validation

---

## Validation

You should always validate any data that gets submitted to your site.

`4xx` errors can be dealt with by the client side, `5xx` errors cannot.

To avoid MySQL errors:

- `required`: any database fields that cannot be `null` should have the `required` validation
- `max:255`: if you're storing data in a `VARCHAR` then make sure you have max length validation that matches the `VARCHAR` length
- `date`/`integer`/`string`: check formats before inserting into MySQL (you will also need the `nullable` validation if the field is not required)


---

## Validation

To add validation we need to create Request classes.

Run `artisan make:request ArticleRequest`.

```php
class ArticleRequest extends FormRequest
{
    public function authorize()
    {
        return true;
    }

    public function rules()
    {
        return [
           "title" => ["required", "string", "max:100"],
           "article" => ["required", "string", "min:50"],
        ];
    }
}
```

---

## Validation

Then update the `Articles` controller to use our validated request instead of the standard `Request` object:

```php
use App\Http\Requests\ArticleRequest;

// ...

public function store(ArticleRequest $request) { /* ... */ }

// ...

public function update(ArticleRequest $request, Article $article) { /* ... */ }
```

---

# Comments

---

## One to Many Relationships

We want to store comments on their own table in the database. But we'll need some way of linking a comment to the article that it belongs to.

Each article can have **many** comments, but each comment can only belong to **one** article. For this reason it is called a **one to many** relationship.

We can store this relationship by referencing the ID of the article for each comment we create. That way we will know which article each comment belongs to.

Under the hood, MySQL can really efficiently use this structure to join together related data.

<img src="img/one-to-many.png" style="width: 450px; margin-top: 3em;">

---

## Commenting

A comment belongs to an article and has an email address and the comment text.

```bash
artisan make:model Comment -m
```

```php
public function up()
{
    Schema::create("comments", function (Blueprint $table) {
        $table->increments("id");
        $table->string("email", 100);
        $table->text("comment");
        $table->timestamps();

        // link up to articles table
        $table->integer("article_id")->unsigned();
        $table->foreign("article_id")->references("id")->on("articles")->onDelete("cascade");
    });
}
```

```bash
artisan migrate
```

---

## Commenting

We need to let our articles know that they have a relationship to comments. That way Eloquent can automatically join them together for us.

Let's update our Article model to let it know that it can have comments:

```php
class Article extends Model
{
    // Only allow the title and article field to get updated via mass assignment
    protected $fillable = ["title", "article"];

    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}
```

---

## Commenting

And let's update the Comment model while we're at it:

```php
class Comment extends Model
{
    protected $fillable = ["email", "comment"];

    public function article()
    {
        return $this->belongsTo(Article::class);
    }
}
```

---


## Commenting

We just need to set up a route to capture the comment `post`:

```php
$router->group(["prefix" => "articles"], function ($router) {
    ...
    $router->post("{article}/comments", "Comments@store");
});
```

We'll need to create the Comments controller too:

```shell
artisan make:controller Comments --api
```

---

## Commenting

Update the `store` method in the `Comments` controller:

```php
use App\Article;
use App\Comment;

class Comments extends Controller
{
    public function store(Request $request, Article $article)
    {
        $comment = new Comment($request->only(["email", "comment"]));

        // store the comments on the article
        $article->comments()->save($comment);

        return $comment;
    }
}
```

---

## Commenting

Now, let's add some comment validation.

```bash
artisan make:request CommentRequest
```

```php
class CommentRequest extends FormRequest
{
    public function authorize()
    {
        return true;
    }

    public function rules()
    {
        return [
            "email" => ["required", "email", "max:100"],
            "comment" => ["required", "string"],
        ];
    }
}
```

---

## Commenting

And update the `Comments` controller to use the validated request:

```php
use App\Http\Requests\CommentRequest;

// ...

public function store(CommentRequest $request, Article $article) { /* ... */ }
```

---

## Commenting

Finally, let's list all the comments for an article:

```php
$router->group(["prefix" => "articles"], function ($router) {
    ...
    $router->get("{article}/comments", "Comments@index");
});
```

```php
class Comments extends Controller
{
    // ...

    public function index(Article $article)
    {
        return $article->comments;
    }
}
```

---

## Resource

Let's create a resource for comments: `artisan make:resource CommentResource`

```php
public function toArray($request)
{
    return [
        "id" => $this->id,
        "email" => $this->email,
        "comment" => $this->comment,
    ];
}
```

And then update our `Comments` controller:

```php
use App\Http\Resources\CommentResource;

// ...

public function index(Article $article)
{
    return CommentResource::collection($article->comments);
}

public function store(CommentRequest $request, Article $article)
{
    // ... store code
    return new CommentResource($comment);
}
```

---

## Key Terms

- **One to Many**: a relationship between two tables of a database where the items of table A can be linked to many items from table B, but items from table B can only be linked to one item in table A.
- **Foreign Key**: a relationship between two tables in MySQL that is enforced by the database

---

## Additional Reading

- [Validation](https://laravel.com/docs/5.6/validation)
- [Eloquent: One to Many Relationships](http://laravel.com/docs/5.6/eloquent-relationships#one-to-many)
