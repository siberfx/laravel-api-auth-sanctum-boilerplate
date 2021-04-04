# laravel-api-auth-sanctum-boilerplate
laravel boilerplate with api auth using sanctum (login, logout, reset password)

## Setup Instructions
- clone the repo
  `git clone this.repo`,
- install dependencies
  `composer install`
- perform migrations
  `php artisan migrate`
- serve
  `php artisan serve`
- server hosted @ `localhost:8000`

### Api endpoints for api Authentication

The auth routes are present in `routes/api.php` and prefixed with `auth` as follows:
```php
Route::prefix('auth')->group(function () {
	Route::post('login', 'Api\Auth\AuthController@login')->name('auth.login');
	Route::post('logout', 'Api\Auth\AuthController@logout')->middleware('auth:sanctum')->name('auth.logout');
	Route::get('user', 'Api\Auth\AuthController@getAuthenticatedUser')->middleware('auth:sanctum')->name('auth.user');

	Route::post('/password/email', 'Api\Auth\AuthController@sendPasswordResetLinkEmail')->middleware('throttle:5,1')->name('password.email');
	Route::post('/password/reset', 'Api\Auth\AuthController@resetPassword')->name('password.reset');
});
```
Hence all the api auth routes are prefixed with `/api/auth` and the routes are:

- Login:

  `POST: /api/auth/login`
  ```json
  {
    "email": "johndoe@example.org",
    "password": "password"
  }
  ```
- Logout:

  `POST: /api/auth/logout`

- Get authenticated user details:

  `GET: /api/auth/user`
- Send forgot password email:

  `POST: /api/auth/password/email`

  ```json
  {
    "email": "johndoe@example.org",
  }
  ```

- Reset password:

  `POST: /api/auth/password/reset`

  ```json
  {
    "email": "johndoe@example.org",
    "token": "valid-token-recieved-in-email",
    "password": "password",
    "password_confirmation": "password"
  }
  ```
  
### Quickly create api auth scafold with sanctum instead ? Follow this guide:
> TODO

