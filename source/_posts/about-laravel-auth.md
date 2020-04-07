---
title: Laravel5.1自带认证系统（Auth）改变加密方式带来的思考
date: 2016.03.03 17:31:33
tags: [php]
categories: 后端
---


最近网站在进行重构，已经敲定使用`laravel 5.11.1 LTS`进行重构，这款框架的学习成本相对来说，还是较高，如果PHP知识不扎实的话，可能大部分时间都处于知其然不知其所以然的状态，关于认证这一块我也是摸索着看源码才慢慢懂究竟怎么使用的。
初期遇到的问题如下：
1. Auth认证加密方式的改变，旧版使用md5的加密方式，如何才能让laravel转换成md5认证。
1. laravel认证的原理是怎么样的。

其实本质上这是两个问题，也是一个问题，因为如果我明白了laravel是如何进行认证的，基本也能改认证形式。

于是开始在各个地方寻找解决问题的方案，起手在google搜`laravel md5`，其实大部分人都不建议使用md5的方式来做用户密码的保存。于是慢慢的我的思路转变为，如果将md5转化为Bcrypt。

在网上搜了很多资料，其实没有特别的帮助我去理解这个认证系统，大部分资料都只告诉了怎么做，于是就自己开始读源码了。
在`vendor/laravel/framework/src/Illuminate/Auth/Guard.php`下，认证系统的逻辑就在这里面，登录使用的函数源码如下：
```
    /**
     * Attempt to authenticate a user using the given credentials.
     *
     * @param  array  $credentials
     * @param  bool   $remember
     * @param  bool   $login
     * @return bool
     */
    public function attempt(array $credentials = [], $remember = false, $login = true)
    {
        $this->fireAttemptEvent($credentials, $remember, $login);
        //下面这句对用户名进行了验证
        $this->lastAttempted = $user = $this->provider->retrieveByCredentials($credentials);

        // If an implementation of UserInterface was returned, we'll ask the provider
        // to validate the user against the given credentials, and if they are in
        // fact valid we'll log the users into the application and return true.
        if ($this->hasValidCredentials($user, $credentials)) {//这里对密码进行了验证，因此要探究加密方式就要从这里看起
            if ($login) {
                $this->login($user, $remember);
            }

            return true;
        }

        return false;
    }

    /**
     * Determine if the user matches the credentials.
     *
     * @param  mixed  $user
     * @param  array  $credentials
     * @return bool
     */
    protected function hasValidCredentials($user, $credentials)
    {
        return ! is_null($user) && $this->provider->validateCredentials($user, $credentials);
    }
```
代码写得很漂亮，很容易看得懂，跟着作者的逻辑跑，在`$this->provider->retrieveByCredentials($credentials);`中，laravel验证了用户是否存在
验证逻辑的实现在`vendor/laravel/framework/src/Illuminate/Auth/EloquentUserProvider.php`
源码如下：
```
    /**
     * Retrieve a user by the given credentials.
     *
     * @param array $credentials
     *
     * @return \Illuminate\Contracts\Auth\Authenticatable|null
     */
    public function retrieveByCredentials(array $credentials)
    {
        // First we will add each credential element to the query as a where clause.
        // Then we can execute the query and, if we found a user, return it in a
        // Eloquent User "model" that will be utilized by the Guard instances.
        $query = $this->createModel()->newQuery();

        foreach ($credentials as $key => $value) {
            if (!Str::contains($key, 'password')) {
                $query->where($key, $value);
            }
        }

        return $query->first();
    }

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
上面这两个函数就不做赘述了，之后就是最关键的密码认证方面了，密码认证函数是写在`vendor/laravel/framework/src/Illuminate/Hashing/BcryptHasher`
我们想要的check和make函数就在这里：
```
    /**
     * Hash the given value.
     *
     * @param string $value
     * @param array  $options
     *
     * @return string
     *
     * @throws \RuntimeException
     */
    public function make($value, array $options = [])
    {
        $cost = isset($options['rounds']) ? $options['rounds'] : $this->rounds;

        $hash = password_hash($value, PASSWORD_BCRYPT, ['cost' => $cost]);

        if ($hash === false) {
            throw new RuntimeException('Bcrypt hashing not supported.');
        }

        return $hash;
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
        if (strlen($hashedValue) === 0) {
            return false;
        }
        
        return password_verify($value, $hashedValue);//关于这个函数可以直接搜索php password_verify。
    }
```
是的，没错，你可以直接修改这里来改变你的加密方式，但是这种做法是不科学的，如果你没有对这个框架烂熟于心，我不建议你直接对源码进行修改！不建议！因为你并不知道改变源码会对你的项目带来什么影响！鉴于这个原因，以及我又很想使用laravel自带的Auth认证系统（废话，能省事怎么会不想用）。于是我打算改变加密策略，对我的密码加密进行升级。具体的实现思路，就是让用户先进行判断用户的密码以及账号是否正确，如果正确则将新版密码替换旧版密码存进数据库。然后进行用户登录。
```
      if ($user) {
            if (MD5Hasher::check($password,$user->user_password)){
                $user->user_password = Hash::make($password);
                $user->save();
            }
        }
```
思路大概如上。
总体来说，在用户数量不是很多的情况下，这种方式还是可以接受的，日后用户全部转换回来之后，可以去除掉这一层检查。关于如何检查用户是否全部升级完密码，检查密码长度则可。

后记，这里必须提一个问题就是，使用attempt登录，laravel是直接在调用了getAuthPassword，里面返回的是$this->password，如果你数据库里面的用户表，密码存储的字段并不是password，而是其他，例如mypassword，则在UserModel改写`getAuthPassword`成`return $this->mypassword`。

添加多一种方法，如果你不想修改源码，但是也想替换的话，那么，你可以通过容器去解决这个问题，laravel的设计确实很巧妙。详细你可以参考我的[另外一篇文章](http://www.jianshu.com/p/8032745fd794)。