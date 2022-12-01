# Tinder
## 1. Сервис для знакомств

### Целевая аудитория 
* 75 миллионов активных пользователей в месяц
* 3 млрд свайпов в день
* более 30 млн совпадений в день

|Распределение аудитории |
|------------------------|
|Северная Америка        |
|Европа                  |


### MVP
* Профиль пользователя
* Персональная лента рекомендаций
* Система свайпов
* Чаты совпавших пользователей

## 2. Расчет нагрузки
### Продуктовые метрики

| Продуктовые метрики                        |                        |
|--------------------------------------------|------------------------|
|Число зарегистрированных аккаунтов          | 75 млн.                |                                      
|Месячная аудитория                          | 37 млн.                |
|Дневная аудитория (35% от месячной)         | 13 млн.                |
|Среднее время использования сервиса в день  | 20 минут               |	
|Кол-во свайпов в день                       | 300 млн.              |
|Количество мэтчей в день                    | 10 млн.                |

|Средний размер хранилища пользователя       |                         |
|--------------------------------------------|-------------------------|
| Фото в профиле (в среднем 5 шт.)           | 2000 Кбайт (10Мбайт)    |
| Сообщения пользователя                     | 1 Мбайт                 |
| Описание профиля и др. контент             | 0.5 Мбайт               |
| Итого                                      | 11.5 Мбайт              |

### Технические метрики


| Размер хранения в разбивке по типам данных (в Тб)|                                   |
|--------------------------------------------------|-----------------------------------|
| Фото                                             | 715 Тб ( 10Mбайт * 75 млн )|
| Текст                                            | 107 Тб                     |

#### Сетевой трафик

* Суммарный суточный (Гбайт/сутки):\
В день происходит 300 млн. свайпов\
При просмотре страницы происходит запрос за фотографией и информацией о просматриваемом человеке.\
В среднем пользователь просматривает 2-3 фотографии каждой страницы\
300 * 10^6 * (6Мб + 1Мб) =  16.4 * 10^6 Гбит в день - 190 Гбит/c.

* Пиковое потребление в течение суток (в Гбит/с):\
Большинство пользователей, активны в приложении в вечернее время с 21:00 до 22:00
Допустим, что пиковое потребление выше суточной в 3 раза, тогда пиковая нагрузка - около 570 ГБит/c.


#### RPS в разбивке по типам запросов

* Свайп создание:\
`300 * 10^6 / (24 * 60 * 60) ≈ 3470 RPS`
* Свайп чтение:\
`280 * 10^6 / (24 * 60 * 60) ≈ 3240 RPS`
* Просмотр профиля:\
`300 * 10^6 / (24 * 60 * 60) ≈ 3470 RPS`
* Фото:\
`300 * 10^6  / (24 * 60 * 60) ≈ 3470 RPS`
* Сообщение на отправку:\
В среднем пользователь за день отправляет 10 сообщений\
`10^7 * 10 * 2 / (24 * 60 * 60) ≈ 2300 RPS`
* Сообщение на чтение:\
`10^7 * 10 * 2 / (24 * 60 * 60) ≈ 2300 RPS`
* Профиль:\
Ежедневно в среднем пользователи создают профиль	около 100 тыс. раз и редактируют профиль	60 тыс.\
`(100e3 + 60e3) / 24 / 60 / 60 ≈ 2 RPS`
* Мэтчи (чтение для составления рекомендаций):\
`10^7 / 24 / 60 / 60 ≈ 115 RPS`
* Мэтчи (создание):\
`10^7 / 24 / 60 / 60 ≈ 115 RPS`


|              |               |
|--------------|---------------|
|Свайп создание | 3470 RPS     |
|Свайп чтение | 3240 RPS      |
|Просмотр профиля | 3470 RPS     |
|Фото          | 3470 RPS     |
|Мэтчи чтение  | 115 RPS       |
|Мэтчи создание| 115 RPS       |
|Сообщение на отправку | 2300 RPS |
|Сообщение на чтение | 2300 RPS   |
|Профиль       | 2 RPS         |

