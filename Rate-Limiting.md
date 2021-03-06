Rate Limiting allows you to limit the number of requests a client can make in a given amount of time. A limit and the expiration time is defined by a throttle. By default the package has two throttles, an authenticated throttle and an unauthenticated throttle.

To disable rate limiting simply remove all the throttles from the configuration file.

### Changing Rate Limiting Key

By default rate limiting is applied to a clients IP address. To change this default behaviour you can register your own resolver which should return a string
to be used by the rate limiter.

```php
App::make('api.limiter')->setRateLimiter(function ($container, $request) {
    return $container['example']->getRateLimiterKey(); 
});
```

The first parameter will be the IoC container and the second is the request instance that is being rate limited.

### Route Specific Throttling

If you want to only rate limit certain routes or groups of routes you can use the `limit` and `expires` options in your routes.

```php
Route::api('v1', function () {
    Route::get('users', ['limit' => 100, 'expires' => 5, function () {
        return User::all();
    }]);
});
```

This would set a request limit of 100 with an expiration time of 5 minutes for this specific route. If you were to set it on the group then each route within the group would have a limit of 100.

```php
Route::api(['version' => 'v1', 'limit' => 100, 'expires' => 5], function () {
    Route::get('users', function () {
        return User::all();
    });

    Route::get('posts', function () {
        return Post::all();
    });
});
```

A user could visit both the `/users` route and the `/posts` route 100 times each. The limit does not apply to the entire group but to each route within the group.

### Custom Throttles

You may need a custom throttle for more complex scenarios where you need to meet a couple of conditions in order for the throttle to be applied. A throttle must implement the `Dingo\Api\Http\RateLimit\ThrottleInterface`, however, an abstract class does exists to quickly get started. Each of the predefined throttles extends this abstract class.

```php
use Illuminate\Container\Container;
use Dingo\Api\Http\RateLimit\Throttle;

class CustomThrottle extends Throttle
{
    public function match(Container $app)
    {
        // Perform some logic here and return either true or false depending on whether
        // your conditions matched for the throttle.
    }
}
```

You can then configure your throttle.

```php
'throttling' => [
    'custom' => new CustomThrottle(['limit' => 200, 'expires' => 10])
]
```

[← Authentication](https://github.com/dingo/api/wiki/Authentication) | [Internal Requests →](https://github.com/dingo/api/wiki/Internal-Requests)
