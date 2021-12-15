# LaraBBS-WEAPP —— 小程序

~~~
git clone git@github.com:coder-xsj/larabbs-1.git
~~~

创建github responsitory 

~~~
git clone git@github.com:coder-xsj/larabbs-weapp.git
~~~



### larabbs-weapp 接口介绍

## 用户登录[#](https://learnku.com/courses/laravel-weapp/2.0/larabbs-interface-detailed-solution/4925#434798)

| 功能                    | 请求方法 | 接口                    | 是否实现     |
| ----------------------- | -------- | ----------------------- | ------------ |
| 小程序用户登录          | POST     | /weapp/authorizations   | 本教程中实现 |
| Token 刷新              | PUT      | /authorizations/current | 已实现       |
| 退出登录 （Token 删除） | DELETE   | /authorizations/current | 已实现       |

## 用户注册

| 功能           | 请求方法 | 接口               | 是否实现     |
| -------------- | -------- | ------------------ | ------------ |
| 获取图片验证码 | POST     | /captchas          | 已实现       |
| 获取短信验证码 | POST     | /verificationCodes | 已实现       |
| 小程序用户注册 | POST     | /weapp/users       | 本教程中实现 |

## 用户个人信息

| 功能         | 请求方法 | 接口              | 是否实现     |
| ------------ | -------- | ----------------- | ------------ |
| 登录用户信息 | GET      | /user             | 已实现       |
| 修改个人信息 | PUT      | /user             | 本教程中修改 |
| 上传头像     | POST     | /images           | 已实现       |
| 权限列表     | GET      | /user/permissions | 已实现       |

## 话题

| 功能             | 请求方法 | 接口              | 是否实现 |
| ---------------- | -------- | ----------------- | -------- |
| 分类列表         | GET      | /categories       | 已实现   |
| 话题列表         | GET      | /topics           | 已实现   |
| 某个用户话题列表 | GET      | /users/:id/topics | 已实现   |
| 删除话题         | DELETE   | /topics/:id       | 已实现   |

## 话题回复

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

~~~
...
import promisify from '@wepy/use-promisify';  
wepy.use(vuex);
wepy.use(promisify);
...
~~~

调用

~~~
src/app.wpy
wepy.app({
  onLaunch() {
    wepy.wx.login().then(res => {
    	console.log('login:', res)
    })
  }
})
~~~

~~~
login: 
{
    errMsg: "login:ok", 
    code: 0332oNkl2XJL274lI1ll2X42Xr42oNkc"
}

~~~

#### 服务器获取 OpenID

