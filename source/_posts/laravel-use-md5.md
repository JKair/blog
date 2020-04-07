---
title: Laravel不修改源码的前提Auth验证替换成md5
date: 2016.12.14 11:48:39
tags: [laravel]
categories: 后端
---

在读这篇文章之前，请先弄明白容器的运作原理。
注意，版本是`5.1 LTS`那个。其他版本的原理都差不多。

我们在换成laravel的时候，可能会遇到一个问题，就是以前可能用md5或者其他验证方式的，然后在laravel里面，密码的认证方式又换了，不想自己重写认证方式，又想沿用以前的md5需求。

其实我们在看laravel的`Auth::attempt()`的时候，可以发现，laravel，调用的验证方法是基于一个hash类。

```
    /**
     * Validate a user against the given credentials.
     *
     * @param \Illuminate\Contracts\Auth\Authenticatable $user
     * @param array                                      $credentials
     *
     * @return bool
     */
    public function validateCredentials(UserContract $user, array $credentials)
    {
        $plain = $credentials['password'];

        return $this->hasher->check($plain, $user->getAuthPassword());
    }
```

我们只要替换了这个hash类，就完全可以做到使用我们想要的验证方式去验证登录了。
那么怎么替换呢？我们其实可以发现，在`config/auth.php`里面，我们设置了auth的驱动，这个驱动一般默认是`eloquent`，在`vendor/laravel/framework/src/Illuminate/Auth/AuthServiceProvider.php`里面。
```
    /**
     * Register the authenticator services.
     *
     * @return void
     */
    protected function registerAuthenticator()
    {
        $this->app->singleton('auth', function ($app) {
            // Once the authentication service has actually been requested by the developer
            // we will set a variable in the application indicating such. This helps us
            // know that we need to set any queued cookies in the after event later.
            $app['auth.loaded'] = true;

            return new AuthManager($app);
        });

        $this->app->singleton('auth.driver', function ($app) {
            return $app['auth']->driver();
        });
    }
```
这里面根据配置绑定了hash的驱动，那么我们接下来就看一下绑定了什么驱动，到目录`vendor/laravel/framework/src/Illuminate/Auth`下，我们可以看到有`EloquentUserProvider.php`。

```
<?php
namespace Illuminate\Auth;

use Illuminate\Support\Str;
use Illuminate\Contracts\Auth\UserProvider;
use Illuminate\Contracts\Hashing\Hasher as HasherContract;
use Illuminate\Contracts\Auth\Authenticatable as UserContract;

class EloquentUserProvider implements UserProvider
{
    /**
     * The hasher implementation.
     *
     * @var \Illuminate\Contracts\Hashing\Hasher
     */
    protected $hasher;

    /**
     * The Eloquent user model.
     *
     * @var string
     */
    protected $model;

    /**
     * Create a new database user provider.
     *
     * @param  \Illuminate\Contracts\Hashing\Hasher  $hasher
     * @param  string  $model
     * @return void
     */
    public function __construct(HasherContract $hasher, $model)
    {
        $this->model = $model;
        $this->hasher = $hasher;
    }
    ....
      /**
     * Validate a user against the given credentials.
     *
     * @param  \Illuminate\Contracts\Auth\Authenticatable  $user
     * @param  array  $credentials
     * @return bool
     */
    public function validateCredentials(UserContract $user, array $credentials)
    {
        $plain = $credentials['password'];

        return $this->hasher->check($plain, $user->getAuthPassword());
    }
    .....
```
通过这个，我们可以看出，这里面在构造方法注入了hash的驱动，然后在验证的时候调用了hash的检查函数去验证用户的密码是否正确，也就是说，我们只要替换了这个hash驱动就可以解决问题了。

那么我们开始做，首先，我们需要去实现laravel的hash接口。弄一个md5的类出来。
```
<?php

namespace Tools\MD5;

use Illuminate\Contracts\Hashing\Hasher;

class MD5 implements Hasher
{
    /**
     * Hash the given value.
     *
     * @param string $value
     *
     * @return array  $options
     * @return string
     */
    public function make($value, array $options = [])
    {
        return md5($value);
    }

    /**
     * Check the given plain value against a hash.
     *
     * @param string $value
     * @param string $hashedValue
     * @param array  $options
     *
     * @return bool
     */
    public function check($value, $hashedValue, array $options = [])
    {
        return $this->make($value) === $hashedValue;
    }

    /**
     * Check if the given hash has been hashed using the given options.
     *
     * @param string $hashedValue
     * @param array  $options
     *
     * @return bool
     */
    public function needsRehash($hashedValue, array $options = [])
    {
        return false;
    }
}

```
这个类你可以自己手动创建在自己对应的文件夹。

然后我们创建一个provider，`php artisan make:provider RiskUserProvider`。
这个类就是关键了。
我们在这个类下，需要直接继承`Illuminate\Auth\EloquentUserProvider`，这样它的许多方法我们都不需要去重写了，我们只要重写它的构造函数。
```
<?php

namespace App\Providers;

use Auth;
use Illuminate\Auth\EloquentUserProvider;

class RiskUserProvider extends EloquentUserProvider
{
  public function __construct($hasher, $model)
  {
      parent::__construct($hasher, $model);
  }
}
```

最后，我们要将这个新的provider，注入auth。
```
        Auth::extend('riak', function($app){
            $model = $this->app['config']['auth.model'];

            return new RiskUserProvider(new MD5(), $model); //这里就是直接将我们新实现的md5类传递过去
        });
```
我们可以在AuthServiceProvider的boot方式注入就好了。
根据[b35f1cb8a1fd](http://www.jianshu.com/users/b35f1cb8a1fd)这个朋友的反馈，5.2是使用`Auth::provider`方法，详细我自己并没有尝试过，不过这位朋友成功了，谢谢这位朋友的反馈。

最后最后，我们在config/auth.php里面，替换驱动成`riak`。这样问题就解决了。

这篇文章写的时候，没有详细解释laravel的容器，如果你觉得看起来晦涩难懂，是因为你没有弄懂容器，详细理解之后会看懂的，laravel的入门门槛确实比较高，对于新手来说可能非常不友好，但是当你理解了之后，会有很大的收获的，克服它。

另外，其实这里的替换方式可以变成直接实现UserProvider的接口，替换整个Provider，这样做会比较正规一点，基本做法和上面描述的都差不多。有错误的地方欢迎指正。

附上一篇[参考资料](https://my.oschina.net/zgldh/blog/379461#OSC_h2_1)