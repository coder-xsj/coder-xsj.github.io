学习laravel

#### Artisan 命令

<font color="red">php artisan list 来查看所有可用的 Artisan 命令</font>

![image-20201208213724137](https://i.loli.net/2020/12/08/L3dxJMAjUniRCEP.png)

- 生成 KEY

  ```php
  php artisan key:generate
  ```

- 创建软链接

  ```php
  php artisan storage:link
  ```

- 数据表迁移

  ```php
  php artisan migrate
  ```

- 生成配置、路由、事件缓存

  ```php
  php artisan route:cache 
  php artisan config:cache 
  php artisan event:cache
  ```



php artisan --version 查看laravel版本

php artisan config:cache 刷新.env缓存

php artisan route:list 显示已添加路由列表

php artisan migrate:refresh --seed   同时完成数据库的重置和填充操作

#### composer

composer require "name" 安装插件

composer remove  "name" 移除插件

composer self-update 升级composer



中间件

php artisan make:middleware CheckToken  定义CheckToken 中间件   在app/Http/middleware目录下

注册中间件 在app/Http/kernel.php $routeMiddleware 中分配一个key

使用中间件

~~~php
->middleware('key');
~~~

php artisan make:controller PostController --resource  创建资源控制器



php artisan make:request StoreBlogPost  创建表单请求类  /app/Http/Request



ctrl + . 折叠代码



#### git 操作流程

 git  对当前文件提供版本管理，其核心思想是对当前文件建立对象数据库，将历史版本信息存放在这个数据库中

最后需要强调的一点是，如果你不慎在创建.gitignore文件之前就push了项目，那么即使你在.gitignore文件中写入新的过滤规则，这些规则也不会起作用，Git仍然会对所有文件进行版本管理。

简单来说，出现这种问题的原因就是Git已经开始管理这些文件了，所以你无法再通过过滤规则过滤它们。

所以大家一定要养成在项目开始就创建.gitignore文件的习惯，否则一旦push，处理起来会非常麻烦。



 原因在于Git的忽略，Git在同步代码时，设置本地忽略文件的前提是，必须保证Git的远程端仓库中没有这个要忽略的文件。当远端包含有该文件时，本地设置的ignore将不再发挥作用。

解决方法：

  在本地的.gitignore文件里面添加上.idea/workspace.xml文件。

  如果已经将本地的文件提交到了远端，那么需要将远端提交的文件给删掉，删除指令为：

```java
git rm -r --cached .idea 
```

可以使用git status指令来查看删掉的文件，基本上都是***.xml文件。

~~~
git config --global user.name "coder-xsj"
git config --global user.email '2449382518@qq.com'

--- 提一句 ---
git config -l 查看当前的所有信息

~~~

##### git常用命令

~~~git
git checkout master    #切换到主分支
git checkout -b static-pages(分支名)     -b 创建新的分支
git checkou master
git add -A
git commit -m "message"
git merge beautiful-layout-style
git push
~~~

**验证当前本地属性：**

~~~git
git config --local --list
git config --global user.email "your-email"
git config --global push.default simple
~~~





##### 初始设置

此设置是 Git 命令 push 的默认模式为 simple，当我们执行 git push 没有指定分支时，自动使用当前分支，而不是报错。

git init

git add .

git commit -m "第一次提交"

\# 创建git仓库

```
git remote add origin git@github.com:coder-xsj/lara_blog.git
git branch -M main
# 推送到远程仓库中
git push -u origin main
```

添加项目到远程仓库

Quick setup — if you’ve done this kind of thing before

Get started by [creating a new file](https://github.com/coder-xsj/hello_laravel_two/new/main) or [uploading an existing file](https://github.com/coder-xsj/hello_laravel_two/upload). We recommend every repository include a [README](https://github.com/coder-xsj/hello_laravel_two/new/main?readme=1), [LICENSE](https://github.com/coder-xsj/hello_laravel_two/new/main?filename=LICENSE.md), and [.gitignore](https://github.com/coder-xsj/hello_laravel_two/new/main?filename=.gitignore).（空项目）

```git
echo "# hello_laravel_two" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:coder-xsj/hello_laravel_two.git
git push -u origin main
                
```

…or push an existing repository from the command line（已存在的项目）

```git
git remote add origin git@github.com:coder-xsj/hello_laravel_two.git
git branch -M main
git push -u origin main
```

所以本地有项目选择第二种

![image-20201208205830234](https://i.loli.net/2020/12/08/J3yefDuhGPMLVHm.png)

接下来设置 Git 推送分支时相关配置：

git config --global push.default simpleCopy
此设置是 Git 命令 push 的默认模式为 simple，当我们执行 git push 没有指定分支时，自动使用当前分支，而不是报错。

项目本地访问地址：http://www.larablog.com/

创建静态页面

git一个分支

~~~git 
git checkout -b static-pages
~~~

![image-20201208211029654](https://i.loli.net/2020/12/08/X2YFGmihnOVPdxD.png)



##### 将本地分支推送到远程并创建

```bash
								  本地分支					远程分支			
git push origin Product-module:Product-module

```

有时候会遇到

```bash
>git push
致命：当前分支购物车订单没有上游分支。
fatal: The current branch Shopping-cart-order has no upstream branch.
要推送当前分支并将远程分支设置为上游，请使用
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin Shopping-cart-order

```

 请使用 关联远程分支

```bash
 git push --set-upstream origin Shopping-cart-order:Shopping-cart-order
```



在 Laravel 中我们较为常用的几个基本的 HTTP 操作分别为 GET、POST、PATCH、DELETE。

1. GET 常用于页面读取
2. POST 常用于数据提交
3. PATCH 常用于数据更新
4. DELETE 常用于数据删除

生成控制器（默认目录app\http\controllers）

~~~
php artisan make:controller StaticPagesController
~~~

![image-20201208211638551](https://i.loli.net/2020/12/08/hbzo9Zpgax1dDty.png)

返回视图

~~~php
return view('static_pages/home');
~~~

#### blade模板继承

layouts/defalut.blade.php

yield:提供(意思就是提供一个占位符)

section-stop: 填充内容

~~~php+HTML
<!DOCTYPE html>
<html>
<head>
    <title>Weibo App</title>
</head>
<body>
    @yield('content')
</body>
</html>
~~~

static_pages/home.blade.php

~~~php
@extends('layouts.default')
@section('content')
  <h1>主页</h1>
@stop
~~~

有值给title，就加载title变量，没有使用默认值Weibo App

```php+HTML
父视图 @yield('content')   ----> 子视图 @section('content')  ... @stop 中内容
```

~~~php
@yield('title', 'Weibo App')
~~~

给值给title

~~~
@section('title', '帮助')
~~~

最终结果

<img src="https://i.loli.net/2020/12/08/ro1lSgcx9tejOP2.png" alt="image-20201208213443910"  />

![image-20201208213457309](https://i.loli.net/2020/12/08/VLzJ5Z4bWaFR6oS.png)



新建并切换到一个新分支后，然后.idea/workspace.xml自动修改了，此时想切换回main分支，然后又不想为这一个文件add，commit，所以查了文档`git checkout filename/directory`，`git checkout . 是清空本地修改`这条命令是回到最近一次git commit 或 git add后的状态，所以解决了问题, 后续避免再出现此类问题,所以在.gitignore文件中配置了.idea文件夹.

![image-20201208215349878](https://i.loli.net/2020/12/08/tVC5x4SFnRUr79o.png)

#### yarn的简介：

Yarn是facebook发布的一款取代npm的包管理工具。

```
npm install -g yarn
查看版本：yarn --version
```

设置源

~~~
npm config set registry=https://registry.npm.taobao.org
yarn config set registry 'https://registry.npm.taobao.org
~~~

将.sass文件编译成.css文件

~~~js
npm run dev
~~~

在每次检测到 .scss 文件发生更改时，自动将其编译为 .css 文件：

~~~js
npm run watch-poll
~~~

![image-20201208222435719](https://i.loli.net/2020/12/08/63VshwdED9pPeaz.png)

laravel在运行时，以public为根目录，将引入 public/css/app.css 样式文件

~~~html
<link rel="stylesheet" href="/css/app.css">
~~~

##### 生成版本号来解决浏览器缓存问题

```php+HTML
<link rel="stylesheet" href="{{ mix('css/app.css') }}">
```

Laravel Mix 给出的方案是为每一次的文件修改做哈希处理，webpack.mix.js中, 加入版本号

~~~js
mix.js('resources/js/app.js', 'public/js')
    .sass('resources/sass/app.scss', 'public/css').version();
~~~

页面结果

![image-20201208224400712](https://i.loli.net/2020/12/08/HYMNagK49srV2OJ.png)

@include 是 Blade 提供的视图引用方法，可通过传参一个具体的文件路径名称来引用视图。

![image-20201208224838248](https://i.loli.net/2020/12/08/A3PMymzbuTKkqtw.png)

{{　}} 是在 HTML 中内嵌 PHP 的 Blade 语法标识符，表示包含在该区块内的代码都将使用 PHP 来编译运行。

route() 方法由 Laravel 提供，通过传递一个具体的路由名称来生成完整的 URL。

##### 命令路由

~~~html
<li class="nav-item"><a class="nav-link" href="{{ route('help') }}">帮助</a></li>
~~~

~~~php
Route::get('/help', 'StaticPagesController@help')->name('help'); //命名路由
~~~

会生成以下路由

http://www.larablog.com/help

![image-20201208233913276](C:\Users\徐大帅\AppData\Roaming\Typora\typora-user-images\image-20201208233913276.png)

命名路由的好处

当需要对生成的url进行更改时，只需要更改路由文件即可，会大大减小工作量。

举例：例如项目中有10处引用`route('help')`，因为特殊原因，url要更改为:http://www.larablog.com/faq,你只需要更改为以下路由，别的啥都不用换，就换一个路由地址

~~~php
Route::get('/faq', 'StaticPagesController@help')->name('help');
~~~

作为比较，如果不使用 route('help') 方式，而是直接在代码中写入 http://www.larablog.com/help ，那你只能一个个的去手工修改。

##### 总结：

一、blade语法

占位符与填充符

~~~
用法1：
@yield('content'); 
@section('content')	
	内容
@stop
用法2:使用默认值，无@stop
@yield('title', 'Weibo App')
@section('title', '帮助')
~~~

继承

```
@extends('layouts.default')
```

引入

```
@include('layouts._footer')
```

当有多个页面内容极为类似时，要采用继承的方式，得有一个父类视图，然后子视图继承。

对于重复的部分，应单独分离出来一个文件。

二、git

前期配置git还得看资料，记不住, 难受

三、yarn、sass、npm与laravelMix构成一套完整的前端工作流

​	明白

四、php artisan 命令行要熟记

五、RESTful风格的路由



放弃所有文件修改：git clean -df



#### Eloquent ORM 

+ increments() 自增长
+ string() 字符型
+ nullable()	不可为空

database/migrations

![image-20201210134634082](https://i.loli.net/2020/12/10/WhbG3M8ykuqr9xI.png)

~~~php
php artisan migrate 生成迁移，执行up方法
~~~

会生成以下表字段,还会生成app/User.php模型

| id                | int       |
| ----------------- | --------- |
| name              | varchar   |
| email             | varchar   |
| email_verified_at | timestamp |
| password          | varchar   |
| remember_token    | varchar   |
| created_at        | timestamp |
| updated_at        | timestamp |

php artisan migrate:rollback 数据回滚down方法

ctrl+shift+R全局替换

生成模型并且创建迁移

~~~php
php artisan make:model Models/Article -m
~~~

![image-20201210140109435](https://i.loli.net/2020/12/10/WFVR8ZxETPY7kQI.png)

#### Tinker 

> Tinker 是一个 REPL (read-eval-print-loop)，REPL 指的是一个简单的、可交互式的编程环境，通过执行用户输入的命令，并将执行结果直接打印到命令行界面上来完成整个操作。

~~~php
php artisan tinker
~~~

> use App\Models\User
>
> User::find(1) 查找id为1
>
> User::first() 查找第一个
>
> User::all()  查找全部
>
> $user->save() 保存到数据库

![image-20201210140742003](https://i.loli.net/2020/12/10/ztbKQBLkav21eTE.png)

#### RESTFul 路由

> 查看目前应用的路由：

~~~php
php artisan route:list
~~~

~~~php
Route::resource('users', 'UsersController');
~~~

![image-20201210142050639](https://i.loli.net/2020/12/10/RCABfljcVqdHQwo.png)

| HTTP请求 | URL                | 动作                    | 作用                   |
| -------- | ------------------ | ----------------------- | ---------------------- |
| GET      | /users             | UsersController@index   | 显示所有用户列表的页面 |
| GET      | /users/{user}      | UsersController@show    | 显示个人用户信息的页面 |
| GET      | /users/create      | UsersController@create  | 创建用户的页面         |
| POST     | /users             | UsersController@store   | 创建用户               |
| GET      | /users/{user}/edit | UsersController@edit    | 修改用户               |
| PATCH    | /users/{user}      | UsersController@update  | 更新用户               |
| DELETE   | /users/{user}      | UsersController@destory | 删除用户               |

> 用户数据进行删除，将数据库重置

~~~php
php artisan migrate:refresh
~~~

> Laravel 提供了全局辅助函数 old 来帮助我们在 Blade 模板中显示旧输入数据。这样当我们信息填写错误，页面进行重定向访问时，输入框将自动填写上最后一次输入过的数据。

~~~php
{{ old('name') }}
~~~

![image-20201210145409755](https://i.loli.net/2020/12/10/eFjzwOgkN2JV7KM.png)

> git提交记录

![image-20201210145529540](https://i.loli.net/2020/12/10/PaTRKMVSIoQHlEr.png)

#### validate的数据验证

~~~php
public function store(Request $request){
    $this->validate($request, [
        'name' => 'required|unique:users|max:50',
        'password' => 'required|min:6|confirmed',
        'email' => 'required|email|unique:users|max:255',
    ]);
 }
~~~

> required: 必填项
>
> unique:users: 唯一性验证
>
> max:255|min:6  长度验证
>
> email: 格式验证
>
> confirmed: 密码匹配验证

> 避免受到csrf跨域请求伪造的攻击

~~~
{{ csrf_field() }}
~~~

> 会自动转化成
>
> CSRF 令牌基于会话（Session），过期时间在 config/session.php 文件中的 lifetime 选项做设定，默认为 2 个小时。

~~~html
<input type="hidden" name="_token" value="sMVWtg5bOlBBFkNbSSOdNcs3ufaxN705vL49eqBn">
~~~

> Laravel 默认会将所有的验证错误信息进行闪存。当检测到错误存在时，Laravel 会自动将这些错误消息绑定到视图上，因此我们可以在所有的视图上使用 errors 变量来显示错误信息。需要注意的是，在我们对 errors 进行使用时，要先使用 count($errors) 检查其值是否为空。

~~~php
@if (count($errors) > 0)
    <div class="alert alert-danger">
        <ul>
            @foreach($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
~~~

#### 汉化
> 1. 安装中文语言包

~~~php
composer require "caouecs/laravel-lang:~6.0"
~~~

> 2. config/app.php

```
'locale' => 'zh-CN',
```

> 3. vender/caouecs/lavavel-lang/src/zn-CN 复制到 resources/lang/zh-CN
> 4. /vendor/caouecs/laravel-lang/json/zh-CN.json 复制到 resources/lang



![image-20201210152748241](https://i.loli.net/2020/12/10/DqbhfzaHor1mdIy.png)

> 由于 HTTP 协议是无状态的，所以 Laravel 提供了一种用于临时保存用户数据的方法 - 会话（Session），并附带支持多种会话后端驱动，可通过统一的 API 进行使用。
> 我们可以使用 session() 方法来访问会话实例。而当我们想存入一条缓存的数据，让它只在下一次的请求内有效时，则可以使用 flash 方法。flash 方法接收两个参数，第一个为会话的键，第二个为会话的值，我们可以通过下面这行代码的为会话赋值。

~~~php
session()->flash('success', '欢迎，您将在这里开启一段新的旅程~');
~~~

> 获取

~~~php
session()->get('success');
~~~

> 实战用法

![image-20201210154204856](https://i.loli.net/2020/12/10/AezrwQvxkK6IlJX.png)

> 用户登录注册功能开发

~~~php
Route::get('login', 'SessionsController@create')->name('login');
Route::post('login', 'SessionsController@store')->name('login');
Route::delete('logout', 'SessionsController@destory')->name('logout');
~~~

| HTTP请求 | URL    | 动作                       | 功能         |
| -------- | ------ | -------------------------- | ------------ |
| GET      | login  | SessionsController@create  | 登录功能页面 |
| POST     | login  | SessionsController@store   | 登录处理逻辑 |
| DELETE   | logout | SessionsController@destory | 退出登录     |

> 登录逻辑

> 例子中，attempt 方法执行的代码逻辑如下：
> 		第一个参数使用 email 字段的值在数据库中查找；
> 		如果用户被找到：
> 1). 先将传参的 password 值进行哈希加密，然后与数据库中 password 字段中已加密的密码进行匹配；
> 2). 如果匹配后两个值完全一致，会创建一个『会话』给通过认证的用户。会话在创建的同时，也会种下一个		名为 laravel_session 的 HTTP Cookie，以此 Cookie 来记录用户登录状态，最终返回 true；
> 3). 如果匹配后两个值不一致，则返回 false；
> 		如果用户未找到，则返回 false。
>
> 第二个参数为是否为用户开启『记住我』功能的布尔值

~~~php
    public function store(Request $request){
        $this->validate($request, [
            'email' => 'required|email|max:255',
            'password' => 'required',
        ]);
        if(Auth::attempt(['email' => $request->email, 'password' => $request->password])){
            session()->flash('success', '欢迎回来~');
            return redirect()->route('users.show', [Auth::user()]);
        } else {
            session()->flash('danger', '很抱歉，您的邮箱和密码不匹配');
            return redirect()->back()->withInput();
        }
    }
~~~

> 使用 withInput() 后模板里 old('email') 将能获取到上一次用户提交的内容，这样用户就无需再次输入邮箱等内容

~~~php
return redirect()->back()->withInput();
~~~

> Auth::user() 方法来获取 当前登录用户 的信息，并将数据传送给路由。

~~~php
return redirect()->route('users.show', [Auth::user()]);
~~~

![image-20201210160655507](https://i.loli.net/2020/12/10/K1gr3pdiHymwbUs.png)

> 由于浏览器不支持发送 DELETE 请求，因此我们需要使用一个隐藏域来伪造 DELETE 请求。
> 在 Blade 模板中，我们可以使用 method_field 方法来创建隐藏域。

~~~php
{{ method_field('DELETE') }}
~~~

转化为HTML

![image-20201210162001573](https://i.loli.net/2020/12/10/pK7m6HJ3AyzDtPn.png)

> 若已登录用不显示帮助和登录

![image-20201210162548673](https://i.loli.net/2020/12/10/VuzcDNEm5gJO1yM.png)![image-20201210162601773](https://i.loli.net/2020/12/10/tgGRSCmOQqW2dfr.png)

> Laravel 中，如果要让一个已认证通过的用户实例进行登录，可以使用以下方法：

~~~php
Auth::login($user);
~~~

> 注销

~~~php
public function destory(){
        Auth::logout();
        session()->flash('success', '您已成功退出！');
        return redirect('login');
    }
~~~

> 在 Laravel 的默认配置中，如果用户登录后没有使用『记住我』功能，则登录状态默认只会被记住两个小时。如果使用了『记住我』功能，则登录状态会被延长到五年。我们可以通过使用 Laravel 提供的『记住我』功能来保存一个记忆令牌，用于长时间记录用户登录的状态。Laravel 默认为用户生成的迁移文件中已包含 remember_token 字段，该字段将用于保存『记住我』令牌。

![image-20201210163156452](https://i.loli.net/2020/12/10/hTRIluyW6tfgCiP.png)

![image-20201210163118371](https://i.loli.net/2020/12/10/JOTkB5vs4pjrMQ2.png)



> edit页面

~~~php
 <form method="POST" action="{{ route('users.update', $user->id )}}">
 	{{ method_field('PATCH') }}
	{{ csrf_field() }}
</form>
~~~

> 转换后的HTML代码

![image-20201211080525595](https://i.loli.net/2020/12/11/xer5WjIGCVPz93h.png)

#### 授权策略

>  使用Laravel 提供身份验证（Auth）中间件来过滤未登录用户的 edit, update 动作
>
> UsersController.php

~~~php
public function __construct(){
    // 必须先登录
    $this->middleware('auth', [
        'except' => ['show', 'create', 'store'], //你可以用的
        'only' => [],	//你不可以用的
    ]);
}
~~~

> 1 创建授权策略 (Policy), 用户只能修改自己的信息

~~~php
php artisan make:policy UserPolicy
~~~

会在`app\Policys下生成UserPolicy.php`策略

~~~php
 public function update(User $currentUser, User $user){
     return $currentUser->id == $user->id;
 }
~~~

update方法用于用户更新时的权限验证



> 2 注册授权策略

app/Providers/AuthServiceProvider.php

~~~php
public function boot()
{
    $this->registerPolicies();
    // 修改策略自动发现的逻辑
    Gate::guessPolicyNamesUsing(function ($modelClass) {
        // 动态返回模型对应的策略名称，如：// 'App\Models\User' => 'App\Policies\UserPolicy',
        return 'App\Policies\\'.class_basename($modelClass).'Policy';
    });

}
~~~

在用户控制器中使用 authorize 方法来验证用户授权策略,authorize 方法接收两个参数，第一个为授权策略的名称，第二个为进行授权验证的数据。

~~~php
$this->authorize('update', $user);
~~~
![image-20201224222653674](https://i.loli.net/2020/12/24/kUj8QBhAzRfKDHr.png)


> 测试，登录1的用户，然后修改2用户的资料

![image-20201211084340997](https://i.loli.net/2020/12/11/SL4ckfPbI1JeaZz.png)

> redirect() 实例提供了一个 intended 方法，该方法可将页面重定向到上一次请求尝试访问的页面上，并接收一个默认跳转地址参数，当上一次请求记录为空时，跳转到默认地址上。

![image-20201211085027191](https://i.loli.net/2020/12/11/8pR3dvgyJaKAh2P.png)

> 只让未登录用户访问登录页面：app/Http/Controllers/SessionsController.php

![image-20201211085739316](https://i.loli.net/2020/12/11/HfwiX5S3MoVWFeq.png)

> 只让未登录用户访问注册页面：app/Http/Controllers/UsersController.php

![image-20201211085905311](https://i.loli.net/2020/12/11/SodC6W8BP2ApaZy.png)

> 测试访问

![image-20201211085945810](https://i.loli.net/2020/12/11/kcE6ICUj9HQo5Yl.png)

> 会被跳转到 Laravel 默认指定的页面 /home ，因我们并没有此页面，所以会报错 404 找不到页面。我们需要修改下中间件里的 redirect() 方法调用，并加上友好的消息提醒：
> app/Http/Middleware/RedirectIfAuthenticated.php

![image-20201211090141495](https://i.loli.net/2020/12/11/cdORAaQxjH1Es2J.png)

> 生成示例用户：假数据
>
> 原始做法就是一个个在数据库中创建
>
> Laravel 提供了一套更加现代化、非常简单易用的数据填充方案b。接下来让我们使用 Laravel 提供的数据填充来批量生成假用户。
> 假数据的生成分为两个阶段：
>
> 1. 对要生成假数据的模型指定字段进行赋值 - 『模型工厂』；
> 2. 批量生成假数据模型 - 『数据填充』；

database/factories/UserFactory.php

赋值用的

database/seeds/UsersTableSeeder.php

填充数据用的，生成

![image-20201213200225668](https://i.loli.net/2020/12/13/fslvx5z7XoYBnqK.png)



##### 分页，记住（paginate）

![image-20201213200328078](https://i.loli.net/2020/12/13/xKrlnkYCPzLDG14.png)

![image-20201213200341665](https://i.loli.net/2020/12/13/1jgQMKDzS9kfIYN.png)



授权策略

~~~php
public function destroy(User $currentUser, User $user){
    // 是管理员，然后不是自己就可以删除
    return $currentUser->is_admin && $currentUser->id !== $user->id;
}
~~~

~~~php
public function destroy(User $user){
    $this->authorize('destroy', $user);  // 调用
    $user->delete();
    session()->flash('success', '用户删除成功!');
    return back();
}
~~~

![image-20201213202630992](https://i.loli.net/2020/12/13/UuVNB5JZbEgsc4t.png)

> 数据库重置并填充数据

~~~php 
php artisan migrate:refresh --seed
~~~



> 在用户模型创建之前生成令牌

~~~php
app/Models/User.php
public static function boot(){
    parent::boot();
     // 在模型被创建之前发生
    static::creating(function ($user) {
        $user->activation_token = Str::random(10);
    });
}

~~~

#### 用户激活

> 1 发送邮箱 UsersController.php

~~~php
protected function sendEmailConfirmationTo($user){
    $view = 'emails.confirm';
    $data = compact('user');
    $from = '2449382518@qq.com';
    $name = 'coder-xsj';
    $to = $user->email;
    $subject = "感谢注册 Weibo 应用！请确认你的邮箱。";

    Mail::send($view, $data, function ($message) use ($from, $name, $to, $subject){
        $message->from($from, $name)->to($to)->subject($subject);
    });
}
~~~

> 2 验证邮箱

~~~php
public function confirmEmail($token){
    $user = User::where('activation_token', $token)->firstOrFail();

    $user->activated = true;
    $user->activation_token = null;
    $user->save();

    Auth::login($user);
    session()->flash('success', '恭喜你，激活成功');
    return redirect()->route('users.show', [$user]);
}
~~~

#### 生产环境中发送邮箱

> 1 qq邮箱开启smtp配置，拿到授权码
>
> 2 配置.env文件
>
> 3 发送邮箱方法修改

~~~
MAIL_MAILER=smtp
MAIL_HOST=smtp.qq.com
MAIL_PORT=25
MAIL_USERNAME=2449382518@qq.com
MAIL_PASSWORD=dmrhgmjftpbzdifa
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=2449382518@qq.com
MAIL_FROM_NAME=WeiboApp
~~~

选项讲解：
MAIL_DRIVER=smtp —— 使用支持 ESMTP 的 SMTP 服务器发送邮件；
MAIL_HOST=smtp.qq.com —— QQ 邮箱的 SMTP 服务器地址，必须为此值；
MAIL_PORT=25 —— QQ 邮箱的 SMTP 服务器端口，必须为此值；
MAIL_USERNAME=xxxxxxxxxxxxxx@qq.com —— 请将此值换为你的 QQ + @qq.com；
MAIL_PASSWORD=xxxxxxxxx —— 密码是我们第一步拿到的授权码；
MAIL_ENCRYPTION=tls —— 加密类型，选项 null 表示不使用任何加密，其他选项还有 ssl，这里我们使用 tls 即可。
MAIL_FROM_ADDRESS=xxxxxxxxxxxxxx@qq.com —— 此值必须同 MAIL_USERNAME 一致；
MAIL_FROM_NAME=WeiboApp —— 用来作为邮件的发送者名称。

> 现在我们已经在环境配置文件完善了邮件的发送配置，因此不再需要使用 from 方法：

![image-20201213221954886](https://i.loli.net/2020/12/13/3vLh1rp9jTo4yVD.png)



> 创建迁移文件

~~~php
php artisan make:migration create_statuses_table --create="statuses"
~~~

> 写入代码字段
>
> 运行迁移

~~~php
php artisan migrate
~~~



#### Eloquent 模型应用

> Eloquent 模型让关联的管理和处理变得更加简单，同时也支持以下几种类型的关联：

+ 一对一
+ 一对多
+ 多对多
+ 远程一对多
+ 多态关联
+ 多态多对多关联

> 微博模型中，指明一条微博属于一个用户。

![image-20201214235055400](https://i.loli.net/2020/12/14/K6sB5h1o73DVZFM.png)

> 在用户模型中，指明一个用户拥有多条微博。

![image-20201214235123096](https://i.loli.net/2020/12/14/y24pvjNTs8d1ISe.png)

##### 日期进行友好化处理（diffForHumans() ）

函数{{ $status->created_at->diffForHumans() }} 该方法的作用是将日期进行友好化处理



#### 生成假数据

> 1、建立工厂

~~~php
php artisan make:factory StatusFactory
~~~

> 2、填入内容

![image-20201214235356605](https://i.loli.net/2020/12/14/iJvYXZbkqlBQWGI.png)

> 3、创建一个 StatusesTableSeeder 文件来对微博假数据进行批量生成。

~~~php
php artisan make:seeder StatusesTableSeeder
~~~

![image-20201214235502433](https://i.loli.net/2020/12/14/fSRbOk9GWJw7U43.png)

> 4、在 DatabaseSeeder 类中指定调用微博数据填充文件。

![image-20201214235538940](https://i.loli.net/2020/12/14/VAICrsDuHtFNYTE.png)

> 5、对数据库进行重置和填充。

~~~php
php artisan migrate:refresh --seed
~~~



> 创建微博操作
>
> view层
>
> ​	给出相应的route
>
> controller层
>
> ​	数据验证
>
> ​	数据写入
>
> ​	页面重定向

授权策略真的实现了我之前觉得很难的实现的功能

模型真的是爱了，不用join连表查询了

数据填充也是爱了, 测试删除功能的时候不用自个录入数据了



> 安装扩展包 验证码

~~~php
 composer require "mews/captcha:~3.0"
~~~



学习正则表达式

裁剪头像（intervention/image）

代码生成器 —— Laravel Scaffold Generator 。代码生成器能让你通过执行一条 Artisan 命令，完成注册路由、新建模型、新建表单验证类、新建资源控制器以及所需视图文件等任务，不仅约束了项目开发的风格，还能极大地提高我们的开发效率。





#### 测试号信息

appID:wxd630d2f5f2250893

appsecret:30d51f25fc50b557f9f8d7cd545ffa80

1. 获取授权码

```
https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect
```

数据

- APPID 测试账号中的 `appID`，填写自己账号的 `appID`
- REDIRECT_URI 用户同意授权后的回调地址，填写 `http://larabbs.test`
- SCOPE 应用授权作用域，填写 `snsapi_userinfo`
- STATE 随机参数，可以不填，我们保持 `STATE` 即可。

![image-20211217232703735](https://s2.loli.net/2021/12/17/xTjsqMNL69KDWSn.png)

```
https://open.weixin.qq.com/connect/oauth2/authorize?appid=wxd630d2f5f2250893&redirect_uri=http://larabbs.test&response_type=code&scope=snsapi_userinfo&state=STATE#wechat_redirect
```

确认登录之后返回地址

```
http://larabbs.test/?code=071tjvFa1XEPZA0eedHa1hsOAg4tjvFP&state=STATE
```

返回的地址就是上面地址填的 `&redirect_uri=http://larabbs.test` 的回调地址，还返回一个 `code`



2. 请求以下链接获取 access_token：

- code 上一步获取的 code

```
https://api.weixin.qq.com/sns/oauth2/access_token?appid=wxd630d2f5f2250893&secret=30d51f25fc50b557f9f8d7cd545ffa80&code=071tjvFa1XEPZA0eedHa1hsOAg4tjvFP&grant_type=authorization_code
```

返回

```json
{
    "access_token": "52_rjlWT7FluEijfrAEYRSpb6yDIFo0bCvWgNdvikbvtboiUgXUl7l_sQ_aun4GtOlrYOlWlcyAlyCPP_yBExvUaw",
    "expires_in": 7200,
    "refresh_token": "52_XGiu33OwG89ZFQEd_p3B-5MyHD-odxUhYEhgz-3qzwtKaHcrCkKpl7_aay7pqkddUpo9zOf9uJTyJl-a6x93_w",
    "openid": "ohLFo5n-FEnpmr3411uk21ntFb9c",
    "scope": "snsapi_userinfo"
}
```



![image-20211217232603958](https://s2.loli.net/2021/12/17/ihTDxMaFNR94LBz.png)



3. 通过 `access_token` 和`openid`获取个人信息

```
https://api.weixin.qq.com/sns/userinfo?access_token=44_m79tW-fKZcwJFmH7N3QbOJzKQwbMiL8tmYkOL398znX1LMT_i2O0YCCRFjR5qybMozhy196thpMO9SpfFjbloQ&openid=ohLFo5n-FEnpmr3411uk21ntFb9c&lang=zh_CN
```

~~~json
{
    "openid": "ohLFo5n-FEnpmr3411uk21ntFb9c",
    "nickname": "我姓徐",
    "sex": 1,
    "language": "zh_CN",
    "city": "",
    "province": "安徽",
    "country": "中国",
    "headimgurl": "https://thirdwx.qlogo.cn/mmopen/vi_32/7S3k867hyib0dpT2CHkJBUKSahyFlicunJRiavic9WuvjicwO0S1e41sBGCX3VJfA6ibibQnbiadGltNGZG6StuRkZCjgg/132",
    "privilege": []
}
~~~

![image-20211217233156865](https://s2.loli.net/2021/12/17/RTeqVyOQ7aNYMLW.png)



#### 功能调试

1. 客户端已经获取`access_token`

~~~php
use Overtrue\Socialite\AccessToken;

// 出于安全的考虑，授权码只能使用一次！！！
$accessToken = new AccessToken([
    'access_token' => '44_m79tW-fKZcwJFmH7N3QbOJzKQwbMiL8tmYkOL398znX1LMT_i2O0YCCRFjR5qybMozhy196thpMO9SpfFjbloQ',
    'openid' => 'ohLFo5n-FEnpmr3411uk21ntFb9c'
]);

$driver = Socialite::driver('wechat');
$oauthUser = $driver->user($accessToken);
$oauthUser->getNickname();
$oauthUser->getId();
~~~

![image-20210507180534048](https://i.loli.net/2021/05/07/Du4HcB2nW8IqGQi.png)

2. 客户端只获取授权码（code）

   通过`微信开发者工具` 获取一个 code

   `https://open.weixin.qq.com/connect/oauth2/authorize?appid=wxd630d2f5f2250893&redirect_uri=http://larabbs.test&response_type=code&scope=snsapi_userinfo&state=STATE#wechat_redirect`

~~~php
$code = 'CODE';
$driver = Socialite::driver('wechat');
$accessToken = $driver->getAccessToken($code);
$oauthUser = $driver->user($accessToken);
$oauthUser->getNickname();
$oatthUser->getId();
~~~

~~~
>>> $oauthUser->getNickname();
=> "我姓徐"
>>> $oauthUser->getId();
=> "ohLFo5n-FEnpmr3411uk21ntFb9c"
~~~

~~~php
<?php

namespace App\Http\Controllers\Api;

Use App\Models\User;
Use Illuminate\Support\Arr;
use Illuminate\Http\Request;
use Overtrue\Socialite\AccessToken;
use Illuminate\Auth\AuthenticationException;
use App\Http\Requests\Api\SocialAuthorizationRequest;
//use App\Http\Controllers\Controller;


class AuthorizationsController extends Controller
{
    public function socialStore($type, SocialAuthorizationRequest $request){
        $driver = \Socialite::driver($type);
        try {
            if($code = $request->code){
                $accessToken = $driver->getAccessToken($code);
            }else{
                $tokenData['access_token'] = $request->access_token;

                // 微信需要增加 openid
                if($type == 'wechat'){
                    $tokenData['openid'] = $request->openid;
                }
                $accessToken = new AccessToken($tokenData);
            }

            $oauthUser = $driver->user($accessToken);
        } catch (\Exception $e){
            throw new AuthenticationException('参数错误，未获取用户信息');
        }

        //
        switch ($type){
            case 'wechat':
                $unionid = $oauthUser->getOriginal()['unionid'] ?? null;

                if($unionid){
                    $user = User::where('weixin_unionid', $unionid)->first();
                }else{
                    $user = User::where('weixin_openid', $oauthUser->getId())->first();
                }

                // 第一次微信授权登录
                if(!$user){
                    $user = User::create([
                        'name' => $oauthUser->getNickname(),
                        'avatar' => $oauthUser->getAvatar(),
                        'weixin_openid' => $oauthUser->getId(),
                        'weixin_unionid' => $unionid,
                    ]);
                }

                break;

        }

        return response()->json(['token' => $user->id]);
    }
}

~~~



JWT: JSON Web Token

+ header
+ payload
+ signature

~~~php
D:\xampp\htdocs\larabbs>php artisan jwt:secret
jwt-auth secret [Xo2LUXkNQ9jV4hViDtjyMp4Sy8SnzsUIk1DIcxbEmkhUrxAXX2WMsjctdjryJKkI] set successfully.
~~~



删除和刷新 token 的路由我设计为：

- PUT /api/authorizations/current —— 替换当前的授权凭证；
- DELETE /api/authorizations/current —— 删除当前的授权凭证。



1. 用户输入手机号，请求图片验证码接口；
2. 服务器返回图片验证码；
3. 使用正确的图片验证码，请求短信验证码；
4. 服务器调用短信运营商的接口，发送短信至用户手机；
5. 通过正确的短信验证码，请求用户注册接口；
6. 完成注册流程。

上面的流程中，我们需要下面三个资源

- captchas —— 图片验证码
- verificationCodes —— 短信验证码
- users —— 用户

对应着三个资源，我们设计出对应的三个接口

- POST api/captchas 创建图片验证码
- POST api/verificationCodes 发送短信验证码
- POST api/users 用户注册







1. 某个用户的信息 —— /users/{user}；

2. 当前登录用户的信息 —— /user

   注意两个接口一个是单数，一个是复数。 user 主要参考了 Github 的设计思路 ，可以理解为`我`的意思。

   ~~~php
   Route::middleware('throttle' . config('api.rate_limits.access'))
       ->group(function (){
           // 游客可以访问的接口
           // 某个用户的详情
           Route::get('users/{user}', 'UsersController@show')
               ->name('users.show');
   
           Route::middleware('auth.api')->group(function (){
               // 登陆后可以访问的接口
               // 当前登录用户信息
               Route::get('user', 'UsersController@me')
                   ->name('user.show');
           });
   	});
   
   ~~~



```php
<?php

   namespace App\Http\Controllers\Api;

   //use App\Http\Controllers\Controller;
   use Illuminate\Support\Str;
   use Illuminate\Http\Request;
   use Overtrue\EasySms\EasySms;
   use App\Http\Requests\Api\VerificationCodeRequest;

   class VerificationCodesController extends Controller
   {
       public function store(VerificationCodeRequest $request, EasySms $easySms){
           $phone = $request->phone;     
     // 生成4位随机数，左侧补0
       $code = str_pad(random_int(1, 9999), 4, 0, STR_PAD_LEFT);
   
       try {
           $result = $easySms->send($phone, [
               'template' => [
                   'template' => config('easysms.gateways.aliyun.templates.register'),
                   'date' => [
                       'code' => $code,
                   ],
               ],
           ]);
       }catch (\Overtrue\EasySms\Exceptions\NoGatewayAvailableException $exception){
           $message = $exception->getException('aliyun')->getMessage();
           abort(500, $message ?: '短信发送异常');
       }
       $key = 'verificationCode_' . Str::random(15);
       $expiredAt = now()->addMinutes(5);
       \Cache::put($key, ['phone' => $phone, 'code' => $code]. $expiredAt);
   
       // 将 key 以及 过期时间 返回给客户端。
       return response()->json([
           'key' => $key,
           'expiredAt' => $expiredAt->toDateTimeString(),
       ])->setStatusCode(201);
   }
}


```
用户1的 jwt 认证

~~~
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9sYXJhYmJzLnRlc3QiLCJpYXQiOjE2MjA4MjQ0MDcsImV4cCI6MTYyMDg4NDQwNywibmJmIjoxNjIwODI0NDA3LCJqdGkiOiJPT3ZGQjJZdWZmUTdYNHNDIiwic3ViIjoxLCJwcnYiOiIyM2JkNWM4OTQ5ZjYwMGFkYjM5ZTcwMWM0MDA4NzJkYjdhNTk3NmY3In0.eEup-NWoDuhK_9mi1K7zOAKa30nCZ4wEWROY5GFXRa0
~~~



   



~~~php
php artisan make:controller Api/ImagesController
php artisan make:model Models/Image
php artisan make:request Api/ImageRequest
~~~

