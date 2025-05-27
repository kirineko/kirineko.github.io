---
title: PHP-Phalcon中的数据库操作
date: 2016-05-04T00:00:00.000Z
author: kirineko
draft: false
summary: 这篇文章讨论了PHP-Phalcon框架中的数据库操作方法。
tags:
  - php
createTime: 2016-05-04T00:00:00.000Z
permalink: /article/wzw5z4xj/
---

这篇文章讨论了PHP-Phalcon框架中的数据库操作方法。
本文描述了PHP-Phalcon框架中数据库操作方法，主要讨论Phalcon框架的Model组件中的操作方法。更详细的Model介绍请参考:[官方文档](https://docs.phalcon.io/4.0/en/db-models#models)

---

### 1. 连接数据库

在Phalcon框架中，通过在DI中注入db参数来实现数据库的连接和配置，基本的配置方法如下：

```php
use Phalcon\Db\Adapter\Pdo\Mysql as DbAdapter;

$di->set('db', function () {
    return new DbAdapter(array(
        "host"     => "localhost",
        "username" => "root",
        "password" => "",
        "dbname"   => "test"
    ));
});
```
通过在$di中设置'db'的连接属性，包括host,username,password,dbname等属性来获取数据库参数，配置好db之后就可以利用Phalcon中的ORM框架了。

另一种通过配置文件的方法如下:

1.首先需要将数据信息写入配置文件，在Phalcon中支持ini, php, json等三种配置文件形式，以下为ini形式的配置文件。

```
[database]
adapter  = Mysql
host     = localhost
username = root
password =
dbname   = test
```

2.利用配置文件将数据库信息写入DI

```php
$di->set('db', function () use ($config) {
    $config = $config->get('database')->toArray();

    $dbClass = 'Phalcon\Db\Adapter\Pdo\\' . $config['adapter'];
    unset($config['adapter']);

    return new $dbClass($config);
});
```

### 2. 建立数据库表

在MySQL中建立数据库表，如下所示:

```sql
CREATE TABLE `customer` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) DEFAULT NULL,
  `password` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
以上语句建立了一个customer表，包括主键id，以及username和password两个字段。

### 3. 创建模型

根据上述数据库表，创建Customer.php模型类，需要注意的是模型类必须继承Phalcon\MVC\Model类，并且采用与数据库表名一致的驼峰命名法。因此建立如下的Customer类:

```php
class Customer extends \Phalcon\Mvc\Model
{
    //采用默认的对应规则会自动映射数据库表的字段，可以不写
    /**
     *
     * @var integer
     */
    public $id;

    /**
     *
     * @var string
     */
    public $username;

    /**
     *
     * @var string
     */
    public $password;

    /**
     * Returns table name mapped in the model.
     *
     * @return string
     */
    public function getSource()
    {
        return 'customer';
    }
}
```
其实只要满足了对应的命名规则，模型类中的属性是不需要定义的，这里定义的目的是方便开发过程中查看，并且可以提高性能；此外，如果没有采用数据库表名对应的类名进行命名的话，就需要在初始化函数中通过setSource方法进行配置，配置代码:

```php
public function initialize()
{
    $this->setSource("tablename");
}
```

从这里可以看出，在现代的Web开发框架中，都遵循约定大于配置(CoC)的基本原则。只要遵循约定，就可以很方便的进行Web开发。

### 4. 数据库操作
在Phalcon中操作数据库提供了基本的ORM映射方式以及更加复杂的PHQL方式。ORM映射方式支持不写SQL语句直接操作数据库，基本上已经提供了绝大多数使用场景的支持；另一种更为高级的PHQL方式，支持编写Phalcon化的SQL语句来操作数据库。这里不编写SQL语句，直接使用ORM映射本身提供的一切功能。

#### 4.1 添加数据
----------
如果我想添加一条数据，最基本的方式演示如下:

1. 新建一个控制器DatabaseController
2. 在控制器中添加InsertAction
3. 在InsertAction中写入下列代码:

```php
public function insertAction()
{
    $customer = new Customer();

    $customer->username = 'liyi';
    $customer->password = '123456';

    $ret = $customer->save();

    if($ret){
        print_r('插入成功');
    } else {
        print_r('插入失败');
    }
}
```

#### 4.2 查询数据

----------

**1.find方法**

Phalcon中提供了静态方法find，采用find方法可以返回表中符合条件的所有数据:

```php
public function findAction()
{
    $customers = Customer::find();
    $result = [];
    foreach ($customers as $customer) {
        $result[] = [
            'username' => $customer->username,
            'password' => $customer->password,
        ];
    }
    $this->response->setContentType('application/json', 'UTF-8');
    return $this->response->setJsonContent($result);
}
```

调用<Model>类的静态方法find()会返回包含有customer数据表内容的结果集，对于结果集的处理通常采用foreach进行遍历，如代码所示，对遍历的每一个对象调取属性值，然后赋值给关联数组。

**2.findFirst方法**

Phalcon中的findFirst方法与find方法的使用方式几乎无异，其区别在于findFirst方法的返回结果为单值，不需要通过foreach遍历获取每个值，在下面的代码中演示了查询数据库表中username字段值等于'liyi'的记录。

```php
public function findfirstAction()
{
    $customer = Customer::findFirst("username = 'liyi'");
    $result = [
            'username' => $customer->username,
            'password' => $customer->password,
    ];
    $this->response->setContentType('application/json', 'UTF-8');
    return $this->response->setJsonContent($result);
}
```

**3.findBy\<属性\>方法**

在Phalcon框架中支持findBy<属性>的扩展查询，在上例中，可以把条件语句改为findFirstByUsername,同时省略查询条件,结果不变。

```php
$customer = Customer::findFirstByUsername('kirineko');
```

**4.参数化查询语句**

如果需要查询的内容较多，或者是从防止SQL注入的安全角度来考虑，应当使用参数化的查询语句。比如如果需要从数据库中查找username=(@用户输入的值)并且password=(@用户输入的值)的用户，那么就应当使用参数化查询。

下面模拟用户登录的过程，用户输入用户名和密码，然后系统在数据库中进行查找，如果找到则返回欢迎页，如果没有找到，就提示错误信息。

```php
public function loginAction()
{
    $username = $this->request->getPost('username');
    $password = $this->request->getPost('password');

    $conditons = 'username = :username: and password = :password:';
    $parameters = [
        'username' => $username,
        'password' => $password,
    ];
    $ret = Customer::findFirst(
    [
        $conditons,
        'bind' => $parameters,
    ]);

    if($ret){
        print_r('login success');
    } else {
        print_r('login failed');
    }
}
```

#### 4.3 更新数据


----------


更新数据的方法通常是先查询数据，如果能查到数据就对数据进行赋值，然后save即可。现在要更新用户名为kirineko的密码为用户指定的密码,代码如下:

```php
public function updateAction()
{
    $password = $this->request->getPost('password');
    $newpassword = $this->request->getPost('newpassword');

    $conditons = 'username = :username: and password = :password:';
    $parameters = [
        'username' => 'kirineko',
        'password' => $password,
    ];
    $customer = Customer::findFirst([
        $conditons,
        'bind' => $parameters,
    ]);

    if($customer){
        $customer->password = $newpassword;
        $customer->save();
        print_r('更新成功');
    } else {
        print_r('用户名不存在或密码错误');
    }
}
```

#### 4.4 删除数据


----------


Phalcon的提供了delete命令用于删除，现要删除用户名为liyi的数据，代码如下:

```php
public function deleteAction()
{
    $customer = Customer::findFirstByUsername('liyi');

    if($customer){
        $res = $customer->delete();
        if($res) {
            print_r('删除成功');
        } else {
            print_r('删除失败');
        }
    } else {
        print_r('用户不存在');
    }
}
```
