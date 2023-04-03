---
title: Adapter для подключения к базе данных на PHP
layout: post
author: Alexandr Grigoriev
description: Пример разработки адаптера для быстрой смены модуля подключения к базе данных
---

В сегодняшней заметке я рассмотрю как организовать быструю смену модуля подключения к базе данных с помощью адаптера.


## Суть проблемы:

Обычно для подключения к базе данных я использую модуль PDO. Подключение к базе данных происходит следующим образом.

```php

$host = '127.0.0.1';
$db   = 'test';
$user = 'root';
$pass = '';
$port = "3306";
$charset = 'utf8mb4';

$options = [
    \PDO::ATTR_ERRMODE            => \PDO::ERRMODE_EXCEPTION,
    \PDO::ATTR_DEFAULT_FETCH_MODE => \PDO::FETCH_ASSOC,
    \PDO::ATTR_EMULATE_PREPARES   => false,
];
$dsn = "mysql:host=$host;dbname=$db;charset=$charset;port=$port";
try {
     $pdo = new \PDO($dsn, $user, $pass, $options);
} catch (\PDOException $e) {
     throw new \PDOException($e->getMessage(), (int)$e->getCode());
}

```

Однако, что делать если в дальнейшем надо будет использовать не PDO а простой модуль MYSQLI, получается что код подключения придется переписывать, и не только код, но все запросы к базе данных.

Ниже я рассмотрю как можно решить данную проблему.

## Создание адаптера подключения к БД

### Предварительная подготовка

Прежде чем начать проектировать адаптер, необходимо выполнить несколько подготовительный шагов:

#### 1. Создать скелет проекта. 

Для наглядности структура папок проекта будет очень простой:

```
.
├── app // в данной директории будут храниться классы подключения к БД.
└── public
    └── index.php
```

#### 2. Подключить автозагрузку классов

Создаем в корневой папке проекта файл <mark>composer.json</mark>, подключаем автозагрузку.
<div class="filename">composer.json</div>
```json
{
    "autoload": {
        "psr-4": {
            "App\\": "app/"
        }
    }
}
```
Выполняем автозагрузку классов. 
<div class="filename">bash</div>
```
composer dump-autoload
```
После выполнения команды в корневой директории вашего проекта должа появиться папка vendor. Далее необходимо подключить автозагрузку для этого в файле <mark>index.php</mark> подключаем файл <mark>autoload.php</mark>
<div class="filename">index.php</div>
```php
<?php 

require_once("../vendor/autoload.php");

```

#### 3. Создать базу данных и таблицу 

Так как мы работаем с базой данных, нам необходимо будет протестировать работу адаптера.

Переходим в phpmyadmin, создаем тестовую базу данных test, внутри которой создаем таблицу names и заполняем ее данными.
<div class="filename">mysql</div>
```mysql
CREATE TABLE names (
id INT AUTO_INCREMENT,
name VARCHAR(255) NOT NULL);

INSERT INTO names (name) VALUES ('one'), ('two'), ('three');
```

## Создаем Адаптер подключения

Так как всегда необходимо стараться проектировать на основе интерфейса, создадим папку Adapter, внутри которой создадим интерфейс AdapterInterface. Наш интерфейс будет содержать два метода:

- connect(Config $config), который будет отвечать за подключение к базе данных, 
- fetch($sql), который будет отвечать за выполнение запросов к БД.
<div class="filename">AdapterInterface.php</div>
```php

<?php

namespace App\Adapter;

use App\Adapter\Config;

interface AdapterInterface
{
    public function connect(Config $config);
    public function fetch($sql);
}

```
Создаем класс Config. Данный класс будет содержать только свойства для подключения к базе данных.
<div class="filename">Config.php</div>
```php

<?php

namespace App\Adapter;

class Config
{
    public $driver = "Pdo";
    public $host = "192.168.56.56";
    public $user = "homestead";
    public $password = "secret";
    public $db = "test";

}
```

где, 

- $driver - драйвер для подключения к БД.
- $host - хост подключения (так как я использую homestead, у меня хост 192.168.56.56, у вас это может быть localhost).
- $user - имя пользователя для подключения к базе данных.
- $password - пароль для подключения к базе данных.
- $db - база данных.

Создаем класс подключения к базе данных PDO Pdo.php. описываем метод подключения connect();
<div class="filename">Pdo.php</div>
```php
<?php

namespace App\Adapter;

use App\Adapter\AdapterInterface;
use App\Adapter\Config;

class Pdo implements AdapterInterface
{
    protected $dbh;

    const OPTIONS = [\PDO::ATTR_ERRMODE => \PDO::ERRMODE_EXCEPTION];

    public function connect($config)
    {
        try{
            $dsn = sprintf("mysql:host=%s;dbname=%s", $config->host, $config->db);
            
            $this->dbh = new \PDO($dsn, $config->user, $config->password);
        }catch(\PDOException $e){
            throw new \PDOException($e->getMessage(), (int)$e->getCode());
        }
    }
}
```
Описываем метод для выполнения запросов к базе данных
<div class="filename">Pdo.php</div>
```php
public function fetch($sql)
{
	       try{
            $stmt = $this->dbh->prepare($sql);
            $stmt->execute();
            return $stmt->fetchAll(\PDO::FETCH_ASSOC);
        } catch(\PDOException $e){
            throw new PDOException($e->getMessage(), (int)$e->getCode());
        }
}

```
В корневой директории проекта создаем фабрику для подключения к базе данных <mark>DBFactory.php</mark>. Данный класс будет отвечать за создание модуля подключения к базе данных.
<div class="filename">DBFactory.php</div>
```php
<?php

namespace App;

use App\Adapter\Config;

class DBFactory
{
    public static function connect(Config $config)
    {
        $className = "App\\Adapter\\".$config->driver;
        
        if(class_exists($className)){
            $adapter = new $className();
        }
        $adapter->connect($config);
        return $adapter;
    }
}
```

Все готово. Проверяем работу класса. Для этого в файле index.php создаем экземпляр класса Config и передаем его в метод connect класса DBFactrory.

<div class="filename">index.php</div>
```php
<?php

require_once "../vendor/autoload.php";

use App\DBFactory;
use App\Adapter\Config;


$config = new Config;

$db = DBFactory::connect($config);

$result = $db->fetch("SELECT * FROM names");
```

Получаем все записи из базы данных, выводим результат <mark>var_dump($result);</mark>

Теперь представьте, что нам необходимо вместо PDO использовать другой модуль подключения к базе данных, например mysqli.

В директории App/Adapter создаем новый файл Mysqli.php, который также как и класс Pdo будет реализовывать интерфейс AdapterInterface.

Также описываем в нем 2 метода connect(Config $config), fetch($sql)

<div class="filename">Mysqli.php</div>
```php
class Mysqli implements AdapterInterface
{
    protected $dbh;

    public function connect(Config $config)
    {
        try{
            $this->dbh = new \mysqli($config->host, $config->user, $config->password, $config->db);
        } catch(\Exception $e){
            throw new \Exception($e->getMessage(), (int)$e->getCode());
        }
        
         
    }

    public function fetch($sql)
    {
        $result = $this->dbh->query($sql);
        $data = $result->fetch_all(MYSQLI_ASSOC);
        return $data;
    }
}
```
В файле Config.php меняем свойство $driver на <mark>mysqli</mark>;
<div class="filename">Config.php</div>
```
   public $driver = "Mysqli";
```

Проверяем работу. [Исходники к статье](https://github.com/Flaaim/DBAdapter).

