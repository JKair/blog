---
title: 学习thinkPHP的笔记
date: 2014.12.03 15:19:51
tags: [php]
categories: 后端
---


#学习thinkPHP的笔记
----------------------

###thinkPHP的文件结构的介绍
- thinkPHP核心文件的介绍
	├─ThinkPHP.php     框架入口文件
	├─Common 框架公共文件
	├─Conf 框架配置文件
	├─Extend 框架扩展目录
	├─Lang 核心语言包目录
	├─Lib 核心类库目录
	│  ├─Behavior 核心行为类库
	│  ├─Core 核心基类库
	│  ├─Driver 内置驱动
	│  │  ├─Cache 内置缓存驱动
	│  │  ├─Db 内置数据库驱动
	│  │  ├─TagLib 内置标签驱动
	│  │  └─Template 内置模板引擎驱动
	│  └─Template 内置模板引擎
	└─Tpl 系统模板目录

- 项目的开始（自动生成目录文件）

``` php
    <?php
    //确定应用名称 Home
    define('APP_NAME', 'Admin');
    //确定应用路径
    define('APP_PATH', './Admin/');
    //引用核心文件
    require './ThinkPHP/ThinkPHP.php';
```

###项目目录结构的说明
---------------------
- Home 前台应用文件夹
├─Common 项目公共文件目录
├─Conf 项目配置目录
├─Lang 项目语言目录
├─Lib 项目类库目录
│  ├─Action Action类库目录
│  ├─Behavior 行为类库目录
│  ├─Model 模型类库目录
│  └─Widget Widget类库目录
├─Runtime 项目运行时目录
│  ├─Cache 模板缓存目录
│  ├─Data 数据缓存目录
│  ├─Logs 日志文件目录
│  └─Temp 临时缓存目录
└─Tpl 项目模板目录

###TP访问项目的五种方式
-----------------------
1. PATHINFO模式
    - 例：`http://域名/项目名/入口文件/模块名/方法名/键1/值1/键2/值2`
2. 普通模式
    - 例：`http://域名/项目名/入口文件?m=模块名&a=方法名&键1=值1&键2=值2`
3. REWRITE模式
    - 例：`http://域名/项目名/模块名/方法名/键1/值1/键2/值2`
4. 兼容模式
    - 例：`http://域名/项目名/入口文件？s=模块名/方法名/键1/值1/键2/值2`
5. 传参模式
    - 例：`http://域名/项目名/入口文件/方法名/方法的参数1/方法的参数2`

###关于TP的数据库操作
---------------
``` php
    <?php
    //以下是连接数据库的两种方式
    //$m=new Model('user');       
    $m=M('user');           
    
    //以下是查询操作
    
    //选取所有数据
    $arr=$m->select();  
    //获取id号为2的单条数据
    $getOne=$m->find(2);   
    //附加条件的字段查询
    $getZiduan = $m->where('id=2')->getField('userName'); 
    //条件查询，大于
    $data['id']=array('GT',6);
    $arr=$m->where($data)->select();
    //条件查询，与和或
    //与
    $getZiduan = $m->where('id=2 and userName="jk"')->select(); 
    //或
    $data['id']=1;
    $data['_logic']='or';
    $getOr = $m->where($data)->select();
    
    //以下是删除操作
    
    //单条删除
    $m->where('userName="gogogo"')->delete();  //返回的是影响的行数
    //删除所有数据
    $m->delete(); 
    
    // 以下是增加数据操作
    $data['userName']='myName';
    $data['sex']=0;
    $m->add();
```
####条件查询中关于逻辑参数
-------------------------
| 参数 |作用|
|:-----:| :--: |
|EQ|等于|
|NEQ|不等于|
|GT|大于|
|EGT|大于等于|
|LT|小于|
|ELT|小于等于|
|LIKE|模糊查询|

####区间查询
```php
    $data['id']=array(array('GT',5),array('lt',8)); //表示SELECT * FROM `tp_user` WHERE ( (`id` > 5) AND (`id` < 8) )
    $data['id']=array(array('GT',5),array('lt',8),'or');//表示SELECT * FROM `tp_user` WHERE ( (`id` > 5) OR (`id` < 8) )
    
   $data['id']=array(array('GT',5),array('lt',8),'or');
   $data['userName']=array(array('LIKE','%2%'),array('LIKE','%啊%'),'or');
    //以上两句表示SELECT * FROM `tp_user` WHERE ( (`id` > 5) OR (`id` < 8) ) AND ( (`userName` LIKE '%2%') OR (`userName` LIKE '%啊%') ) 
```
- 实际上区间查询就是两个以上的数组组成的条件的组合，让TP自动生成查询语句。可以上一个点中提到的逻辑参数进行选择