## 3. Логическая схема БД
![image](https://user-images.githubusercontent.com/74594938/204008143-00bea196-43a3-4f1f-a15c-b678c90f9448.png)
## 4. Физическая схема БД

| Таблица | Общий размер | RPS insert | RPS select | Шардинг| Репликация|
| --- | --- | --- | --- |-----|-----|
| Users | 48 Гб | 2 | 3585 | горизонтальное шардирование, шарды по user_id| master-slave репликация - по 2 реплики на сервер|
| Swipes | 128 Гб | 3470 | 3240 | горизонтальное шардирование, шарды по  user_id (в таблице имеются встречные строки)| master-slave репликация - по 2 реплики на сервер|
| Sessions | 7 Гб | 2 | 8300 | шарды по хэш слотам | master-slave репликация - по 2 реплики на сервер|
| Messages | 7.5 Тб | 2300 | 2300 | горизонтальное шардирование, шарды по  match_id| master-slave репликация - по 2 реплики на сервер|
| Matches | 45 Гб | 115 | 115 | горизонтальное шардирование, шарды по  first_user_id (в таблице имеются встречные строки)| master-slave репликация - по 2 реплики на сервер|
| Photos | 140 Гб | 2 | 3585 | горизонтальное шардирование, шарды по  user_id| master-slave репликация - по 2 реплики на сервер |

Photos:
| Поле | Тип | Размер |
| --- | --- | --- |
| Photo id | bigint | 8 |
| User id | bigint | 8 |
| Photo url | varchar(256)  | 256 |

На 100 миллионов аккаунтов\
всего фотографий 5 * 100 * 10^6 = 500 * 10^6 =>\
`Размер базы ссылок на фотографии: 500 * 10^6 * 300 байт = 140 Гб`\
`insert RPS  = 2 RPS (создание профиля, редактирование)`\
`select RPS  = 3470 + 115 = 18635 RPS ( отображение профиля при свайпе, при мэтче)`


Matches:
| Поле | Тип | Размер |
| --- | --- | --- |
| Match id | bigint | 8 |
| First user id | bigint | 8 |
| Second user id | bigint | 8 |
| Time sending | timestamp | 8 |

На 100 миллионов аккаунтов\
Так как количество мэтчей постоянно растет, (как среднее число мэтчей у одного пользователя - 10).\
Для шардинга используется first user id, поэтому необходимы встречные записи, тогда\
общее кол-во записей 10 * 100 * 2 = 2000 
`Общий размер: 2000 * 10^6 * 24 = 45 Гб`\
`insert RPS  = 115 RPS ( создание мэтча )`\
`select RPS  = 115 RPS ( чтение для рекомендательной системы )`

Messages:
| Поле | Тип | Размер |
| --- | --- | --- |
| Message id | bigint | 8 |
| User sender id | bigint | 8 |
| User receiver id | bigint | 8 |
| Match id | bigint | 8 |
| Text | varchar(100) | 100 |
| Time sending | timestamp | 8 |

На 100 миллионов аккаунтов\
Взяв за среднее кол-во мэтчей - 10, предположим, что на одного пользователя приходится 30 сообщений в чате, тогда\
`Общий размер: 2 * 30 * 10 * 100 * 10^6 * 140 = 7.5 Тб`\
`insert RPS  = 2300 RPS (отправка сообщения)`\
`select RPS  = 2300 RPS (чтение сообщения)`

Sessions:
| Поле | Тип | Размер |
| --- | --- | --- |
| User id | bigint | 8 |
| Session token | varchar(64) | 64 |

На 100 миллионов аккаунтов\
`Общий размер: 100 * 10^6 * 72 = 7 Гб`\
`insert RPS  = 2 RPS (создание профиля, авторизация, выход)`\
`select RPS  = 3470 + 115*2 + 2300*2 = 8300 RPS (запросы требующие проверку сессии - свайп, мэтч,сообщения кроме создания профиля и авторизации)`


Swipes:
| Поле | Тип | Размер |
| --- | --- | --- |
| Swipe id | bigint | 8 |
| User id | bigint | 8 |
| User target id | bigint | 8 |
| value | boolean | 1 |
| Time sending | timestamp | 8 |

за день имеем 300 млн. свайпов и 10 млн. мэтчей\
взаимные положительные свайпы ( при создании мэтча) будут удаляться\
Следовательно в среднем мы храним 280 млн. свайпов \
предположим, что хранить данные о свайпах будем 7 дней\
Для шардинга используется user id, поэтому необходимы встречные записи, тогда\
`Общий размер: 2 * 35 * 7 * 300 * 10^6 = 128 Гб`\
`insert RPS  = 3470 RPS ( действие свайп )`\
`select RPS  = 3240 RPS ( для нахождения мэтча )`


Users:
| Поле | Тип | Размер |
| --- | --- | --- |
| User id | bigint | 8 |
| Name | timestamp | 8 |
| Age | varchar(20) | 20 |
| Phone number | varchar(20) | 20 |
| Sex | varchar(20) | 20 |
| City | varchar(20) | 20 |
| Hobby | varchar(100) | 100 |
| Job  | varchar(100) | 100 |
| Education | varchar(100) | 100 |
| Social network accounts | varchar(100) | 100 |

На 100 миллионов аккаунтов\
`Общий размер: 100 * 10^6 * 520 = 48 Гб`\
`insert RPS  = 2 RPS (создание профиля, редактирование)`\
`select RPS  = 3470 + 115 = 3585 RPS ( отображение профиля при свайпе, при мэчте )`



 ### Индексы
  | Таблица | Поле | Описание  |
  |-------|--------|------------|
  | Sessions | session token | необходимо для всех запросов пользователя, кроме запросов для профиля (поле телефон)|
  | Users | Phone number |необходимо для запросов для профиля |
  | Users | City | необходимо для запросов для рекомендательной системы |
  | Users | Age | необходимо для запросов для рекомендательной системы |
  | Messages | User sender id | необходимо для запросов на получение сообщений |
  | Messages | User receiver id | необходимо для запросов на получение сообщений |

![image](https://user-images.githubusercontent.com/74594938/204601463-16a9c872-64a2-4ab5-9ae0-d50fad1cf019.png)

#### Базы данных
PostgreSQL: Users, Matches, Swipes, Messages\
Redis: Sessions\
Amazon S3: хранение фотографий.
#### Схема шардинга
Использоваться будет master-slave репликация - по 2 реплики на сервер.

Схема работы: мастер принимает запросы на запись, реплики - на чтение, при выходе из строя мастера одна из реплик
возьмет на себя запросы на запись, при выходе из строя всех реплик запросы будут приходить только на мастер.

| Таблица  | Шардинг по |
|----------| ---------- |
| Users    | User id    |
| Swipes   | User id    |
| Matches  | First user id |
| Messages | Match id |


## 5. Технологии

Для балансировки на Backend и отдачи статических файлов будет использован веб-сервер Nginx, который активно применяется в этих сферах.\
Для Backend части будет использован язык программирования Go, который используется для написания высоконагруженных веб-сервисов.\
Backend на микросервисной архитектуре. Общение между микросервисами будем организовывать при помощи gRPC.\
Как СУБД будут использованы PostgreSQL и Redis, которые обладают хорошими функциональностью и поддержкой.\
Для хранения изображений будем пользоваться Amazon S3 хранилищем.
Так как основная аудитория проекта - пользователи мобильных устройств, для разработки приложения используются Swift на IOS и Kotlin на Android как основные языки разработки для данных платформ.\
В web версии для frontend части будет использован фреймворк React, который обладает высокой популярностью и поддержкой.
## 6. Схема проекта
![image](https://user-images.githubusercontent.com/74594938/204130576-960a3687-5a95-451a-ab9c-0c7111ff9c04.png)\
Для балансировки применяется DNS балансировка, затем L7-балансировка на Nginx.\
Для балансировки кэширующих прокси:  прокси по хэшу запрашиваемого URL, для обеспечения нормального горизонтального масштабирования - Consistent hashing\
JS-клиент и статические файлы раздаются Nginx, который также проксирует запросы на сами микросервисы.
## 7. Список серверов

|Узел          |Cores  | RAM| SDD/HDD  | Количество (репликация)  | 
|--------------|-------|------|--------|------------|
| Nginx     | 16    | 32|	256 SSD | 30 (10 мастеров, 20 слейвов)|   
| User microservice| 40  | 512|	256 SSD  | 18 (6 мастеров, 12 слейвов)|
| Messages   microservice | 40   | 512	|256 SSD | 24 (8 мастера, 16 слейвов)|
| Swipes microservice  | 44  | 512|	256 SSD| 32 (12 мастера, 20 слейвов)|
| Recommendation microservice| 40 | 512	| 256 SSD |6(2 мастера, 4 слейва)|  
| Cache proxy | 36   | 128  |12 ТБ SSD| 10 |
|PostgreSQL|64|512|64 Тб HDD|12( 4 мастера, 8 слейвов)|
|Redis|16|64|128|6(2 мастера, 4 слейва)|

#### Nginx:
`суммарный RPS : 18482  Примем суммарное CPS на загрузку ресурсов: 200000 CPS`\
`суммарный входящий трафик(пик): 570 Гбит/с`\
`суммарный исходящий трафик(пик) в бекенды: 570 Гбит/с`\
для обработки HTTPS-запросов со средним размером 1 МБ будет используем 16-ядерный процессор\
`200000 CPS / 6,676 CPS = 16 ядер * 30 серверов`\
`пропускная способность: 30 * 66 Гбит/с = 1980 Гбит/с`\
#### User microservice:
`суммарный RPS: 3472 RPS`\
Трафик вносят запросы на просмотр профиля\
на одно ядро процессора примем RPS = 5\
` 3472 RPS/5 RPS = 695 ядер = 40 ядер * 18 серверов`\
#### Messages microservice:
`суммарный RPS: 4600 RPS`\
на одно ядро процессора примем RPS = 5\
` 4600 RPS/5 RPS = 920 ядер = 40 ядер * 24 сервера`\
#### Swipes microservice:
`суммарный RPS: 6710 RPS`\
Трафик вносят запросы на создание и чтение свайпов\
на одно ядро процессора примем RPS = 5\
` 6710 RPS/5 RPS = 1342 ядер = 44 ядра * 32 сервера`\
#### Recommendation microservice:
`суммарный RPS: 230 RPS`\
Трафик вносят запросы на чтение и создание мэтчей\
на одно ядро процессора примем RPS = 1\
` 230 RPS/1 RPS = 230 ядер = 40 ядер * 6 сервер`\
#### Cache proxy
На 40 млн. активных пользователей, у каждого в среднем по 5 фотографий, средним размером 2 МБ,\
желаем получить cache hit 60%, тогда объем кэша\
`40 * 10^6 * 5 * 2 * 0,6 МБ =  228 Тб`\
#### PostgreSQL
`суммарный RPS: 15131 RPS ( 5891 insert RPS + 9240 select RPS)`\
`объем данных : 8 Тб `
#### Redis
`суммарный RPS: 8302 RPS ( 2 insert RPS + 8300 select RPS)`\
`объем данных : 7 Гб `


#### Источники
1. https://vc.ru/services/68840-obzor-rynka-onlayn-znakomstv-skolko-zarabatyvayut-prilozheniya-i-kakie-biznes-modeli-ispolzuyut?ysclid=l879uyof0632857830
2. https://www.knowyourmobile.com/ru/news/tinder-stats-facts/
3. https://www.businessofapps.com/data/tinder-statistics/
4. https://www.huffingtonpost.co.uk/entry/how-to-get-tinder-matches_n_56a78f4be4b0172c659422da
5. https://www.reuters.com/article/us-match-group-results-idUSKBN2A22V1
