# LaraBBS-WEAPP —— 小程序

~~~
git clone git@github.com:coder-xsj/larabbs-1.git
~~~

创建github responsitory 

~~~
git clone git@github.com:coder-xsj/larabbs-weapp.git
~~~

WePY 构建的微信小程序 larabbs-weapp

### 小程序信息

AppID(小程序ID)wx11cc205dc5a34f83

AppSecret(小程序密钥)088dd648106c859c50cc98c40558cdae



### larabbs-weapp 接口介绍

#### 用户登录

| 功能                    | 请求方法 | 接口                    | 是否实现     |
| ----------------------- | -------- | ----------------------- | ------------ |
| 小程序用户登录          | POST     | /weapp/authorizations   | 本教程中实现 |
| Token 刷新              | PUT      | /authorizations/current | 已实现       |
| 退出登录 （Token 删除） | DELETE   | /authorizations/current | 已实现       |

#### 用户注册

| 功能           | 请求方法 | 接口               | 是否实现     |
| -------------- | -------- | ------------------ | ------------ |
| 获取图片验证码 | POST     | /captchas          | 已实现       |
| 获取短信验证码 | POST     | /verificationCodes | 已实现       |
| 小程序用户注册 | POST     | /weapp/users       | 本教程中实现 |

#### 用户个人信息

| 功能         | 请求方法 | 接口              | 是否实现     |
| ------------ | -------- | ----------------- | ------------ |
| 登录用户信息 | GET      | /user             | 已实现       |
| 修改个人信息 | PUT      | /user             | 本教程中修改 |
| 上传头像     | POST     | /images           | 已实现       |
| 权限列表     | GET      | /user/permissions | 已实现       |

#### 话题

| 功能             | 请求方法 | 接口              | 是否实现 |
| ---------------- | -------- | ----------------- | -------- |
| 分类列表         | GET      | /categories       | 已实现   |
| 话题列表         | GET      | /topics           | 已实现   |
| 某个用户话题列表 | GET      | /users/:id/topics | 已实现   |
| 删除话题         | DELETE   | /topics/:id       | 已实现   |

#### 话题回复

| 功能               | 请求方法 | 接口                     | 是否实现     |
| ------------------ | -------- | ------------------------ | ------------ |
| 回复列表           | GET      | /topics/:id/replies      | 已实现       |
| 某个用户的回复列表 | GET      | /users/:id/replies       | 已实现       |
| 发布回复           | POST     | /topics/:id/replies      | 已实现       |
| 删除回复           | DELETE   | /topics/:id/topics       | 已实现       |
| 消息通知列表       | GET      | /user/notifications      | 已实现       |
| 标记通知已读       | PUT      | /user/read/notifications | 本教程中修改 |

~~~
app-id：wx3e6f580471702e2f
app-secert：b975d96745e4edf49b39427684953120
~~~



###  `WePy` 生成了一些基础代码。

WePY 文件结构简介：

| 文件夹名称          | 类型 | 简介                                                |
| ------------------- | ---- | --------------------------------------------------- |
| src                 | 目录 | 源码文件                                            |
| src/app.wpy         | 目录 | 项目入口文件                                        |
| src/pages           | 目录 | 存放小程序页面                                      |
| src/components      | 目录 | 存放小程序组件                                      |
| src/mixins          | 目录 | 存放 Mixin 文件                                     |
| node_modules        | 目录 | NPM 依赖模块                                        |
| wepy.config.js      | 文件 | 全局配置文件                                        |
| yarn.lock           | 文件 | 依赖列表，确保这个应用的副本使用相同版本的依赖      |
| package.json        | 文件 | 项目的 package 配置                                 |
| project.config.json | 文件 | 开发者工具配置                                      |
| .wepyignore         | 文件 | WePY 忽略的文件                                     |
| .wepycache          | 文件 | WePY 缓存文件，防止在 build 时，重复 build npm 目录 |
| .prettierrc         | 文件 | prettier 配置文件                                   |
| .eslintrc.js        | 文件 | eslint 配置文件                                     |
| .eslintignore       | 文件 | eslint 忽略的文件                                   |
| .editorconfig       | 文件 | 编辑器配置文件                                      |





