#+title: laravel 8.x rest
#+author: mit
#+EMAIL: your@mail.address
#+category: programming
#+startup:
#+tags:


#+setupfile: /home/mit/Documents/org/org-macros.setup
# //  フォントの色の変更，ハイライト https://taipapamotohus.com/post/html_export/

#+HTML_HEAD: <link href="http://fonts.googleapis.com/css?family=Roboto+Slab:400,700|Inconsolata:400,700" rel="stylesheet" type="text/css" />
# #+HTML_HEAD: <link rel="stylesheet" type="text/css" href="https://raw.githubusercontent.com/thi-ng/org-spec/master/css/style.css">
#+HTML_HEAD: <link href="http://fonts.googleapis.com/css?family=Roboto+Slab:400,700|Inconsolata:400,700" rel="stylesheet" type="text/css" />

#+HTML: <div class="outline-2" id="meta">
| *Author* | {{{author}}} ({{{email}}})    |
| *Date*   | {{{time(%Y-%m-%d %H:%M:%S)}}} |
#+HTML: </div>

# +INFOJS_OPT: view:info toc:t path:http://orgmode.org/org-info.js mouse:underline
# view: info/overview/content/showall
# #+TOC: headlines 2

#+BEGIN_COMMENT
https://qiita.com/deco/items/997b25a3bd09baaa8d8a
https://takaxp.github.io/org-ja.html

C-u C-u C-u TAB	--> すべてを展開して表示

C-c C-e h	--> (org-export-as-html)
C-c C-e h h	--> write html
C-c C-e h o     --> Open with Browser

C-c C-s	--> SCHEDULED: <2012-11-15 木>
C-c C-d	--> DEADLINE: <2012-11-16 金>
C-c .	--> <2012-11-16 金>
C-u C-c .	--> <2012-11-16 金 22:11>

# org-structure-template-alist
<c Tab	--> tempo-template-org-comment
<s Tab	--> tempo-template-org-comment
        i(index), A(ascii), H(html), L(latex), v(verse), s(src), q(quote),
        E(export), l(export latex), h(export html), e(example), C(comment), c(center), I(include)...

<>(active), [](no-active)

 Repeat(d,w,m,y) <2019-11-17 Sun 12:30 +1y>
 Range <2020-11-17 10:00>--<2020-11-19 12:00>




日時（所要時間）
場所
参加者（担当者）
議題
配布資料の有無


#+END_COMMENT





Refer
https://dev.to/kingsconsult/how-to-create-a-secure-crud-restful-api-in-laravel-8-and-7-using-laravel-passport-31fh

* STEP 1: install laravel 8
** To install the latest laravel framework, which is laravel 8.0 as of the time of publishing this article, run the command below

#+BEGIN_SRC
composer create-project --prefer-dist laravel/laravel [laravel_8_api_crud]
#+END_SRC

** APP_KEY
#+BEGIN_SRC
php artisan key:generate --ansi
#+END_SRC


* Step 2: Database setup

#+BEGIN_SRC
...
DB_CONNECTION=sqlite
#DB_CONNECTION=mysql
#DB_HOST=127.0.0.1
#DB_PORT=3306
#DB_DATABASE=laravel
#DB_USERNAME=root
#DB_PASSWORD=
...
#+END_SRC


* Step 3: Install Laravel Passport
** install passport.
#+BEGIN_SRC
composer require laravel/passport
#+END_SRC

** app/Providers/AppServiceProvider.php
#+BEGIN_SRC
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Schema;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Schema::defaultstringLength(191);  //
    }
}
#+END_SRC

** Create Database
#+BEGIN_SRC
php artisan migrate
#+END_SRC

* Step 4: Create the encryption keys

#+BEGIN_SRC
php artisan passport:install
#+END_SRC

* Step 5: Add the HasApiTokens trait to our user model
Go to App\Models\User.php and tell the User class to

#+BEGIN_SRC
use HasApiTokens,
#+END_SRC

Also add this to the top

#+BEGIN_SRC
use Laravel\Passport\HasApiTokens;
#+END_SRC

* Step 6: Call the passport routes in AuthServiceProvider
Go to App/Providers/AuthServiceProvider.php and add

