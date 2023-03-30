---
title: Как добавить phpmyadmin в Homestead
layout: post
author: Alexandr Grigoriev
description: Подборка способов как добавить и настроить phpmyadmin в среде Homestead.
---

Небольшая заметка о том как настроить phpmyadmin в среде Homestead. Если вы успешно установили себе homestead, то скорее всего вам также дополнительно понадобиться установить себе phpmyadmin для работы с базой данных.

Сегодня мы рассмотрим как это сделать.

## Установка phpmyadmin

После успешной настройки homestead у вас на виртуальной машине должна появиться папка с вашими проектами. У меня эта папка <mark>code</mark>.

### 1 способ установки:

Заходим в homestead.
<div class="filename">bash</div>
```
ssh vagrant@homestead.
```
Переходим в вашу папку с проектами, напомню что у меня это папка code. 
<div class="filename">bash</div>
```
cd code
```
Выполняем команду
<div class="filename">bash</div>
```
curl -sS https://raw.githubusercontent.com/grrnikos/pma/master/pma.sh | bash
```
После выполнения команды в вашей директории с проектами должна появиться папка <mark>phpmyadmin</mark>.

Далее переходим к финальным шагам установки.

## 2 способ установки: 

Данный способ подразумевает установку phpmyadmin из Ubuntu репозиториев. Выполняем команду
<div class="filename">bash</div>
```
sudo apt-get install phpmyadmin
```
Создаем символическую ссылку
<div class="filename">bash</div>
```
sudo ln -s /usr/share/phpmyadmin/ /home/vagrant/code/phpmyadmin
```
*подразумевается что ваши проекты также расположены в папке code.

Далее выполняем команду
<div class="filename">bash</div>
```
cd ~/Code && serve phpmyadmin.test /home/vagrant/code/phpmyadmin
```
Далее переходим к финальным шагам установки.
### Финальные шаги

Теперь на  вашей основной машине необходимо добавить phpmyadmin в файл hosts. Открываем файл <mark>C:\Windows\System32\drivers\etc\hosts и добавляем в него.</mark>
<div class="filename">hosts</div>
```
127.0.0.1 phpmyadmin.test
```

Переходим http://phpmyadmin.test:8000

## Устранение проблем

При работе с phpmyadmin в homestead я столкнулся со следующей проблемой:

При установке нового проекта в homestead и выполнения команды <mark>vagrant reload --provision</mark>, phpmyadmin постоянно слетал. Вместо формы входа в phpmyadmin отображался один из моих сайтов.

Решалась данная проблема радикальным способом. Выполнение команды в папке с проектами
<div class="filename">bash</div>
```
rm phpmyadmin
```
и последующей установкой его заново.

Другой способ устранения данной проблемы, это добавить в файл <mark>homestead.yaml</mark> маппингом папок.

В файле <mark>homestead.yaml</mark> необходимо добавить
<div class="filename">homestead.yaml</div>
```
   	- map: phpmyadmin.test
      	to: /home/vagrant/code/phpmyadmin
```
где phpmyadmin.test это адрес из файла hosts.
