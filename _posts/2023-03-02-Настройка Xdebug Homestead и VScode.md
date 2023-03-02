---
title: Настройка Xdebug, Homestead и VS code
layout: post
author: Alexandr Grigoriev
---

В сегодняшней заметке рассмотрим как сконфигурировать Xdebug на виртуальной машине и настроить его на VScode.

## 1. Коннектимся к Homestead
<div class="filename">bash</div>

```
ssh vagrant@homestead
```

## 2. Включаем Xdebug в Homestead
После успешного подключения к виртуальной машины, необходимо включить Xdebug. Делается это следующей командой:
<div class="filename">bash</div>
```
xon
```
Проверить запущен ли у вас Xdebug, а также посмотреть используемую php можно командой 
<mark>php -v</mark>

Если у вас включен xdebug, под версией php вы должны увидеть следующую запись
<mark>with Xdebug v3.1.2, Copyright (c) 2002-2021, by Derick Rethans</mark>

## 3. Находим IP адрес шлюза гостевой машины

Данный IP необходим в дальнейшем в настройке конфига. Он будет использоваться для связи вашей гостевой машины с хостом.
<div class="filename">bash</div>
```
netstat -rn | grep "^0.0.0.0 " | cut -d " " -f10
```

У меня выводиться: <mark>10.0.2.2</mark>. Запомните данный IP, он нам понадобиться дальше.

## 4. Находим xdebug.ini
<div class="filename">bash</div>
```
php --ini | grep 'xdebug'
```
Данная команда возвращает путь к вашему xdebug.ini файлу. 

<mark>/etc/php/8.1/cli/conf.d/20-xdebug.ini</mark>

## 5. Настраиваем конфиг xdebug

Редактируем 20-xdebug.ini
<div class="filename">bash</div>
```
sudo vim /etc/php/8.1/cli/conf.d/20-xdebug.ini
```

В редакторе vim откроется файл, в который необходимо внести следующие данные
<div class="filename">20-xdebug.ini</div>
```
zend_extension=xdebug.so
xdebug.mode = debug,develop
xdebug.discover_client_host = 1
xdebug.client_port = 9003
xdebug.max_nesting_level = 512
xdebug.client_host = 10.0.2.2
xdebug.start_with_request = yes
```

## 6. Перезагружаем PHP-FPM
<div class="filename">bash</div>
```
sudo service php8.1-fpm restart
```
Переходим к настройке Vscode.

## Настройка VScode

### 1. Устанавливаем плагин [PHP debug](https://marketplace.visualstudio.com/items?itemName=xdebug.php-debug). 
В VScode в разделе Extensions находим PHP debug, устанавливаем его
### 2. Создаем файл launch.json. 
Переходим в раздел Run and Debug (нажимаем Ctrl + Shift + D), нажимаем на create a <mark>launch.json file</mark>
### 3. Настраиваем launch.json
После нажатия откроется файл конфигурации, в него необходимо добавить следующие строки
<div class="filename">launch.json</div>
```
    {
        "name": "Listen for XDebug on Homestead",
        "type": "php",
        "request": "launch",
        "pathMappings": {
            "/home/vagrant/code/{yourproject}": "${workspaceFolder}"
        },
        "port": 9003,
    },
```

Обратите внимание на строку pathMappings. Внутри ее необходимо указать маппинг путей, где <mark>{yourproject}</mark> это папка с вашим проектом в Homestead

### 4. Включаем Debug

В выпадающем списке выбираем <mark>Listen for XDebug on Homestead</mark>. Далее нажимаем на зеленую стрелочку.
![xdebug-homestead-vscode](/assets/images/articles/xdebug-homestead-vscode/xdebug-homestead-vscode.jpg)

Далее вы должны увидеть следующую панель для управления xdebug.
![xdebug-homestead-vscode](/assets/images/articles/xdebug-homestead-vscode/xdebug-panel.jpg)

### 5. Проверяем работу

Выведем какую-нибудь несуществующую переменную, например <mark>echo $foo</mark>

Обновляем страницу и видим, что xdebug в vscode сообщает нам об ошибке.

![xdebug-homestead-vscode](/assets/images/articles/xdebug-homestead-vscode/xdebug-error.jpg)