~~~
>>> $miniProgram = \EasyWeChat::miniProgram();
=> EasyWeChat\MiniProgram\Application {#4558}
>>> $miniProgram->auth->session('093lY9000Un7IL1QBB00080tDp4lY90p');
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

~~~
src/pages/user.wpy

<template>
   <div class="page">
       <div class="page__bd">
           <div class="weui-panel weui-panel_access">
               <div class="weui-panel__hd" v-if="loggedIn">
                   已登录
               </div>
               <div v-else>
                   <a class="weui-cell weui-cell_access" url="pages/auth/login">
                       <div class="weui-cell__bd">未登录</div>
                       <div class="weui-cell_access weui-cell__ft"></div>
                   </a>
               </div>
           </div>
       </div>
   </div>
</template>

<script>
    import wepy from '@wepy/core'

    wepy.page({
        config:{
            navigationBarTitleText: '我的'
        },
        data: {
            loggedIn: false
        },
        onShow: {
            if(wx.getStorageSync('access_token')){
                this.loggedIn = true;
            }
        }
    })
</script>


~~~

request.js

~~~
// 请求前显示 loading
import wepy from "@wepy/core";

// 服务器地址
const host = 'http://larabbs.test/api/v1';

// 普通请求
const request = async (url, options = {}, showLoading = true) => {
  // 显示加载
  if (showLoading){
    wx.showLoading({title: '加载中'})
  }

  options.url = host + url

  // 发起网络请求
  let response = await wepy.wx.request(options)

  // 隐藏加载
  if(showLoading){
    wx.hideLoading()
  }

  if(response.statusCode >= 201 && response.statusCode < 300){
    return response
  }

  // 显示模态框
  if(response.statusCode === 429){
    wx.showModal({
      title: '提示',
      content: '请求太频繁，请稍后再试'
    })
  }

  if(response.statusCode === 500){
    wx.showModal({
      title: '提示',
      content: '服务器错误，请联系管理员或重试'
    })
  }

  const error = new Error(response.data.message)
  error.response = response
  return Promise.reject(error)

}

export {
  request
}

~~~

user.wpy

~~~
<template>
   <div class="page">
       <div class="page__bd">
           <div class="weui-panel weui-panel_access">
               <div class="weui-panel__hd" v-if="loggedIn">
                   已登录
               </div>
               <div v-else>
                   <a class="weui-cell weui-cell_access" url="pages/auth/login">
                       <div class="weui-cell__bd">未登录</div>
                       <div class="weui-cell_access weui-cell__ft"></div>
                   </a>
               </div>
           </div>
       </div>
   </div>
</template>

<script>
    import wepy from '@wepy/core'

    wepy.page({
        config:{
            navigationBarTitleText: '我的'
        },
        data: {
            loggedIn: false
        },
        onShow() {
            if(wx.getStorageSync('access_token')){
                this.loggedIn = true;
            }
        }
    })
</script>


~~~

login.wpy

~~~
<style lang="less">
.login-wrap {
  margin-top: 90px;
}
.weui-toptips {
  display: block;
}
</style>
<template>
  <div class="page">
    <div class="page__bd">
      <div class="page__bd login-wrap">
        <div class="weui-toptips weui-toptips_warn fadeIn" v-if="errorMessage">{{ errorMessage }}</div>
        <div class="weui-cells__title">Larabbs 用户登录</div>
        <div class="weui-cells weui-cells_after-title">
          <div class="weui-cell weui-cell_input" :class="{'weui-cell_warn': hasError}">
            <div class="weui-cell__hd">
              <div class="weui-label">用户名</div>
            </div>
            <div class="weui-cell__bd">
              <input class="weui-input" placeholder="手机号或邮箱" v-model="form.username" />
            </div>
            <div v-if="hasError" class="weui-cell__ft">
              <icon type="warn" size="23" color="#E64340"></icon>
            </div>
          </div>
          <div class="weui-cell weui-cell_input" :class="{'weui-cell_warn': hasError}">
            <div class="weui-cell__hd">
              <div class="weui-label">密码</div>
            </div>
            <div class="weui-cell__bd">
              <input class="weui-input" placeholder="输入密码" v-model="form.password" type="password" />
            </div>
            <div v-if="hasError" class="weui-cell__ft">
              <icon type="warn" size="23" color="#E64340"></icon>
            </div>
          </div>
        </div>

        <div class="weui-btn-area">
          <button class="weui-btn" type="primary" @tap="submit">登录</button>
        </div>
      </div>
    </div>
  </div>
</template>
<config>
{
<!--navigationBarTitleText: '登录',-->
}
</config>

<script>
  import wepy from "@wepy/core";
  import {login} from '@/api/auth';

  wepy.page({
    data: {
      // 用户名
      form: {},
      // 是否有错
      hasError: false,
      // 错误信息
      errorMessage: '',
    },
    methods: {
        // 表单提交
      async submit(){
        // 提交时重置错误
        this.hasError = false;
        this.errorMessage = '';

        // 检测用户名和密码是否输入
        if(!this.form.username || !this.form.password){
            this.hasError = true;
            this.errorMessage = '请填写账户名和密码';
            return;
        }
        // 拿到参数
        let params = this.form;
        // 调用微信登录接口
        const loginData = await wx.login();
        // 查看 code 码
        params.code = loginData.code;

        try {
            // 请求登录接口
            const loginResponse = await login(params);
            const accessToken = loginResponse.data.access_token;
            const accessTokenExpiredAt = new Date().getTime() + loginResponse.data.expires_in * 1000;

            wx.setStorageSync('access_token', accessToken);
            wx.setStorageSync('access_token_expired_at', accessTokenExpiredAt);

            wx.navigateBack();

        } catch (err){
            this.hasError = true;
            this.errorMessage = err.response.data.message;
        }
      }
    },
    // 页面打开事件
      async onShow(){
        try {
            // 获取微信登录 code
            const loginData = await wx.login();
            // 请求登录接口
            const loginResponse = await login({
                code: loginData.code
            });

            // 如果找到code对应的用户则保存 token 和过期时间，然后返回
            const accessToken = loginResponse.data.access_token;
            const accessTokenExpiredAt = new Date().getTime() + loginResponse.data.expires_in * 1000;

            wx.setStorageSync('access_token', accessTokenExpiredAt);
            wx.setStorageSync('access_token_expired_at', accessTokenExpiredAt);

            wx.navigateBack();


        }catch (e) {

        }
      }
  })
</script>

~~~

