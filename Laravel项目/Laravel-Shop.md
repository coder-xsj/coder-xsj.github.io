# Laravel-Shop

### 舞台布置

##### 辅助函数

app/helpers.php

~~~php
function test(){
	return 'ok';
}
~~~

> php artisan tinker

`>>>`test()

会报错

原因是还没有引入 `helpers.php`

可以使用  `composer`  的  `dumpautoload` 来引入

1. composer.json 中的 autoload 选项  `files` 加入 `helpers.php`

   ~~~php
   "autoload": {
           "psr-4": {
               "App\\": "app/",
               "Database\\Factories\\": "database/factories/",
               "Database\\Seeders\\": "database/seeders/"
           },
           "files": [
               "app/helpers.php"
           ]
       },
   ~~~
2. `composer dumpautoload`

3. 重新测试即可

   ![image-20210912105146025](https://i.loli.net/2021/09/12/tWE765Qozmr9xeX.png)



#####  Composer

> composer 配置安装加速

```php
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```
> 查看 composer 全局配置

```php
composer config -l -g
```

##### 1. 创建 laravel 项目

~~~bash
composer create-project laravel/laravel laravel-shop --prefer-dist "8.*"
~~~

##### npm

> npm 配置安装加速

~~~
npm config set registry https://registry.npm.taobao.org
~~~
> 查看 npm 全局配置 --- yarn 同理
```bash
npm config list -g
```
##### yarn

> yarn 配置安装加速

~~~
yarn config set registry https://registry.npm.taobao.org
~~~

> 安装依赖

教程中写的是

~~~node
SASS_BINARY_SITE=http://npm.taobao.org/mirrors/node-sass yarn
~~~

但是我不是 `hometstead` 的环境，所以就用不了，改了下面的方法，分两步走

~~~
yarn config set SASS_BINARY_SITE=http://npm.taobao.org/mir
rors/node-sass

yarn install
~~~

设置安装方式是 `node-sass` 

```bash
npm config set sass_binary_site=https://npm.taobao.org/mirrors/node-sass/
npm config set phantomjs_cdnurl=https://npm.taobao.org/mirrors/phantomjs/
npm config set electron_mirror=https://npm.taobao.org/mirrors/electron/
npm config set registry=https://registry.npm.taobao.org
```



##### 2. 集成 BootStrap

```bash
composer require laravel/ui:"^3.0"
```

  引入 BootStrap

```bash
php artisan ui vue
```



#####  3. 运行 laravel-mix

```bash
npm run wathc-poll
> @ watch-poll C:\Users\XuShengjin\PhpstormProjects\laravel-shop
> mix watch -- --watch-options-poll=1000

[webpack-cli] Error: Cannot find module 'webpack/lib/rules/DescriptionDataMatcherRulePlugin'

```

发现报错缺少 `webpack/lib/rules/DescriptionDataMatcherRulePlugin`,  经多方百度发现一下解决方案
> 安装以下几个包解决问题

~~~bash
npm i vue-loader
npm install --save-dev webpack-cli
npm install webpack-cli
npm install webpack  --save-dev
npm install node-sass
yarn add node-sass
~~~

继续测试下

```bash
npm run wathc-poll
```

![image-20210924001340872](https://i.loli.net/2021/09/24/KuNhk5Xp2YSzjGg.png)



### 用户模块

#### 1. 登录与注册

##### 2. 收货地址





#### 页面调优

此刻我们的页面存在很大的 **性能隐患**，为了能更直观地看到问题，我们先安装 Laravel 开发者工具类 - [laravel-debugbar](https://github.com/barryvdh/laravel-debugbar)。

##### 安装 Debugbar

使用 Composer 安装：

```php
$ composer require "barryvdh/laravel-debugbar:~3.2" --dev
```



> 新增收货地址

进入 终端

```php
php artinsan tinker
```

调用工厂方法 创建数据

```php
 App\Models\UserAddress::factory()->count(3)->create(['user_id' => 1])
```



##### 收藏商品

收藏商品本质上是用户和商品的多对多关联，因此不需要创建新的模型，只需要增加一个中间表即可：

```
php artisan make:migration create_user_favorite_products_table --create=user_favorite_products
```

> 注意：中间表命名，要越直白越好，名字长点也无所谓。一个简单的判断命名是否合格的方法是 —— 想象自己半年一年以后是否能快速地从数据库表名得知此表的功能。

~~~

~~~



### 5. 购物车 & 订单模块

购物车表设计	

| 字段名称       | 描述        | 类型              | 加索引缘由 |
| -------------- | ----------- | ----------------- | ---------- |
| id             | 自增长 id   | unsigined big int | 主键       |
| user_id        | 用户 id     | unsigined big int | 外键       |
| product_sku_id | 商品 sku id | unsigined big int | 外键       |
| amount         | 商品数量    | unsigned int      | 无         |



创建迁移以及 model

```
php artisan make:model CartItem -m
```

根据上述设计表关系

```php
Schema::create('cart_items', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->unsignedBigInteger('user_id');
            $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
            // $table->unsignedBigInteger('product_sku_id');
            // $table->foreign('product_sku_id')->references('id')->on('product_skus')->onDelete('cascade');
            // foreignId 会自动创建 product_sku_id
            $table->foreignId('product_sku_id')->constrained('product_skus')->onDelete('cascade');
            $table->unsignedInteger('amount');
        });
```

model 建立 关系模型



执行迁移

```php
php artisan migrate
```



创建 `CartController`、 `AddCartRequest`

```
php artisan make:controller CartController
php artisan make:request AddCartRequest
```



 编写 `AddCartRequest` 规则





登陆完之后还返回，商品详情页







##### Font Awesome

一套绝佳的图标字体库和CSS框架



##### 关于购物车问题,应该优化勾选

1. 如果不使用全选按钮，用户全部选中商品之后，全选按钮也应该勾选

2. 如果全选之后，用户去掉一个商品勾选，全选的打勾应该去掉

3. 默认的是商品的全部勾选，但是全选按钮并没有勾选





**打开Git命令页面，执行git命令脚本：修改设置，解除ssl验证**

```
git config --global http.sslVerify "false"
```



新建 `movies`

```
php artisan make:model movie -m
```



`2021_09_27_144937_create_movies_table.php` 编写表结构

```php
public function up()
{
  Schema::create('movies', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->string('title');
    $table->integer('director');
    $table->string('description');
    $table->tinyInteger('rate');
    $table->enum('released', [0, 1]);
    $table->timestamp('release_at');
    $table->timestamps();
  });
}
```

`app\Models\Movie.php` 可填充字段

```php
protected $fillable = [
  'title',
  'director',
  'describe',
  'rate',
  'released',
  'release_at',
];
```

`app\Admin\routes.php` 路由

```php
Route::group([
    'prefix'        => config('admin.route.prefix'),
    'namespace'     => config('admin.route.namespace'),
    'middleware'    => config('admin.route.middleware'),
    'as'            => config('admin.route.prefix') . '.',
], function (Router $router) {
    # 后台
    $router->get('/', 'HomeController@index');
    # 用户管理页面
    $router->get('users', 'UsersController@index');
		...
    $router->resource('movies', 'MoviesController');  //影片管理
 }
});
```

创建 `app\Admin\Controllers\MoviesController.php`

```
php artisan admin:make MoviesController --model=App\Models\Movie
```

```php
<?php

namespace App\Admin\Controllers;

use App\Models\Movie;
use Encore\Admin\Controllers\AdminController;
use Encore\Admin\Form;
use Encore\Admin\Grid;
use Encore\Admin\Show;

class MoviesController extends AdminController
{
    /**
     * Title for current resource.
     *
     * @var string
     */
    protected $title = '电影';

    /**
     * Make a grid builder.
     *
     * @return Grid
     */
    protected function grid()
    {
        $grid = new Grid(new Movie());
        // 第一列显示id字段，并将这一列设置为可排序列
        $grid->column('id', 'ID')->sortable();
        // 第二列显示title字段，由于title字段名和Grid对象的title方法冲突，所以用Grid的column()方法代替
        $grid->column('title', '影片名称');
        // 第三列显示director字段，通过display($callback)方法设置这一列的显示内容为users表中对应的用户名
//        $grid->column('director')->display(function($userId) {
//            return User::find($userId)->name;
//        });
        // 第四列显示为describe字段
        $grid->column('description', '简介')->limit(50);
         // 第五列显示为rate字段
        $grid->column('rate', '评分');
        // 第六列显示released字段，通过display($callback)方法来格式化显示输出
        $grid->column('released', '上映')->display(function ($released) {
            return $released ? '是' : '否';
        });
        // 下面为三个时间字段的列显示
        $grid->column('release_at', '上映时间');
        $grid->column('created_at', '创建时间');
        $grid->column('updated_at', '更新时间');
        // filter($callback)方法用来设置表格的简单搜索框
        $grid->filter(function ($filter) {
            // 设置created_at字段的范围查询
            $filter->between('created_at', 'Created Time')->datetime();

            $filter->where(function ($query) {
                $query->where('title', 'like', "%{$this->input}%")
                    ->orWhere('description', 'like', "%{$this->input}%");
            }, 'Text');
            $filter->expand();
        });
        return $grid;
    }

    /**
     * Make a show builder.
     *
     * @param mixed $id
     * @return Show
     */
    protected function detail($id)
    {
        $show = new Show(Movie::findOrFail($id));

        $show->field('id', 'ID');
        $show->field('title', '标题');
        $show->field('description', '简介');
        $show->field('rate', '评分');
        $show->field('created_at', '创建时间');
        $show->field('updated_at', '更新时间');
        $show->field('release_at', '上映时间');

        return $show;
    }

    /**
     * Make a form builder.
     *
     * @return Form
     */
    protected function form()
    {
        $form = new Form(new Movie());
        // 显示记录id
        $form->display('id', 'ID');
        // 添加text类型的input框
        $form->text('title', '电影标题');
        $directors = [
            1 => '陈凯歌',
            2 => 'Smith',
            3 => 'Kate' ,
        ];
        $form->select('director', '导演')->options($directors);
        // 添加describe的textarea输入框
        $form->textarea('description', '简介');
        // 数字输入框
        $form->number('rate', '评分');
        // 添加开关操作
        $states = [
            'on'  => ['value' => '1', 'text' => '上映', 'color' => 'success'],
            'off' => ['value' => '0', 'text' => '下架', 'color' => 'danger'],
        ];
        $form->switch('released', '发布')->states($states)->default('1');
        // 添加日期时间选择框
        $form->datetime('release_at', '上映时间');
        // 两个时间显示
        $form->display('created_at', '创建时间');
        $form->display('updated_at', '修改时间');

        return $form;
    }
}

```

首页

![](https://i.loli.net/2021/09/28/b6CNMrRF5wy3YHS.png)

创建影片

![image-20210928005059608](https://i.loli.net/2021/09/28/GktdIcQxi5RFah8.png)

显示影片 --- 我觉得可以换个模板

![image-20210928005354678](https://i.loli.net/2021/09/28/1iXQZCjPUa72zAh.png)



1. 想法：可以加一个图片

创建 迁移文件

```php
php artisan make:migration add_image_to_movies_table --table=movies
```

`2021_09_27_165625_add_image_to_movies_table.php` 添加字段

```php
public function up()
{
  Schema::table('movies', function (Blueprint $table) {
    $table->string('image');
  });
}
```

执行迁移

```php
php artisan migrate
```

`app\Models\Movie.php` 补充填充字段

```php
protected $fillable = [
  'title',
  'director',
  'describe',
  'rate',
  'released',
  'release_at',
  'image',
];
```

在`app\Admin\Controllers\MoviesController.php` 补充显示字段

 

#### 订单模块

​	表设计

​		因为一笔订单支持多个 SKU 商品，则需要 `orders`, `order_items` 两张表

`orders` 保存用户、金额、收货地址等信息

`order_items` 保存商品 sku id，数量，以及和 `orders` 表关联

`orders` 表字段

| 字段名称        | 描述              | 类型               | 加索引缘由 |
| --------------- | ----------------- | ------------------ | ---------- |
| id              | 自增长 ID         | unsigned big int   | 主键       |
| no              | 订单流水号        | varchar            | 唯一       |
| user_id         | 下单用户的 ID     | unsigned big int   | 外键       |
| address         | JSON 格式收货地址 | text               | 无         |
| total_account   | 订单总金额        | decimal            | 无         |
| remark          | 订单备注          | varchar            | 无         |
| paid_at         | 支付时间          | datetime, null     | 无         |
| payment_method  | 支付方式          | varchar, null      | 无         |
| payment_no      | 支付平台的订单号  | varchar, null      | 无         |
| refounds_status | 退款状态          | varchar            | 无         |
| refound_no      | 退款单号          | varchar, null      | 唯一       |
| closed          | 订单是否关闭      | tinyint, default 0 | 无         |
| reviewed        | 用户是否评价      | tinyint, default 0 | 无         |
| ship_status     | 物流状态          | varchar            | 无         |
| ship_data       | 物流数据          | text, null         | 无         |
| extra           | 其它额外数据      | text, null         | 无         |

收货地址 `JSON` 格式

`order——items` 表字段

| 字段名称       | 描述            | 类型               | 加索引缘由 |
| -------------- | --------------- | ------------------ | ---------- |
| id             | 自增长 ID       | unsigned big int   | 主键       |
| order_id       | 所属订单 ID     | unsigned big int   | 外键       |
| product_id     | 对应商品 ID     | unsigned big int   | 外键       |
| product_sku_id | 对应商品 SKU ID | unsigned big int   | 外键       |
| amount         | 数量            | unsigned int       | 无         |
| price          | 单价            | decimal            | 无         |
| rating         | 用户评分        | unsigned int, null | 无         |
| review         | 用户评价        | text               | 无         |
| reviewed_at    | 评价时间        | timestamp          | 无         |



> 创建模型

```php
php artisan make:model Order -mf
php artisan make:model OrderItem -mf  
```

​	`orders`

~~~php
Schema::create('orders', function (Blueprint $table) {
  $table->bigIncrements('id');
  $table->string('no')->unique();
  $table->unsignedBigInteger('user_id');
  $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
  $table->text('address');
  $table->decimal('total_account', 10, 2);
  $table->text('remark');
  $table->dateTime('paid_at')->nullable();
  $table->string('payment_method')->nullable();
  $table->string('payment_no')->nullable();
  $table->string('refunds_status')->default(\App\Models\Order::REFUND_STATUS_PENDING);
  $table->string('refund_no')->unique()->nullable();
  $table->boolean('closed')->default(false);
  $table->boolean('reviewed')->default(false);
  $table->string('ship_status')->default(\App\Models\Order::SHIP_STATUS_PENDING);
  $table->text('ship_data')->nullable();
  $table->text('extra')->nullable();
  $table->timestamps();
});
~~~

`order_items`

```php
Schema::create('order_items', function (Blueprint $table) {
  $table->bigIncrements('id');
  $table->unsignedBigInteger('order_id');
  $table->foreign('order_id')->references('id')->on('orders')->onDelete('cascade');
  $table->unsignedBigInteger('product_id');
  $table->foreign('product_id')->references('id')->on('products')->onDelete('cascade');
  $table->unsignedBigInteger('product_sku_id');
  $table->foreign('product_sku_id')->references('id')->on('product_skus')->onDelete('cascade');
  $table->unsignedInteger('amount');
  $table->decimal('price', 10, 2);
  $table->unsignedInteger('rating')->nullable();
  $table->text('review')->nullable();
  $table->timestamp('reviewed_at')->nullable();

});
```

`app\models\Order`

```php
		const REFUND_STATUS_PENDING = 'pending';
    const REFUND_STATUS_APPLIED = 'applied';
    const REFUND_STATUS_PROCESSING = 'processing';
    const REFUND_STATUS_SUCCESS = 'success';
    const REFUND_STATUS_FAILED = 'failed';

    const SHIP_STATUS_PENDING = 'pending';
    const SHIP_STATUS_DELIVERED = 'delivered';
    const SHIP_STATUS_RECEIVED = 'received';

    public static $refundStatusMap = [
        self::REFUND_STATUS_PENDING    => '未退款',
        self::REFUND_STATUS_APPLIED    => '已申请退款',
        self::REFUND_STATUS_PROCESSING => '退款中',
        self::REFUND_STATUS_SUCCESS    => '退款成功',
        self::REFUND_STATUS_FAILED     => '退款失败',
    ];

    public static $shipStatusMap = [
        self::SHIP_STATUS_PENDING   => '未发货',
        self::SHIP_STATUS_DELIVERED => '已发货',
        self::SHIP_STATUS_RECEIVED  => '已收货',
    ];

    protected $fillable = [
        'no',
        'address',
        'total_amount',
        'remark',
        'paid_at',
        'payment_method',
        'refund_status',
        'refund_no',
        'closed',
        'reviewed',
        'ship_status',
        'ship_data',
        'extra',
    ];

    protected $casts = [
      'closed' => 'boolean',
      'reviewed' => 'boolean',
      'address' => 'json',
      'ship_data' => 'json',
      'extra' => 'json',
    ];

    protected $dates = [
        'paid_at'
    ];

    protected  static function boot() {
        parent::boot();
        // 监听模型创建事件，在写入数据库之间触发
        static::creating(function ($model) {
            // 如果模型的 no 字段为空
            if (!$model->no) {
                // 生成订单号
                $model->no = static::findAvailableNo();
                // 如果生成失败，则终止创建订单
                if (!$model->no) {
                    return false;
                }
            }
        });
    }

    // 反向关联
    // 拥有此订单的用户
    public function user() {
        return $this->belongsTo(User::class);
    }

    // 订单中可以有多个商品
    public function items() {
        return $this->hasMany(OrderItem::class);
    }

    public static function findAvailableNo() {
        // 订单流水号前缀
        $prefix = date('YmdHis');
        for ($i = 0; $i < 10; $i++) {
            // 生成随机 6 位置数字
            $no = $prefix.str_pad(random_int(0, 999999), 6, '0', STR_PAD_LEFT);
            // 判断是否存在
            if (!static::query()->where('no', $no)->exists()) {
                return $no;
            }
        }
        \Log::warning('find order no faild');
        return false;
    }
```

`app\Moldes\OrderItem`

```php
protected $fillable = [
        'amount',
        'price',
        'rating',
        'review',
        'reviewed_at'
    ];

    protected $dates = ['reviewed_at'];
    public $timestamps = false;

    public function product() {
        return $this->belongsTo(Product::class);
    }

    public function productSku() {
        return $this->belongsTo(ProductSku::class);
    }

    public function order() {
        return $this->belongsTo(Order::class);
    }

```



创建任务

```php
php artisan make:job CloseOrder
```



启动	`redis`

> 1. 临时启动

```
d:
cd phpstudy\Extensions\redis3.0.504
redis-server.exe  redis.windows.conf
```

> 2. 永久启动

```
d:
cd phpstudy\Extensions\redis3.0.504

redis-server --service-install redis.windows-service.conf --loglevel verbose
# 新建 logs 文件夹
# 给 redis3.0.504 只读权限去掉
redis-server --service-start


```

另一种不知道其不起作用

```
net start redis
```



测试 `redis`

`public\redis.php`

```php
<?php
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379);
    $redis->set('name', 'xsj');
    echo $redis->get('name');
```



```sql
  {
            "name": "laravel/horizon",
            "version": "v5.6.2",
            "source": {
                "type": "git",
                "url": "https://github.com/laravel/horizon.git",
                "reference": "26b737da35fd3eaa478f4e7572added44c31fb73"
            },
            "dist": {
                "type": "zip",
                "url": "https://api.github.com/repos/laravel/horizon/zipball/26b737da35fd3eaa478f4e7572added44c31fb73",
                "reference": "26b737da35fd3eaa478f4e7572added44c31fb73",
                "shasum": ""
            },
            "require": {
                "ext-json": "*",
                "ext-pcntl": "*",
                "ext-posix": "*",
                "illuminate/contracts": "^8.0",
                "illuminate/queue": "^8.0",
                "illuminate/support": "^8.0",
                "nesbot/carbon": "^2.17",
                "php": "^7.3|^8.0",
                "ramsey/uuid": "^4.0",
                "symfony/error-handler": "^5.0",
                "symfony/process": "^5.0"
            },
            "require-dev": {
                "mockery/mockery": "^1.0",
                "orchestra/testbench": "^6.0",
                "phpunit/phpunit": "^9.0",
                "predis/predis": "^1.1"
            },
            "suggest": {
                "ext-redis": "Required to use the Redis PHP driver.",
                "predis/predis": "Required when not using the Redis PHP driver (^1.1)."
            },
            "type": "library",
            "extra": {
                "branch-alias": {
                    "dev-master": "5.x-dev"
                },
                "laravel": {
                    "providers": [
                        "Laravel\\Horizon\\HorizonServiceProvider"
                    ],
                    "aliases": {
                        "Horizon": "Laravel\\Horizon\\Horizon"
                    }
                }
            },
            "autoload": {
                "psr-4": {
                    "Laravel\\Horizon\\": "src/"
                }
            },
            "notification-url": "https://packagist.org/downloads/",
            "license": [
                "MIT"
            ],
            "authors": [
                {
                    "name": "Taylor Otwell",
                    "email": "taylor@laravel.com"
                }
            ],
            "description": "Dashboard and code-driven configuration for Laravel queues.",
            "keywords": [
                "laravel",
                "queue"
            ],
            "support": {
                "issues": "https://github.com/laravel/horizon/issues",
                "source": "https://github.com/laravel/horizon/tree/v5.6.2"
            },
            "time": "2020-12-15T19:04:00+00:00"
        },
        
        
        
        //        "laravel/horizon": "~5.6",
```

