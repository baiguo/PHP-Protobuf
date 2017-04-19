# PHP-Protobuf
PHP中使用Protobuf简介
Protocol Buffers 是一种轻便高效的结构化数据存储格式，可用于结构化数据串行化，很适合做数据存储或 RPC 数据交换格式。它可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。目前只提供了 C++、Java、Python 三种语言的 API。

官方不支持PHP，不用担心，高手在民间。上Github搜索一下就有了。

liunx 访问上级目录是 ../
比如：
php ../php-protobuf-master/protoc-gen-php.php message.proto
在最新的版本中， 执行protoc的是 protoc-gen-php.php


安装protobuf
wget http://protobuf.googlecode.com/files/protobuf-2.4.1.tar.bz2
tar -jxvf protobuf-2.4.1.tar.bz2 
cd protobuf-2.4.1/
./configure
make
make install
查看protobuf版本

protoc --version
编译php的protobuf扩展
在Google上搜索 protobuf php 可在github上找到两个与之相关的开源项目，它们是：

https://github.com/allegro/php-protobuf
https://github.com/drslump/Protobuf-PHP

在这里主要介绍allegro的编译方法（别问为什么，因为drslump的编译方法在我的电脑上不成功 -_- ||）。

下面介绍centos（CentOS Linux release 7.0.1406 (Core)）环境中下载，编译：

wget https://github.com/allegro/php-protobuf/archive/master.zip
unzip master.zip
cd php-protobuf-master
安装依赖

yum install php-devel
假如phpize不在默认目录，可以使用 which phpize查看，然后写全路径去执行phpize，我这里是默认在 /usr/bin目录，可以直接执行，执行之后可以看到以下输出

[root@VM_18_199_centos php-protobuf-master]# phpize
Configuring for:
PHP Api Version:         20100412
Zend Module Api No:      20100525
Zend Extension Api No:   220100525
php-config的目录也使用which php-config这个命令去查看路径，这一步需要安装gcc

yum -y install gcc-c++
安装完gcc之后则开始正式安装pb

./configure
make
make install
最后会看到类似这样的安装提示，则说明安装成功。

[root@VM_18_199_centos php-protobuf-master]# make install
Installing shared extensions:     /usr/lib64/php/modules/
现已成功把pb编译至/usr/lib64/php/modules/protobuf.so

开启php的pb扩展
找到php.in，加入这行代码：

extension=/usr/lib64/php/modules/protobuf.so
重启nginx

/usr/sbin/nginx -s reload

//重启php-fpm
kill -SIGUSR2 `cat /run/php-fpm/php-fpm.pid`
假如是Apache，则重启Apache

//centos 7.0
systemctl restart httpd
//centos 6.5
service httpd restart
编译proto文件
新建一个简单的proto文件：

vim test.proto
输入以下内容：

message PhoneNumber {
    required string number = 1;
    required int32 type = 2;
  }

message Person {
      required string name = 1;
      required int32 id = 2;
      optional string email = 3;
      repeated PhoneNumber phone = 4;
      optional double money = 5;
}

message AddressBook {
  repeated Person person = 1;
}
编译。注意，这里需要引用到protoc-php.php这个文件，注意路径。

php ./php-protobuf-master/protoc-php.php  test.proto
编译成功后会在当前目录生成一个pb_proto_test.php文件，内容如下：

 /**
 * Auto generated from test.proto at 2015-10-22 23:19:46
 */

/**
 * PhoneNumber message
 */
class PhoneNumber extends \ProtobufMessage
{
    /* Field index constants */
    const NUMBER = 1;
    const TYPE = 2;

    /* @var array Field descriptors */
    protected static $fields = array(
        self::NUMBER => array(
            'name' => 'number',
            'required' => true,
            'type' => 7,
        ),
        self::TYPE => array(
            'name' => 'type',
            'required' => true,
            'type' => 5,
        ),
    );

    /**
     * Constructs new message container and clears its internal state
     *
     * @return null
     */
    public function __construct()
    {
        $this->reset();
    }

    .......

}
这里会继承ProtobufMessage类，它是在protobuf.so中自动加载的。
写一个测试类来测试一下：

<?php
require_once 'pb_proto_test.php';

$foo = new Person();
$foo->setName('hellojammy');
$foo->setId(2);
$foo->setEmail('helloxxx@foxmail.com');
$foo->setMoney(1988894.995);

$phone_num = new PhoneNumber();
$phone_num->setNumber('1351010xxxx');
$phone_num->setType(3);

$foo->appendPhone($phone_num);
//$foo->appendPhone(2);
$packed = $foo->serializeToString();
//echo $packed;exit;
#$foo->clear();
echo "-----------src------------\n";
echo $foo->getName() ."\n";
echo $foo->getPhone()[0]->getNumber() ."\n";
$foo->dump();
echo "------------------------\n\n\n";


try {
      $p = new Person();
      $p->parseFromString($packed);
      echo "------------parsed-------\n";
      echo $p->getName() ."\n";
      echo $p->getEmail() ."\n";
      echo $p->getMoney() ."\n";
      echo $p->getId() . "\n";
      echo $p->getPhone()[0]->getNumber() ."\n";

      //$p->dump();
      echo "------------------------\n";
      //print_r($xiao);
      } catch (Exception $ex) {
      die('Upss.. there is a bug in this example');
}
运行

php text.php
可以看到输出：

-----------src------------
hellojammy
1351010xxxx
Person {
  1: name => 'hellojammy'
  2: id => 2
  3: email => 'helloxxx@foxmail.com'
  4: phone(1) =>
    [0] =>
      PhoneNumber {
        1: number => '1351010xxxx'
        2: type => 3
      }
  5: money => 1988894.995
}
------------------------


------------parsed-------
hellojammy
helloxxx@foxmail.com
1988894.995
2
1351010xxxx
------------------------