#+BEGIN_SRC
Passport::routes();
#+END_SRC

To the boot method, also add the path before the class at the top

#+BEGIN_SRC
use Laravel\Passport\Passport;
#+END_SRC

* Step 7: Set the driver
This will be the final step in the setting up and configuration of Laravel\Passport, we going to change our api driver from the default token to passport.
Go to config\auth.php

#+BEGIN_SRC

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
            'hash' => false,
        ],
    ],

#+END_SRC

* Step 8: Create the Migration file for our CRUD api project
** make Model
#+BEGIN_SRC
php artisan make:model Project -m
#+END_SRC

** migration file in database/migrations folder.
#+BEGIN_SRC
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateProjectsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('projects', function (Blueprint $table) {
            $table->id();
            $table->string('name', 255);
            $table->string('introduction', 500)->nullable();
            $table->string('location', 255)->nullable();
            $table->integer('cost')->nullable();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('projects');
    }
}
#+END_SRC

** model file. in app/Models folder
#+BEGIN_SRC
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Project extends Model
{
    use HasFactory;

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name',
        'location',
        'introduction',
        'cost',
    ];


    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = [
        'cost' => 'int',
    ];
}
#+END_SRC

* Step 9: Migrate the new table
#+BEGIN_SRC
php artisan migrate
#+END_SRC

* Step 10: Create a Resource
When building an API, the response will always be in JSON format, so we need a transformation layer that sits between our Eloquent models and the JSON responses. This is what will serve the response to the application’s user in a JSON format. Laravel provides us with a resource class that will help in transforming our models and the model collections into JSON. So we are going to create that

#+BEGIN_SRC
php artisan make:resource ProjectResource
#+END_SRC

This will create a folder in the app directory called Resources and also a file ProjectResource.php inside the resources.

#+BEGIN_SRC
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class ProjectResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return parent::toArray($request);
    }
}
#+END_SRC

*Step 11: Create our Controllers
The Controller is responsible for the direction of the flow of data and an interface between the user and the database and views. In this case, we are not interacting with views now because we are dealing with API, so our response will be in JSON format. The standard for RESTful APIs is to send the response in JSON.
We are going to be creating two controllers, the first will be the Authentication Controller and the second is our Project Controller, we need the Authentication Controller in order to generate the token to use in Project Controller.

#+BEGIN_SRC
php artisan make:controller API/AuthController
#+END_SRC

This will create a folder called API in App/Http/Controllers. It will also create a new file called AuthController.php. Click on AuthController.php and update it with the following code.

#+BEGIN_SRC
<?php

namespace App\Http\Controllers\API;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use App\Models\User;
use Illuminate\Support\Facades\Hash;



class AuthController extends Controller
{
    public function register(Request $request)
    {
        $validatedData = $request->validate([
            'name' => 'required|max:55',
            'email' => 'email|required|unique:users',
            'password' => 'required|confirmed'
        ]);

        $validatedData['password'] = Hash::make($request->password);

        $user = User::create($validatedData);

        $accessToken = $user->createToken('authToken')->accessToken;

        return response(['user' => $user, 'access_token' => $accessToken], 201);
    }

    public function login(Request $request)
    {
        $loginData = $request->validate([
            'email' => 'email|required',
            'password' => 'required'
        ]);

        if (!auth()->attempt($loginData)) {
            return response(['message' => 'This User does not exist, check your details'], 400);
        }

        $accessToken = auth()->user()->createToken('authToken')->accessToken;

        return response(['user' => auth()->user(), 'access_token' => $accessToken]);
    }
}
#+END_SRC

*In our AuthController, we created two methods: register and logic methods
In the register method, we use the Laravel Validate method to make sure that the name, email, and password is provided, this will also make sure that the email has not been taken and is a valid email address, the password must be confirmed before the user will be added.

After the validation, we use hash to encrypt the password before creating the user, we can't store plain password, lastly, we grab the access token and return it with the user’s information.
In the login method, we also validate the data been pass, to make sure the email and password are submitted, if the data did not correspond to any user, it will return a message that the user does not exist, if it corresponds, then it returns the user and the access token.

