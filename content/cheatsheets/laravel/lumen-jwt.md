---
title: Lumen JWT
date: 2019-06-16 
draft: true
---
https://github.com/tymondesigns/jwt-auth/issues/1102#issuecomment-296785192

## add tymon/jwt

```bash
composer add tymon/jwt
```

## Follow guide on link
&nbsp;

## Using middleware

use middleware 

- `jwt.auth` result --> error unauthorizedHttpException token not provided
- `auth` result --> return string 'unauthorized'

Use the `auth` one, 

### Using the Auth

in this case, we wanted to protect our `AuthController` with this middleware, I have tried 2 ways, 

1. Assign middleware at route

```php
<?php

$route->group(['prefix' => 'auth', 'middleware' => 'auth'], function($router){
    //...
});

?>
```

This attempt could get your middleware wraps your route group, but you could NOT make any route exception. I have asked at forum you could not define exception on this. Don't know why yet. 

Now to the real solution

### 2. Assign middleware at Controller

This is somewhat better approach for my case, I could make exception for login route, so I could access login page without needing to as token from login page. Yo Dawg!

#### Make Sure you have auth middleware registered

Follow tutorial on the link for now

#### Assign middleware via controller's __construct

```php
<?php
    class AuthController extends Controller
    {
        public function __construct()
        {
            $this->middleware('auth');
            $this->middleware('auth', ['except' => ['login']]);
        }
    }
?>
```

the first one will add auth middleware to all function, the second one will assign middleware and add exception to login function. 

**NOTE** : inside except you assign the function name, not route

## Generate Token

to generate token, you use 

`Tymon\JWTAuth\Facades\JwtAuth::attempt()`

ex.

```php
<?php
use Tymon\JWTAuth\Facades\JwtAuth;
    
    public function login()
    {
        JWTAuth::attempt(['emial@mail.com', "asdfasdf"]);
    }
?>
```

## Get User from token

```php
<?php
use Tymon\JWTAuth\Facades\JwtAuth;
    
    // ... class declaration

    public function me()
    {
        if(! $user = JWTAuth::parseToken()->authenticate()){
            return response()->json(['user not found'], 404);
        }
        return $user;
    }
?>
```
