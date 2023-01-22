
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

