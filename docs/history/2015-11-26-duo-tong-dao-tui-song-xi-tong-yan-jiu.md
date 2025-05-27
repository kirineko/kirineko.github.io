---
title: 多通道推送系统研究——推送系统接入启示录
date: 2015-11-26T00:00:00.000Z
author: kirineko
draft: false
summary: 上周开发多通道推送遇到了很多'惊喜'，在这里总结一下
tags:
  - php
  - push
createTime: 2015-11-26T00:00:00.000Z
permalink: /article/lemaqmne/
---

上周开发多通道推送遇到了很多"惊喜"，在这里总结一下：

>本系列共分为3篇，该篇为第一篇：基本原理篇。

### 基本原理篇

先来张图好了:

![基本原理](/images/in-post/3.bmp)

1. Bind用户推送数据:首先移动客户端执行bind操作，将客户端相关信息比如uuid,push_cid,user_id,user_type,app_type,user_role等信息绑定到服务器，写入数据库表user_push.以上信息由客户端生成，并在用户登陆app时写入数据库。

2. 服务器推送: 服务器根据user_id,user_role,app_type查询user_push表，找到push_channel和push_cid,根据push_channel选择进入哪一条push通道，通过push_cid告知第三方推送服务push到哪台设备的客户端。

3. 调用第三方推送: 第三份推送本质上提供的是HTTP Web Service接口，并在此基础上封装了专用的SDK供开发者调用，不同服务所封装的sdk也不尽相同。但基本是类似的，第三方推送服务需要从服务器端验证appKey，secretKey等信息，确保是在它那里注册的服务器发送的推送请求，然后从服务器读取cid，message，options等信息。

4. 第三方推送执行推送: 第三方推送根据读取的cid，message，options等信息，决定推送的消息类型，推送的消息内容，推送的过期保存时间，是否重发等。

### 代码解释

#### Step 1 : 查询数据库获取push_cid

``` php
$push_channel = $this->push_channel;
$push = UserPush::findFirst("user_id='{$user_id}' AND role='{$user_role}' AND app_type = '{$app_type}' AND push_channel = '{$push_channel}'");

$this->push_params = [$push['push_cid'],$msg,$opts];
```

- `UserPush`是Phalcon框架中的Model类，提供了一些操作数据库的方法，同时也支持PDO访问数据库
- `findFirst()`方法以对象形式返回模型数据查询的第一条结果，参数为查询条件。
- 将$push_cid,$msg,$opts三个参数信息传给制定的第三方sdk

#### Step 2 : 初始化第三方推送

``` php
$config = \Phalcon\DI::getDefault()->getConfig()->huawei_push->toArray();
$this->push_config = $config[$app_type][$device_name][$develop_status];
$this->app_id = $this->push_config['appId'];
$this->app_secret = $this->push_config['appSecret'];

$this->initSdkConf($app_type,$device_type,$develop_status);
$this->sdk = (new HwPushSDK($this->push_config['appId'], $this->push_config['appSecret']));
```

- 初始化第三方sdk，在Phalcon框架中\Phalcon\DI::getDefault()获取di，需要先把相关配置文件路径写入di，然后通过di调用getConfig获取配置文件中的数组
- 获取到配置信息以后，就可将appId，appSecret等信息读出用于初始化第三方sdk

#### Step 3 : 第三方推送初始化

``` php
/**
 * HwPushSDK constructor.
 * @param string $app_id
 * @param string $app_secret
 */
public function __construct($app_id = '', $app_secret = ''){
    $this->appId = $app_id;
    $this->appSecret = $app_secret;
    $this->push_uri =\Phalcon\DI::getDefault()->getConfig()->hw_push_url;
}
```

- 第三方sdk构造函数，主要用来读取appKey和secretKey进行服务端验证

#### Step 4 : 调用第三方SDK执行推送

```php
$message->title($title);
$message->description($desc);
$message->payload(json_encode($msg));
$message->extra('custom',json_encode($msg));
$message->build();

$targetMessage = new TargetedMessage();
$targetMessage->setTarget($regId, TargetedMessage::TARGET_TYPE_REGID);
$targetMessage->setMessage($message);

$result =  $sender->send($message, $regId, 0);
```

- 根据第三方SDK进行推送
- 小米推送需要区分消息类型：透传还是通知栏
- 设定description，title，payload，extra等信息
- 最后小米推送将根据message，regId进行推送
