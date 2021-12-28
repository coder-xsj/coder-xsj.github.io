# 

## 1. Request Headers 标准的请求头

| header          | 说明                                                 | 示例                                                         |
| --------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| Accept          | 指定客户端能够接收的内容类型                         | application/json, text/javascript, */*; q=0.01               |
| Accept-Encoding | 客户端可接受处理的编码压缩格式                       | gzip, deflate, br                                            |
| Accept-Language | 客户端可接受的自然语言                               | zh,zh-CN;q=0.9,en;q=0.8                                      |
| Content-Type    | 返回内容的MIME类型,通常只会用在POST和PUT方法的请求中 | application/x-www-form-urlencoded; charset=UTF-8             |
| Content-Length  | 请求体内容长度                                       | Content-Length: 348                                          |
| Cookie          | 客户端向服务器发送请求时发送cookie                   | sc_jwb=ajvKXJdvmHIeI2RXiKNXxXIRZuzZw%2FAEqT2Rn955QA8%3D;     |
| Referer         | 当前请求的 URL是在那个地址引用的                     | A 请求 B，则在 B 返回内容里面，显示 A，通常我们见到的图片防盗链就是用这个实现的 |
| User-Agent      | 用户的浏览器相关信息                                 | Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36 |

## 2. 通用但非标准的 HTTP 头

| header            | 说明                                                         | 示例           |
| ----------------- | ------------------------------------------------------------ | -------------- |
| X-Requested-With  | 主要是用来识别 ajax 请求                                     | XMLHttpRequest |
| X-Forwarded-For   | 记录一个请求从客户端出发到目标服务器过程中经历的代理，或者负载平衡设备的IP。 |                |
| X-Forwarded-Proto | 记录一个请求一个请求最初从浏览器发出时候，是使用什么协议。因为有可能当一个请求最初和反向代理通信时，是使用https，但反向代理和服务器通信时改变成http协议，这个时候，X-Forwarded-Proto的值应该是https |                |



## 3. Response Headers 部分各个字段的功能

| Header                      | 说明                                                 | 示例                                                         |
| :-------------------------- | :--------------------------------------------------- | :----------------------------------------------------------- |
| Access-Control-Allow-Origin | 该站点可以被哪些网站进行 跨域资源共享                | *，http://xushengjin.cn                                      |
| Accept-Ranges               | 表明服务器是否支持指定范围请求及哪种类型的分段请求   | bytes                                                        |
| Allow                       | 可以允许的请求行为，不允许则访问405                  | GET，POST                                                    |
| Age                         | 从原始服务器到代理缓存形成的估算时间（以秒计，非负） | 12                                                           |
| Cache-Control               | 告诉所有的缓存机制是否可以缓存及哪种类型             | no-cache, private                                            |
| Connection                  |                                                      | keep-alive                                                   |
| Content-Encoding            | 服务器支持的返回内容压缩编码类                       | gzip                                                         |
| Content-Language            | 响应体的自然语言                                     | zh,zh-CN;                                                    |
| Content-Length              | 响应体的长度                                         | Content-Length: 348                                          |
| Content-Type                | 返回内容的MIME类型                                   | application/json; charset=utf-8                              |
| Server                      | web服务器软件名称                                    | nginx                                                        |
| Set-Cookie                  | 设置Http Cookie                                      | larabbs_session=eyJpdiI6IlV3ejRuTmZuN0tOWVEvR0ZVL2xJeUE9PS... |
| Date                        | 原始服务器消息发出的时间                             | Wed, 22 Dec 2021 06:01:20 GMT                                |
| ETag                        | 请求变量的实体标签的当前值                           | e7B+l/au02BDHTKTFb2IHkvLKOk=                                 |
| Expires                     | 响应过期的日期和时间                                 | Fri, 09 Dec 2022 12:19:14 GMT                                |
| Transfer-Encoding:          | 文件传输编码                                         | chunked                                                      |
| Vary                        | 告诉下游代理是使用缓存响应还是从原始服务器请求       | Accept-Encoding                                              |



## 4. curl_setopt() 先起效还是后起效

```
curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);
```

