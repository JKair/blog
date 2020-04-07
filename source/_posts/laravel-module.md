---
title: Laravel 组件小总结
date: 2016.03.24 16:02:50
tags: [laravel]
categories: 后端
---

最近捣鼓laravel捣鼓的多，感觉不总结一下过阵子又会忘记了，然后又好像没学到什么一样，由于接触到的是新框架，就要多多的理解框架的特性。
这里稍微踩一下，不知道是国内环境问题还是某度问题，laravel相关的知识真心少，真心不多！
这里想写的是对于laravel各个组件的总结。
`备注：本篇不是教程！而是总结！没有可以完全照抄的代码，需要理解运作流程再动手写！每个人的环境和流程都不一样，请勿完全照抄。`
1. ORM
这一块是相对比较通用的一块，其实关于这个并不是laravel最大的特性，因为很多框架都会有，laravel也有。之前使用这一块的时候，由于文档说明的真心不多，导致使用的时候很多爆炸性报错，然后最后只能慢慢读源码，看一下laravel的设定是怎么样的。
首先要说明的是几个属性的问题，如下：
```
    /**
     * 这里设定的是表名，就是本模型在数据库里面所对应的表
     */
    protected $table = 'user';
    /**
     * 这里是设定字段，说明什么字段可以赋值
     */
    protected $fillable = ['user_name', 'user_password'];
    /**
     * 这里的设定，是当你实例化此模型（意思就是你在数据库里面查出来了某条对应的数据），需要隐藏这个模型某些字段的信息
     */
    protected $hidden = ['user_password', 'remember_token'];
    /**
     * 创建时间维护，如果你这里标识为null，就表示不维护创建时间，更新时间是同理的
     */
    const CREATED_AT = 'create_time';
    /**
     * 更新时间维护
     */
    const UPDATED_AT = 'update_time';
```
这里你很有可能会有这样的需求，就是当你只想让laravel维护你的更新时间，并不想让它维护你的创建时间，或者反过来的情况的时候，你可以设置其中一个时间为null，不对时间进行维护，文档中只说明了，如果开关维护时间，这点比较坑= =，反正我是看完源代码才明白，它里面本身就有这个设定，源码如下:
```
     /**
     * Update the creation and update timestamps.
     *
     * @return void
     */
    protected function updateTimestamps()
    {
        $time = $this->freshTimestamp();

        if (! $this->isDirty(static::UPDATED_AT)) {   //这一行代码就对更新时间进行了设定，
                                             //如果更新时间不存在，那么就不会设置更新时间，
                                             //所以设置UPDATED_AT为null，是可以取消对更新时间的维护的，
                                             //下面创建时间是同理的
            $this->setUpdatedAt($time);
        }

        if (! $this->exists && ! $this->isDirty(static::CREATED_AT)) {
            $this->setCreatedAt($time);
        }
    }

    /**
     * Determine if the model or given attribute(s) have been modified.
     *
     * @param  array|string|null  $attributes
     * @return bool
     */
    public function isDirty($attributes = null)
    {
        $dirty = $this->getDirty();

        if (is_null($attributes)) {
            return count($dirty) > 0;
        }

        if (! is_array($attributes)) {
            $attributes = func_get_args();
        }

        foreach ($attributes as $attribute) {
            if (array_key_exists($attribute, $dirty)) {
                return true;
            }
        }

        return false;
    }

    /**
     * Get the attributes that have been changed since last sync.
     *
     * @return array
     */
    public function getDirty()
    {
        $dirty = [];

        foreach ($this->attributes as $key => $value) {
            if (! array_key_exists($key, $this->original)) {
                $dirty[$key] = $value;
            } elseif ($value !== $this->original[$key] &&
                                 ! $this->originalIsNumericallyEquivalent($key)) {
                $dirty[$key] = $value;
            }
        }

        return $dirty;
    }
```
以上是关于模型设置的小总结。
这里特别提醒一点，就是关联模型的使用的！在文档中也有提醒，假设你需要使用的关联表数据比较多，请直接贪婪架加载，[用with方法](http://laravelacademy.org/post/140.html)！
另外，当我们需要进行更多层的关联的时候，可以使用with('xxx.xxx')去关联，来达到这种效果：当前模型->关联模型->关联模型。举个例子：当你需要通过当前的模型来查找用户的信息，而用户的信息的获取需要先关联用户表(假设关联关系user)，再关联用户信息表(假设关联关系为info)才能拿到的时候，你可以这样写with('user.info')，这样就能拿到信息了。

2. Request
Request这个设定极大的方便了对于表单的处理，在这一块功能内，你可以对表单的请求进行控制，当然，可以对整个请求进行控制，权限控制什么的，表单验证什么的。我对于Request的理解，就是这一层可以对表单请求进行验证，以及对此次表单请求的权限等进行验证。
使用的流程如下：
  - 创建一个Request `php artisan make:request MyRequest`
  - 在request中加入你的使用逻辑：
  ```
<?php
namespace App\\Http\\Requests\\Auth;
use App\\Http\\Requests\\Request;
class MyRequest extends Request
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        //这里写你的权限逻辑，return true or false 代表着请求通过不通过
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            //这里写验证规则
        ];
    }
}
  ```


  - 在控制层的请求实例化它
    ```
    use App\\Http\\Request\\MyRequest;
    public function getIndex(MyRequest $request)
    {
          //.............你需要做的处理
    }
    ```
 - 这样你就能完整使用一个Request了，注意，上面的代码并没有完全实现，只是其中一部分关键代码，请认真理解，而不是直接照抄！
总结：Request极大的方便了对于表单的处理，以前对表单的处理太复杂，验证是表单最麻烦的一块，laravel的这一块极大的方便了开发。具体如何使用验证的，请参考[文档](http://laravelacademy.org/post/240.html)

3. Middleware
  中间件，这是laravel5新增的东西，简直就是666666666666，关于这一块，做权限验证是爆炸性好用，你可以在路由层就直接拦截请求进行处理，实在方便，而且甚至你还可以在请求达到Request层之前，就对数据进行准备！6666。中间件的使用并不难，步骤如下：
  - `php artisan make:middleware MyMiddleware`
  - 在`Kernel.php`内添加这个中间件，还可以设置别名。
  - 在路由层设置中间件。
  ```
  Route::put('post/{id}', ['middleware' => 'MyMiddleware', function ($id) {
    //
  }]);
  ```
  - 这里不对如何使用进行强调了，因为[文档](http://laravelacademy.org/post/57.html)已经说得很清楚了，我想说的是另外一件事情，关于resource如何使用中间件的。目前我的使用方式就是将resource放入group来使用。resource本身没有middware的设定。= =在stackoverflow，有鬼佬说可以使用before或者after，具体还没实验，日后补上。
4. Event
说到Event组件，不得不说，这一层也可以完美的处理我们的一个事件流程，这里我说一个场景来帮助我们理解这个组件。
假设有一个系统，上面如果有用户注册了，需要发送短信以及邮件去通知管理员，那我们最平常的处理方法就是在将用户信息写入数据库之后，再调用发送短信以及邮件函数，去达到通知管理员的目的，但是这样做有存在一个问题，就是假设多个地方需要做这样的通知，那就会有茫茫多的冗余代码，而假设不止要做短信通知和邮件通知那么简单，在短信通知里面还要分对象，不同模板去通知，这样下来的处理就会大爆炸，代码会非常恶心，都是一块一块的冗余。然而这个组件帮我们解决了这样的问题，而且我们还可以十分方便的做多个通知，甚至还能阻止事件冒泡（关于这个后面举一个详细的例子）。
关于Event组件如何使用，代码如何写，配置如何写的，这里不做赘述，[文档](http://laravelacademy.org/post/198.html)非常清楚。我想通过一个流程来说明这个组件：
```
用户注册->信息写入数据库->Event::fire(new UserRegister())(触发注册事件)->Listenser捕捉并处理事件
```
这一套处理流程能让我们更清晰以及更简洁的去描述整个注册流程，鹅妹子吟。当然我所说的只是一种应用场景，还有更加丰富的用法，我举这个例子，是为了帮助我们更好的去理解[Event](http://laravelacademy.org/post/198.html)在我们这整个系统中的应用
5. Polices（策略）
关于策略这个组件，最大的好处就是和中间件配合使用了，我对策略这个组件的理解，就是分离函数权限，中间件我用来控制整个控制器的权限，而策略更加细化，可以细化到某个函数，我在某些控制器方法里面进行了过滤，可能我需要的权限检查，不止一层，有好几层，但是只单单针对一个方法的，那么你完全可以使用策略这个组件满足你的需求。策略的使用方法分为以下几个步骤：
 - 创建策略 `php artisan make:policy MyPolicy`
 - 填写策略 
    ```
    public function index(User $user,TestModel $test)
    {
        return $user->can('use_this');
    } 
    ```
  - 在Providers/AuthServiceProvider里面配置$policies
  - 在控制器中使用Gate门面调用。`Gate::denies('index', new TestModel())`
  - 这里给一个注意点，就是策略的使用，第二个参数，只要传模型实例，laravel会根据你的配置，调用策略，前面的参数则是调用策略的方法。

目前只简单的总结这几个组件，接下来有时间，会继续分析这个框架。

如果你有什么问题，可以在文章下留言，进行交流。以下推荐了两个网站是关于laravel文档的，你可以在里面获取laravel相关的知识。