####统计查询
```php
    $data['userName']=array('LIKE','%啊%');
    $arr=$m->where($data)->count();
    //SELECT COUNT(*) AS tp_count FROM `tp_user` WHERE ( `userName` LIKE '%啊%' ) LIMIT 1
    
    $arr=$m->max('id');
    //SELECT MAX(id) AS tp_max FROM `tp_user` LIMIT 1
    
    $arr=$m->avg('id');
    //SELECT AVG(id) AS tp_avg FROM `tp_user` LIMIT 1
```

- 统计查询中，可能用到的参数

|参数|作用|
| :------: | :-------: |
|count|获取个数|
|max|获取最大值|
|min|获取最小值|
|avg|获取平均数|
|sum|获取总和|

####数据库的连贯操作
|参数|作用|
| :------: | :-------: |
|where|查询条件的定义|
|order|排序|
|limit|限制查询结果|
|field|定义查询的字段|
|table|定义要操作的数据表名称|
|group|group操作|
|having|having操作|
####SQL直接查询（即不使用TP提供的任何方法）
```php
    $m=M();         //创建一个不指向任何表的模块
    $arr=$m->query('select * from tp_user where id>5');     //如果成功返回数据，不成功返回false
    var_dump($arr);
    
    $arr=$m->execute("insert into tp_user(userName) values('kair')");   //成功就返回影响函数，否则返回false
    var_dump($arr);             
```

###向视图文件中传递参数
---------------------
首先，在Tpl文件下创建对应控制器函数名的html文件，例子：
####lib/Action/IndexAction.php
``` php
<?php
class IndexAction extends Action 
{
    public function index()
    {
        $m=M('user'); 
        $data['id']=array('GT',6);
        $arr=$m->where($data)->select();
        $this->assign('data',$arr);
        $this->display();
    }
}
```

####tpl/User/index.html
``` html
<!DOCTYPE html>

<html>
    <head>
        <meta charset="UTF-8">
        <title>title</title>
    </head>
    <body>
        <table border="1" width="500" align='center'>
            <tr>
                <th>id</th>
                <th>username</th>
                <th>sex</th>
                <th>操作</th>
            </tr>
            <volist name='data' id='vo'>
                <tr>
                    <td><{$vo.id}></td>
                    <td><{$vo.userName}></td>
                    <td><{$vo.sex}></td>
                    <td><a href="#">删除</a> | <a href="#">修改</a></td>
                </tr>
            </volist>
        </table>
    </body>
</html>

```

###关于TP的项目设置
-------------------
- 通过修改生成目录下的config文件来达到项目修改的目的

| 设置 | 参数 |例子|作用|
|:-----:|:-----:|:--:| :--: |
|URL_PATHINFO_DEPR|任意合法符号|/|修改URL分隔符|
|TMPL_L_DELIM|任意符号|<{|修改左边界符|
|TMPL_R_DELIM|任意符号|}>|修改右边界符|
|DB_TYPE|所要关联数据库的类型|mysql|设置数据库关联的类型|
|DB_HOST|主机|localhost|设置主机|
|DB_NAME|数据库名|thinkphp|设置数据库|
|DB_USER|用户名|数据库的用户名|设置用户名|
|DB_PWD|密码|数据库的密码|设置密码|
|DB_PORT|端口|3306|设置端口|
|DB_PREFIX|表前缀|tp_|设置表前缀|
|SHOW_PAGE_TRACE|true|false|开启页面跟踪|
|DB_DSN|数据库的DSN参数|mysql://root:@localhost:3306/thinkphp|通过DSN方式访问数据库|
|TMPL_TEMPLATE_SUFFIX|想要设置的后缀名|.html / .php|控制器读取模板的后缀名将会改变|

- 例子

```php
<?php
return array(
    //'配置项'=>'配置值'
    'TMPL_L_DELIM'=>'<{',   //修改左定界符
    'TMPL_R_DELIM'=>'}>',    //修改右定界符
    'DB_TYPE' =>'mysql',    //设置数据库类型
    'DB_HOST'=>'localhost',     //设置主机
    'DB_NAME'=>'thinkphp',      //设置数据库
    'DB_USER'=>'root',      //设置用户
    'DB_PWD'=>'',       //设置密码
    'DB_PORT'=>'3306',      //设置端口
    'DB_PREFIX'=>'tp_',      //设置表前缀
    'SHOW_PAGE_TRACE'=>TRUE     //页面跟踪
);
```