### 项目开始

#### 安装 WePY-cli

全局安装 `@wepy/cli`

~~~
npm install @wepy/cli -g
~~~

查看版本

~~~
wepy -v
~~~

#### 初始化项目

~~~
cd larabbs-weapp
wepy init standard#2.0.x ./
~~~

#### 安装依赖

~~~
npm install
~~~

#### 小程序获取 Code

调用 `wx.login()` 接口获取 `code`

~~~js
src/app.wpy
wepy.app({
  onLaunch() {
    wx.login({
      success(res){
        console.log(res)
      }
    })
  }
})
~~~

~~~
{
	errMsg: "login:ok", 
	code: "023r2tFa1FwO2B0iN7Ga1mYURM0r2tFp"
}

~~~

使用 Promise 方法 获取  `code`

安装

~~~
npm install @wepy/use-promisify --save
~~~

src/app.wpy

~~~js
...
import promisify from '@wepy/use-promisify';  
wepy.use(vuex);
wepy.use(promisify);
...
~~~

调用

~~~js
src/app.wpy
wepy.app({
  onLaunch() {
    wepy.wx.login().then(res => {
    	console.log('login:', res)
    })
  }
})
~~~

~~~json
login: 
{
    errMsg: "login:ok", 
    code: 0332oNkl2XJL274lI1ll2X42Xr42oNkc"
}

~~~

#### 服务器获取 OpenID

~~~
$miniProgram = \EasyWeChat::miniProgram();
=> EasyWeChat\MiniProgram\Application {#4558}
$miniProgram->auth->session('093lY9000Un7IL1QBB00080tDp4lY90p');
=> [
     "errcode" => 40029,
     "errmsg" => "invalid code, hints: [ req_id: vaHcuUore-xrSbMa ]",
   ]
重新换个 code 调用
$miniProgram->auth->session('0332oNkl2XJL274lI1ll2X42Xr42oNkc');
=> [
     "session_key" => "3dxjf1ncisguYxM8P2/X9g==",
     "openid" => "oqduV4tLyFjKrEn05VyX1rnmryz8",
   ]


~~~

![image-20210516014418707](https://i.loli.net/2021/05/16/ScHQtbCVXwxhEzn.png)

![image-20210516014657736](https://i.loli.net/2021/05/16/zFbJhe3YPCnRSo4.png)









![image-20210516021831730](https://i.loli.net/2021/05/16/gyXhOAPUlEcejdt.png)



![image-20210516022030783](https://i.loli.net/2021/05/16/yODcB1ToEehLRmZ.png)

#### 小程序页面生命周期

+ onLoad: 页面加载 —— 一个页面只会调用一次；
+ onShow: 页面显示 —— 每次打开页面都会调用一次；
+ onReady: 页面初次渲染完成 —— 一个页面只会调用一次，代表页面已经准备妥当，可以和视图层进行交互；
+ onHide: 页面隐藏 —— 当 navigateTo 或底部 tab 切换时调用；
+ onUnload: 页面卸载 —— 当 redirectTo 或 navigateBack 的时候调用。



```sql
access_token: "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9sYXJhYmJzLnRlc3RcL2FwaVwvdjFcL3dlYXBwXC9hdXRob3JpemF0aW9ucyIsImlhdCI6MTYzOTg5MDY5MSwiZXhwIjoxNjM5OTUwNjkxLCJuYmYiOjE2Mzk4OTA2OTEsImp0aSI6IlBLYlBSR2I5VEJTOXhjS3oiLCJzdWIiOjI4LCJwcnYiOiIyM2JkNWM4OTQ5ZjYwMGFkYjM5ZTcwMWM0MDA4NzJkYjdhNTk3NmY3In0.hln8D7yzMeKdHl_w1SMUTCazIs9EXgXP4mp6wow7RcQ"
expires_in: 60000
token_type: "Bearer"
```

