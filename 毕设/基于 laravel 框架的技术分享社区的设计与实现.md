# 基于 laravel 框架的技术分享社区的设计与实现

## 功能修改

- [ ] QQ 邮箱模板 --- 激活、修改密码、站内通知（网站名称）

  ![image-20211112003606448](https://i.loli.net/2021/11/12/AvgJyEls1ekZUdc.png)

- [x] 域名要等备案好

  备案/许可证编号为：皖ICP备2021016299号，审核通过日期：2021-11-17。

  在您的网站首页底部中间位置，放置您的备案号并链接至"http://beian.miit.gov.cn/"。

  公安备案：`皖公网安备 34052102000123号` 2021-11-29 
  
- [x] 图片 url 地址 --- 等上一步

- [x] 提示汉化  

  2021-11-25 21:53

- [x] 资源推荐加上图片

  2021-11-25 23:15

- [x] 考虑是否加上友链 

  2021-11-27 0:47

  

- [ ] 导航栏是否加上**搜索**功能

  ElasticSearch 

  小项目的话 meilisearch 够了 ES 太重

- [ ] 个人中心 --- 关注 --- 话题 --- 评论数量

- [ ] 帖子点赞  --- coffeephp 动画

- [ ] 评论点赞 和 @

- [ ] 帖子内关注功能

- [ ] 最新和热门

- [ ] 第三方登录（QQ，微信）

- [ ] 考虑换个后台模板，用 Dcat-admin --- 但后台功能要重写，工作量有些大

  https://gitee.com/jqhph/dcat-admin
  
- [ ] 帖子内目录功能

  参考掘金 --- https://segmentfault.com/a/1190000040942905

- [ ] 默认头像

  https://en.gravatar.com/

- [ ] 评论 `禁词` 过滤

- [ ] 







## 样式修改

- [ ] 导航栏 click 事件改为 hover 事件

  https://www.guke1.com/#/md 上的`其它工具` hover 效果

- [ ] 编辑器更改为支持 markdown 的编辑器

   [tui.editor](https://github.com/nhnent/tui.editor)

   ![image-20211125140912532](https://i.loli.net/2021/11/25/DezlNWQaPhZ5IYF.png)

- [ ] 导航栏加载条效果

  https://www.bootcss.com/p/metro-ui-css/progress.html

  ui:https://blog.vini123.com/483

- [ ] 返回顶部功能

   效果参考掘金

- [ ] 个人中心左侧背景

   https://segmentfault.com/u/coder_xsj

   ![image-20211111204714801](https://i.loli.net/2021/11/19/4oJMhEU19yZLfTx.png)

- [ ] 友链样式需要修改

	参考 laravel-china
	
- [ ] 


Laravel8+Vue+AntDesign前后端分离开发框架【旗舰版】https://www.rxthink.cn/

非经营性网站，主要是在生活上、学习上、工作上的个人分享，希望大家健康快乐每一天

http://112.124.21.240:81/#

95187 ---> 3





```sql
php artisan vendor:publish --provider="Encore\Admin\AdminServiceProvider"
```

