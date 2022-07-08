---
title: Как вручную установить laravel/homestead vagrant box
layout: post
author: Alexandr Grigoriev

---


Рассмотрим способ добавления vagrant box’a laravel/homestead вручную. Данный способ может вам пригодиться если при выполнении команды vagrant box add laravel/homestead вы получаете ошибку как на изображении ниже

![laravel-homestead](/assets/images/articles/manual-add-vagrant-box/laravel-add-box-404.png)

Для того чтобы добавить вручную laravel homestead необходимо выполнить следующее:

1. Переходим по ссылке [https://app.vagrantup.com/laravel/boxes/homestead](https://app.vagrantup.com/laravel/boxes/homestead).

2. Находим последний релиз. В моем случае это 12.1.0. Находим провайдер virtualbox. Нажимаем скачать.

3. Скачиваем box. После скачивания можно переименовать бокс как вам удобно (я свой переименовал в virtualbox), главное в конце не забыть добавить расширение <mark>.box</mark>


Теперь устанавливаем скачанный box. Для этого вводим команду 
``` 
vagrant box add laravel/homestead [полный путь к скачанному файлу].
```
В моем случае код будет выглядеть следующим образом.

``` 
vagrant box add laravel/homestead  file:///c:/users/flaai/downloads/virtualbox.box
```
После успешного добавления бокса, остается его проапдейтить.

Для этого идем в папку 

<mark>c:/users/flaai/.vagrant.d/boxes/laravel-VAGRANTSLASH-homestead</mark>

В данной папке создаем файл <mark>metadata_url</mark>

В данном файле прописываем следующую ссылку

```
https://app.vagrantup.com/laravel/boxes/homestead 
```

Далее переименовываем папку с названием 0 на 12.1.0, где 12.1.0 это версия laravel/homestead бокса
