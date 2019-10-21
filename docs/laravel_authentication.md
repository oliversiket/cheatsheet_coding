## Learning Objectives
- Understand how an API authenticates requests
- Be able to setup Laravel Passport to handle request authentication
- Update your API to authenticate routes in different ways

## Key Language
- Authentication
- Access Token

## Authentication
APIs typically use tokens to authenticate users. The process is as follows:
- The user makes a request to a dedicated authentication endpoint, providing credentials such as username and password
- The API checks whether these credentials are valid
- If valid, the API returns a token that is unique to the user
- The user makes all future requests supplying this token in the header
- The API checks the validity of the token
- The API grants or denies requests based on whether the user is valid and who the user is

### Installation
Install Laravel Passport
````
composer require laravel/passport
````

Create database tables to store access tokens in (vagrant ssh)
````
artisan migrate
````

Install Laravel Passport
````
artisan passport:install
````

Make a note of the password grant Client ID and Client Secret that are returned following successful installation. These look something like:
````
...
Password grant client created successfully.
Client ID: 2
Client secret: QtE4syo8QFwvw4bMEK5Ej2zaJ3jgF3RYmF1JjZmU
````

In App/User.php add the HasApiTokens trait:
````
use Laravel\Passport\HasApiTokens;
...

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;
    ...
}
````


In AuthServiceProvider.php use the Passport class to add all necessary routes to your app. This saves you having to write them yourself.  
````
use Laravel\Passport\Passport;
...
class AuthServiceProvider extends ServiceProvider
{
	...
	public function boot()
    {
    	...
    	Passport::routes();
    }
}
````

Finally, in config/auth.php config file, set the driver of the api authentication guard to 'passport'.
````
'guards' => [
    ...

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
````


Create a single user on the command line...
````
artisan tinker
App\User::create(array('name' => 'An Author', 'email' => 'an.author@gmail.com', 'password' => Hash::make('password')));
````

You should now be able to make a request in Postman to POST `http://homestead.test/oauth/token` with the body:  
````
{
	"grant_type": "password"
	"client_id": "<your_client_id>",
	"client_secret": "<your_client_secret>",
	"username": "an.author@gmail.com",
	"password": "password"
}
````
If everything is setup correctly, the token that gets returned can be stored then sent as a Bearer Token with each following request.

### Simple authentication

If a valid access token is sufficient authentication, Laravel provides some simple methods. In `routes/api.php`:

#### Single route
Add the `auth:api` middleware to any routes that require a valid access token:
````
$router->get("", "Articles@index")->middleware('auth:api');
````
Try calling `GET http://homestead.test/api/articles` in Postman (remember to add the header `Accept: application/json`). You should get a 401 response.

This time choose `Bearer Token` in the Authorization tab and paste in the access token you got back from the `oauth/token` endpoint. Your request should now authenticate successfully and return the usual response.

#### Groups
To add authentication to the entire group:  
````
$router->group(["prefix" => "articles", "middleware" => ["auth:api"]], function ($router) {
	...
}
````

### More complex authentication
In reality our blog api probably needs different levels of authentication. Posts should be open to anyone, only the owner of the blog should be able to write and edit blog posts, and only logged in users should be able to comment.

This suggests two user roles - `Author` and `Subscriber`.

#### Modify Users table
We'll need to modify the Users table to be able to store the user role.
````
artisan make:migration modify_users_table
````

In the generated migration file, add your new column in the up() method, and drop it in the down() method:
````
class ModifyUsersTable extends Migration
{
    public function up()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('role')->after('name')->default('subscriber');
        });
    }

    public function down()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('role');
        });
    }
}
````

````
artisan migrate
````

We'll need to update our Users model to make this new column fillable.
````
class User extends Model
{
    protected $fillable = ['name', 'email', 'password', 'role'];
}
````

Now let's update our existing user to have the role of `'author'` and generate a new user with the role `'subscriber'`.

````
artisan tinker
App\User::where('id', 1)->update(['role' => 'author']);
App\User::create(array('name' => 'A Subscriber', 'role' => 'subscriber', 'email' => 'a.subscriber@gmail.com', 'password' => Hash::make('password')));
````

#### Checking user role
Laravel provides a simple way for us to check who is making the request. In the `authorize()` method on the FormRequest classes that we extended, `$this->user()` will return an instance of User associated with the token sent with the request.  

Let's create a new request for storing articles:
````
artisan make:request ArticleStoreRequest
````

The `rules()` method should return the same validation as ArticleRequest:
````
public function rules()
{
    return [
        "title" => ["required", "string", "max:100"],
        "article" => ["required", "string"]
    ];
}
````

But let's write a conditional outcome in the `authorize()` method:  
````
public function authorize()
{
    return $this->user()['role'] === 'author';
}
````

We'll need to tell the `store()` method in Articles.php to use this new Request:
````
use App\Http\Requests\ArticleStoreRequest;

class Articles extends Controller
{
    ...
    public function store(ArticleStoreRequest $request)
    {
        ...
    }
}
````

In Postman, make a call to `POST /oauth/token`, supplying the body as before, replacing the user details with the subscriber we generated above. Make a call to `POST /api/articles` suppling the returned access token as a the Bearer Token. Your request should receive a 403 response. Replace the token with your author token. This should authenticate and a new Article should be returned.

### Extension Tasks
- Validate comment routes so that only subscribers and authors can comment
- Authenticate update and delete routes so that only the writer of the article/comment etc. should be able to edit it. (Hint: you'll need to set up a relationship between Users and Articles)
- Add another role of `admin` and generate an admin user. Admins should be able to do everything on the site, including deleting comments and articles that they didn't write.
- Extend your API providing endpoints for users to sign up to your blog