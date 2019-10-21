# Building an API

with Laravel

---

# Tags

---

## Many to Many Relationships

Tags and articles have a more complex relationship than comments and articles. An article can have any number of tags, but a tag can also belong to any number of articles.

We can't just reference the article or tag ID from the other table in this case, as that way we could only reference a single item.

In this case we need a **pivot table**. The pivot table *just* stores the relationship between articles and tags.

<img src="img/pivot.png" style="height: 200px; margin-top: 3em;">

---

## Tags

Let's add tags to our articles. First, let's create a Tag model: `artisan make:model Tag -m`

```php
public function up()
{
    Schema::create("tags", function (Blueprint $table) {
        $table->increments("id");
        $table->string("name", 30); // tags just need a name property, don't need timestamps
    });

    Schema::create("article_tag", function (Blueprint $table) {
        $table->increments("id");
        $table->integer("article_id")->unsigned();
        $table->integer("tag_id")->unsigned();
        $table->foreign("article_id")->references("id")->on("articles")->onDelete("cascade");
        $table->foreign("tag_id")->references("id")->on("tags")->onDelete("cascade");
    });
}

public function down()
{
    Schema::drop("article_tag");
    Schema::drop("tags");
}
```

---


## Tags

We'll need to setup our `Article` model to link to tags:

```php
public function tags()
{
    return $this->belongsToMany(Tag::class);
}
```

---

## Tags

Next, let's update our `Tag` model to include the `articles` relationship:

```php
class Tag extends Model
{
    public $timestamps = false; // don't need timestamps
    protected $fillable = ["name"]; // name should be fillable

    public function articles()
    {
        return $this->belongsToMany(Article::class);
    }
}
```

---

## Tags

We'll need to add validation rules for tags to `ArticleRequest`:

```php
public function rules()
{
    return [
       "title" => ["required", "string", "max:100"],
       "article" => ["required", "string", "min:50"],
       "tags" => ["required", "array"], // check tags is an array
       "tags.*" => ["string", "max:30"], // check members of tags are strings
    ];
}
```

---

## Tags

Now, we'll need to update our `Articles` controller to add tags to the model:

```php
use App\Tag;

// ...

public function store(ArticleRequest $request)
{
    $data = $request->only(["title", "article"]);
    $article = Article::create($data);

    $tags = Tag::parse($request->get("tags"));
    $article->setTags($tags);

    return $article;
}


public function update(ArticleRequest $request, Article $article)
{
    // ...

    $tags = Tag::parse($request->get("tags"));
    $article->setTags($tags);

    // ...
}
```

---

## Tags

The `Tag::parse()` and `$article->setTags()` methods don't exist yet, so we'll need to create them ourselves.

In the `Tag` model:

```php
// accepts the array of strings from the request
public static function parse(array $tags)
{
    // turns into a collection and maps over
    return collect($tags)->map(function ($tag) {
        // remove any blank spaces either side
        $string = trim($tag);
        return static::makeTag($string);
    });
}

private static function makeTag($string)
{
    // check if tag already exists
    $exists = Tag::where("name", $string)->first();

    // if tag exists return it, otherwise create a new one
    return $exists ? $exists : Tag::create(["name" => $string]);
}
```

---

## Tags


In the `Article` model:

```php
use Illuminate\Support\Collection;

// ...

public function setTags(Collection $tags)
{
    // update the pivot table with tag IDs
    $this->tags()->sync($tags->pluck("id")->all());
    return $this;
}
```

---

## Resource

Finally, let's update our article resources to include tags.

In `ArticleResouce` and `ArticleListResource`:

```php
public function toArray($request)
{
    // make sure tags are up to date
    $this->resource->load("tags");

    return [
        // ... other properties
        "tags" => $this->tags->pluck("name"), // just return a list of tag names
    ];
}
```

---

## Key Terms

- **Many to Many**: a relationship between two tables of a database where the items of table A can be linked to many items from table B, and the items from table B can be linked to many items in table A.

---

## Additional Reading

- [Eloquent: Many to Many Relationships](http://laravel.com/docs/5.6/eloquent-relationships#many-to-many)
- [Collections](https://laravel.com/docs/5.6/collections)
