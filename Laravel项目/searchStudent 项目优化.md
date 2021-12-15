#### searchStudent 项目优化

本次优化

+ 查询优化
+ 建立索引

------

##### 安装 Debugbar

使用 Composer 安装：

```php
composer require "barryvdh/laravel-debugbar:~3.2" --dev
```

生成配置文件，存放位置 `config/debugbar.php`：

```php
php artisan vendor:publish --provider="Barryvdh\Debugbar\ServiceProvider"
```

打开 `config/debugbar.php`，将 `enabled` 的值设置为：

```php
'enabled' => env('APP_DEBUG', false),
```

修改完以后，Debugbar 分析器的启动状态将由 `.env` 文件中 `APP_DEBUG` 值决定

------

##### 优化查询语句

优化前查询代码：

```php
    public function store(UserRequest $request){
        $name = $request->name;
        $candidateNumber = $request->candidate_number;

        // 查询表中是否有此学生
        $user = DB::table('users')
                    ->where([
                        ['name', '=', $name],
                        ['candidate_number', '=', $candidateNumber],
                      
                    ])
                    ->orWhere([
                        ['name', '=', $name],
                        ['id_card', '=', $candidateNumber],
                    ])
                    ->first();

        if (is_object($user) and !empty($user)){
            return view('users.show', compact('user'));
        } else{
            return redirect()->back()->withInput()->with('danger', '未查到此学生信息，请重新输入查询信息！');
        }

    }

```

优化条件

sql 语句 `or`

优化思路：

1. 根据接收到的参数位置来判断是否为 考生号 还是 身份证号码
2. 用变量来代替 sql 语句中的查询条件字段

```sql
    public function store(UserRequest $request){
        $name = $request->name;
        $candidateNumber = $request->candidate_number;
        ### 优化开始
        $type = 'candidate_number';
        if (mb_strlen($candidateNumber) !== 14) {
          $type = 'id_card';
        }

        // 查询表中是否有此学生
        $user = DB::table('users')
                    ->where([
                        ['name', '=', $name],
                        [ $type , '=', $candidateNumber],
                    ])
                    ->first();
				### 优化结束
        if (is_object($user) and !empty($user)){
            return view('users.show', compact('user'));
        } else{
            return redirect()->back()->withInput()->with('danger', '未查到此学生信息，请重新输入查询信息！');
        }

    }

```



------

##### 测试1

![image-20211201153709833](https://i.loli.net/2021/12/01/oA3JOn5vSpz26Wx.png)

| 动作                       | SQL 查询优化前  ms | 页面查询优化前  ms | SQL 查询优化后 ms | 页面查询优化后  ms |
| -------------------------- | ------------------ | ------------------ | ----------------- | ------------------ |
| 查询数据表中第 1000 位同学 | 17.49              | 189                | 4.64              | 107                |
| 查询数据表中第 2000 位同学 | 8.89               | 165                | 5.55              | 122                |
| 查询数据表中第 3000 位同学 | 9.55               | 158                | 5.66              | 109                |
| 查询数据表中第 4000 位同学 | 19.44              | 313                | 5.89              | 113                |

可以从结果看出：优化后 sql 语句查询时间平均在 5ms 左右，减少 5 - 15 ms 左右.

优化前 4 位同学查询截图：

![image-20211201154040834](https://i.loli.net/2021/12/01/9RzHY3WFyZT8L2i.png)

![image-20211201154341869](https://i.loli.net/2021/12/01/ljdfEB2hQ91FZRV.png)

![image-20211201154459343](https://i.loli.net/2021/12/01/vFuQGS8j4OxPAps.png)

![image-20211201154550921](https://i.loli.net/2021/12/01/s2496RSAcVHkMK1.png)

优化后 4 位同学查询截图：

![image-20211201161346827](https://i.loli.net/2021/12/01/oCwinsDWYTVZgt4.png)

![image-20211201161436369](https://i.loli.net/2021/12/01/gIEizD91kNeCbjm.png)

![image-20211201161528065](https://i.loli.net/2021/12/01/FrhWXeJlIqadMzg.png)

![image-20211201161615569](https://i.loli.net/2021/12/01/awk3RuobmtCcdZg.png)

##### 优化索引

![image-20211201162831720](https://i.loli.net/2021/12/01/IsthQiP4871g2lM.png)

##### 测试2

经测试 sql 查询时间无多大变化