Let us create our ProjectController in the API using --api switch.

#+BEGIN_SRC
php artisan make:controller API/ProjectController --api --model=Project
#+END_SRC

The --api switch will create our Controller without the create and edit methods, those methods will present HTML templates.
Go to App/Http/Controller/API, and click on ProjectController, copy the code below and update the methods.

#+BEGIN_SRC
<?php

namespace App\Http\Controllers\API;

use App\Http\Controllers\Controller;
use App\Models\Project;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;
use App\Http\Resources\ProjectResource;

class ProjectController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $projects = Project::all();
        return response([ 'projects' => ProjectResource::collection($projects), 'message' => 'Retrieved successfully'], 200);
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $data = $request->all();

        $validator = Validator::make($data, [
            'name' => 'required|max:255',
            'description' => 'required|max:255',
            'cost' => 'required'
        ]);

        if ($validator->fails()) {
            return response(['error' => $validator->errors(), 'Validation Error']);
        }

        $project = Project::create($data);

        return response(['project' => new ProjectResource($project), 'message' => 'Created successfully'], 201);
    }

    /**
     * Display the specified resource.
     *
     * @param  \App\Models\Project  $project
     * @return \Illuminate\Http\Response
     */
    public function show(Project $project)
    {
        return response(['project' => new ProjectResource($project), 'message' => 'Retrieved successfully'], 200);
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Models\Project  $project
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Project $project)
    {
        $project->update($request->all());

        return response(['project' => new ProjectResource($project), 'message' => 'Update successfully'], 200);
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param  \App\Models\Project  $project
     * @return \Illuminate\Http\Response
     */
    public function destroy(Project $project)
    {
        $project->delete();

        return response(['message' => 'Deleted']);
    }
}
#+END_SRC

- The index method will retrieve all the projects in the database with a success message (Retrieved successfully) and returns a status code of 200.
- The store method will validate and store a new project, just like the AuthController, and returns a status code of 201, also a message of "Created successfully".
- The show method will retrieve just one project that was passed through the implicit route model binding, and also returns an HTTP code of 200 if successful.
- The update method receives the HTTP request and the particular item that needs to be edited as a parameter. It updates the project and returns the appropriate response.
- The destroy method also receives a particular project through implicit route model binding and deletes it from the database. ### Step 12: Create our Routes Go to routes folder and click on api.php, and updates with the following code

#+BEGIN_SRC
<?php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\API\AuthController;
use App\Http\Controllers\API\ProjectController;

/*
|--------------------------------------------------------------------------
| API Routes
|--------------------------------------------------------------------------
|
| Here is where you can register API routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| is assigned the "api" middleware group. Enjoy building your API!
|
*/

Route::middleware('auth:api')->get('/user', function (Request $request) {
    return $request->user();
});


Route::post('register', [AuthController::class, 'register']);
Route::post('login', [AuthController::class, 'login']);

Route::apiResource('projects', ProjectController::class)->middleware('auth:api');
#+END_SRC

* Step 13: Testing
We are going to be testing our API with Postman, due to complexity and time, I might not be able to explain everything on Postman, but observe the red squares on this page

#+BEGIN_SRC
 curl --header 'Content-Type: application/json' \
        --request POST \
        --data '{ "name": "test", "email": "test@email.com", "password": "secret", "password_confirmation": "secret" }' \
        http://localhost:8000/api/register
#+END_SRC

#+BEGIN_SRC
 curl --header 'Content-Type: application/json' \
        --request POST \
        --data '{ "email": "test@test.com", "password": "test" }' \
        http://localhost:8000/api/login
#+END_SRC

#+BEGIN_SRC
## non Auth data
 curl --header 'Content-Type: application/json' \
        http://localhost:8000/api/open

setenv token
#+END_SRC

#+BEGIN_SRC
## Auth data
 curl --header 'Content-Type: application/json' \
      --header "Authorization: Bearer $TOKEN" \
        http://localhost:8000/api/user

## Auth data
 curl --header 'Content-Type: application/json' \
      --header "Authorization: Bearer $TOKEN" \
        http://localhost:8000/api/closed
#+END_SRC
