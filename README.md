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

### Endpoints for API Authentication

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
### api endpoints
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
  
### Quickly create api auth scaffold with sanctum instead ? Follow this guide:

1. Create a laravel project
	
`composer create-project laravel/laravel my-project`

2. Install sanctum

`composer require laravel/sanctum`

3. Configure sanctum

`php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"`

4. Migrate databases

`php artisan migrate`

5. To begin issuing tokens for users, your User model should use the `Laravel\Sanctum\HasApiTokens` trait:

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```
6. Create a AuthController that has all the authentication methods. Use any Folder Structure/Namespace you want, Here `App\Http\Controllers\Api\Auth` namespace is used.

`php artisan make:controller Api/Auth/AuthController`

7. Add following methods inside AuthController.

```php
<?php

namespace App\Http\Controllers\Api\Auth;

use Illuminate\Validation\ValidationException;
use Illuminate\Auth\Events\PasswordReset;
use Illuminate\Support\Facades\Password;
use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Hash;
use Illuminate\Http\Request;
use Illuminate\Support\Str;
use App\Models\User;

class AuthController extends Controller
{
	/*
	 * Generate sanctum token on successful login
	*/
	public function login(Request $request) {
		$request->validate([
			'email' => 'required|email',
			'password' => 'required',
		]);

		$user = User::where('email', $request->email)->first();

		if (! $user || ! Hash::check($request->password, $user->password)) {
			throw ValidationException::withMessages([
				'email' => ['The provided credentials are incorrect.'],
			]);
		}

		return response()->json([
			'user' => $user,
			'access_token' => $user->createToken($request->email)->plainTextToken
		], 200);
	}


	/*
	 * Revoke token; only remove token that is used to perform logout (i.e. will not revoke all tokens)
	*/
	public function logout(Request $request) {

		// Revoke the token that was used to authenticate the current request
		$request->user()->currentAccessToken()->delete();
		//$request->user->tokens()->delete(); // use this to revoke all tokens (logout from all devices)
		return response()->json(null, 200);
	}


	/*
	 * Get authenticated user details
	*/
	public function getAuthenticatedUser(Request $request) {
		return $request->user();
	}


	public function sendPasswordResetLinkEmail(Request $request) {
		$request->validate(['email' => 'required|email']);

		$status = Password::sendResetLink(
			$request->only('email')
		);

		if($status === Password::RESET_LINK_SENT) {
			return response()->json(['message' => __($status)], 200);
		} else {
			throw ValidationException::withMessages([
				'email' => __($status)
			]);
		}
	}

	public function resetPassword(Request $request) {
		$request->validate([
			'token' => 'required',
			'email' => 'required|email',
			'password' => 'required|min:8|confirmed',
		]);

		$status = Password::reset(
			$request->only('email', 'password', 'password_confirmation', 'token'),
			function ($user, $password) use ($request) {
				$user->forceFill([
					'password' => Hash::make($password)
				])->setRememberToken(Str::random(60));

				$user->save();

				event(new PasswordReset($user));
			}
		);

		if($status == Password::PASSWORD_RESET) {
			return response()->json(['message' => __($status)], 200);
		} else {
			throw ValidationException::withMessages([
				'email' => __($status)
			]);
		}
	}
}
```
8. Add Authentication related routes inside `routes/api.php` to bind appropriate routes with appropriate methods of AuthController

```php
// Auth
Route::prefix('auth')->group(function () {
	Route::post('login', 'Api\Auth\AuthController@login')->name('auth.login');
	Route::post('logout', 'Api\Auth\AuthController@logout')->middleware('auth:sanctum')->name('auth.logout');
	Route::get('user', 'Api\Auth\AuthController@getAuthenticatedUser')->middleware('auth:sanctum')->name('auth.user');

	Route::post('/password/email', 'Api\Auth\AuthController@sendPasswordResetLinkEmail')->middleware('throttle:5,1')->name('password.email');
	Route::post('/password/reset', 'Api\Auth\AuthController@resetPassword')->name('password.reset');
});
```

Thats about it ! All the endpoints and implementations for auth routes as mentioned [here](#api-endpoints) is complete and ready for test.

> If theres any problem on this please open an issue !
