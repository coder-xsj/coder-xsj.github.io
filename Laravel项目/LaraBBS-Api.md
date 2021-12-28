# LaraBBS-Api

### 发送验证码

```php
$sms = app('easysms');
try {
    $sms->send(18196761224, [
         'template' => 'SMS_230640326',
         'data' => [
             'code' => 1234
         ],
    ]);
} catch (\Overtrue\EasySms\Exceptions\NoGatewayAvailableException $exception) {
    $message = $exception->getException('aliyun')->getMessage();
    dd($message);
}
```



