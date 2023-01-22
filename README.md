
# Laravel — React Application with Passport, Redux, and Material UI (Backend)  

I this project, shall be building a fullstack application (frontend and backend) using Laravel and React. I will focus more on the security architecture (authentication and authorization). For the backend authentication, I will be using Passport for authentication. There are other methods of authentication you can use to secure your API in Laravel. We have:

### Authentication methods
1.JWT (I wrote a tutorial on this, you can check it out here )
2.Laravel Sanctum
3.Laravel Authentication — Built in Laravel Authentication
4.Laravel Socialite.

### Application Structure
The application will be divided into two. 
    The backend where i will design our API and the frontend where i  will design the User interface etc Hence, thr application will be structured as follows.

 1 .Backend — Containing our Laravel application for developing the API (we will work on this in this first part of the tutorial)
 
 2 .Frontend — Containing our React — Redux Application

### Creating the application
#### Step 1 : Installing laravel
 ` laravel new react-backend
   or
  composer create-project laravel/laravel react-backend`
 
#### step 2: Installing passport
`
composer require laravel/passport`

#### Step 3: Artisan migrate
`php artisan migrate`

#### Step 4: Setting up/Installing passport inoder to generate the secured keys for authentications

`php artisan passport:install`

#### Step 5: Adding HasAPITokens Trait to the users model
The Traits suplies our model with methods to inspect the authenticated users scopes and tokens


`use Laravel\Passport\HasApiTokens;
class User extends Authenticatable
{
    //Add this...
    use HasApiTokens, HasFactory, Notifiable;
    …
}`

#### Step 6: Registering Routes for issuing access tokens
We need to register the routes necessary to issue and revoke access tokens. 
To do this, we will call the `Passport::routes` method within the **boot** method of the **AuthServiceProvider.**
php file located in **app/Providers folder.** 
Also, within this method, we can indicate _how long_ we want the token to remain **valid** 
`
....
class AuthServiceProvider extends ServiceProvider
{
...
public function boot()
{
    $this->registerPolicies();
    if (! $this->app->routesAreCached()) {
    Passport::routes();
    Passport::tokensExpireIn(now()->addDays(1));
}
}
}`

#### Step 6: Setting the driver in the auth token for the api to work(config/auth.php)=>Authservice Provider
`return [
    'defaults' => [
        'guard' => 'web',
        'passwords' => 'users',
    ],`

   
  ` 'api' => [
       'driver' => 'passport',
       'provider' => 'users',
       'hash' => false
   ],
    'guards' => [
        'web' => [`
  `          'driver' => 'session',
            'provider' => 'users',
        ],
    ],
    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\Models\User::class,`
     `   ],
        // 'users' => [
        //     'driver' => 'database',
        //     'table' => 'users',
        // ],
    ],`

    'password_timeout' => 1080
   ];`

#### Step 8 : Creating  CORSMiddleware and registering it.

`php artisan make:middleware CORSMiddleware`

This will help us to handle and set all required headers for the react application. 
It will allow cross origin request coming from all several domains.

I will make a middleware for this, 
which will serve as a bridge between the request and the response,filtering all request and reponces via it.

`class CORSMiddleware
{
public function handle(Request $request, Closure $next)
{`
   ` //Intercepts OPTIONS requests`

    if($request->isMethod('OPTIONS')){
    $response = \response('', 200);
    }else{

    //Pass the request to the next middleware

    $response = $next($request);
    }
    $response->header('Access-Control-Allow-Origin',"*");
    $response->header('Access-Control-Allow-Methods','PUT, GET, POST, DELETE, OPTIONS, PATCH');
    $response->header('Access-Control-Allow-Headers',$request->header('Access-Control-Request-Headers'));
    $response->header('Access-Control-Allow-Credentials','true');
    $response->header('Accept','application/json');
    $response->header('Access-Control-Expose-Headers','location');
`return $response;
}`

#### Step 9 : Register the middleware in the kernel

`protected $routeMiddleware = [
...
'CORS' => \App\Http\Middleware\CORSMiddleware::class,
];`

#### Step 10: Create API routes

This will create endpoints where we can access our application. I will also attach the CORSMiddleware  so that all requests will pass through the middleware

`
Route::group(['prefix' => 'users', 'middleware' => 'CORS'], function ($router) {
Route::post('/register', [UserController::class, 'register'])->name('register.user');
Route::post('/login', [UserController::class, 'login'])->name('login.user');
Route::get('/view-profile', [UserController::class, 'viewProfile'])->name('profile.user');
Route::get('/logout', [UserController::class, 'logout'])->name('logout.user');
});`

#### Step 11:  Creating  a respondWithToken helper function in Controller.php
 ...
