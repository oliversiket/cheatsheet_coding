# Laravel
​
## Key Language 

### Framework
-  A foundation of existing code that supports the building and deployment of applications using less code 
​
### Variable
- A value that is stored and can change over time
​
### Function
- A piece of code that takes an argument, performs an action, and returns an output 
​
### Object
- A way of storing various types of information (properties or instance methods) in key/value pairs
​
### Class
- A way of encapsulating code as a template/blueprint for instantiating an Object instance, or creating reusable code
​
### Property
- A variable within a Class  
​
### Method
- A function within a Class
​
### Instance / Instantiate
- An Object instance is an instantiated class 
​
### Class method
- A method that you can call without instantiating a Class
​
### Instance method
- A method that exists within an Object instance
​
### MVC
- Model View Controller 
- A **model** is a t    ype of class where you put code that is responsible for putting data into and getting data out of the database - Singular!! -> Book
- A **view** is ...
- A **controller** is a type of class where you put code that code handles the request and sends the response (manages the requests and responses) - Plural!! -> Books
​
### OOP 
- Object Oriented Programming - a paradigm of organising code 
​
### API
- Application Programming Interface (API)
- In basic terms, APIs just allow applications to communicate with one another
​
### HTTP 
    - Hypertext Transfer Protocol (HTTP) is essentially a well defined text format or standard that the browser is able to interpet into a web page. 
    - By default, servers liten on port 80 for any request that looks like HTTP. If it receives a valid request, it sends back an HTTP response.
​
## HTTP Request Methods
​
- GET - used to retrieve data
- POST - used to submit data
- PUT - used to edit data
- PATCH - used to edit data
- DELETE - used to delete data 
​
### Migration
    - Migrations are like version control for your database, allowing your team to easily modify and share the application's database schema. 
    - Blueprints for the DataBase structure (tables,columns)

### Resource 
    - To generate a resource class, you may use the make:resource Artisan command. By default, resources will be placed in the app/Http/Resources directory of your application. Resources extend the Illuminate\Http\Resources\Json\JsonResource class.
    - Control the format of the JSON response.  

### Mass Assignment Vulnerability
    - Mass assignment is when you send an array to the model creation, basically setting a bunch of fields on the model in a single go, rather than one by one, something like.

### Route Model Binding
    - Laravel route model binding provides a convenient way to automatically inject the model instances directly into your routes. For example, instead of injecting a user's ID, you can inject the entire User model instance that matches the given ID.

### Validation
    - Rules for incoming data and also provide error messages

### Static
    - property that is available on the class not the instance  


## Classes

### Controller
    - Takes an HTTP request and send out HTTP response. plural
### Request
    - 
### Migration
    -

### One-to-many relationship
    - A one-to-many relationship is used to define relationships where a single model owns any amount of other models. For example, a blog post may have an infinite number of comments. Like all other Eloquent relationships, one-to-many relationships are defined by placing a function on your Eloquent model: