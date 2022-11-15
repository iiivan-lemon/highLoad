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
|Кол-во свайпов в день                       | 1.6 млрд.              |
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
В день происходит 1.6 млрд. свайпов\
При просмотре страницы происходит запрос за фотографией и информацией о просматриваемом человеке.\
В среднем пользователь просматривает 2-3 фотографии каждой страницы\
1.6 * 10^9 * (6Мб + 1Мб) =  87.5 * 10^6 Гбит в день - 1012 Гбит/c.

* Пиковое потребление в течение суток (в Гбит/с):\
Большинство пользователей, активны в приложении в вечернее время с 21:00 до 22:00
Допустим, что пиковое потребление выше среднего в 2 раза, тогда пиковая нагрузка - около 2024 ГБит/c.


#### RPS в разбивке по типам запросов

* Свайп:\
`1.6 * 10^9 / (24 * 60 * 60) ≈ 18520 RPS`
* Фото:\
`1.6 * 10^9  / (24 * 60 * 60) ≈ 18520 RPS`
* Чат на запись:\
`10^7 / (24 * 60 * 60) ≈ 116 RPS`
* Чат на чтение:\
`13^7 / (24 * 60 * 60) ≈ 150 RPS`
* Список чатов:\
`10^7 / (24 * 60 * 60) ≈ 115 RPS`
* Сообщение на отправку:\
`10^7 / (24 * 60 * 60) ≈ 116 RPS`
* Профиль:\
Ежедневно в среднем пользователи создают профиль	около 100 тыс. раз и редактируют профиль	60 тыс.\
`(100e3 + 60e3) / 24 / 60 / 60 ≈ 2 RPS`
* Мэтчи (на чтение для составления рекомендаций):\
`12e6 / 24 / 60 / 60 ≈ 150 RPS`


##### Итог 
Всего: 37723 RPS

|              |               |
|--------------|---------------|
|Свайпы        | 18520 RPS     |
|Фото          | 18520 RPS     |
|Мэтчи         | 150 RPS       |
|Чат на запись | 115 RPS       |
|Чат на чтение | 150 RPS       |
|Список чатов  | 150 RPS       |
|Сообщения     | 116 RPS       |
|Профиль       | 2 RPS         |
|Итого         | 37723 RPS     |
## 3. Логическая схема БД
 ![image](https://user-images.githubusercontent.com/74594938/199004619-14e64377-31fb-4468-af1b-3915008cb9b9.png)
## 4. Физическая схема БД
Photos:
| Поле | Тип | Размер |
| --- | --- | --- |
| Photo id | bigint | 8 |
| User id | bigint | 8 |
| Photo url | varchar(256)  | 256 |

всего фотографий 5 * 75 * 10^6 = 375 * 10^6 =>\
Размер базы ссылок на фотографии: 375 * 10^6 * 300 байт = 105 Гб

Matches:
| Поле | Тип | Размер |
| --- | --- | --- |
| Match id | bigint | 8 |
| First user id | bigint | 8 |
| Second user id | bigint | 8 |

Так как количество мэтчей постоянно растет, (среднее число мэтчей у одного пользователя - 10).\
Общий размер: 10 * 75 * 10^6 * 24 = 17 Гб

Messages:
| Поле | Тип | Размер |
| --- | --- | --- |
| Message id | bigint | 8 |
| User sender id | bigint | 8 |
| User receiver id | bigint | 8 |
| Text | varchar(100) | 100 |
| Time sending | timestamp | 8 |

Взяв за среднее кол-во мэтчей - 10, предположим, что на одного пользователя приходится 30 сообщений в чате, тогда
Общий размер: 2 * 30 * 10 * 75 * 10^6 * 135 = 5.5 Тб

Sessions:
| Поле | Тип | Размер |
| --- | --- | --- |
| User id | bigint | 8 |
| Session token | varchar(64) | 64 |

На 100 миллионов аккаунтов
Общий размер: 100 * 10^6 * 72 = 7 Гб

Swipes:
| Поле | Тип | Размер |
| --- | --- | --- |
| Swipe id | bigint | 8 |
| User id | bigint | 8 |
| User target id | bigint | 8 |
| value | boolean | 1 |
| Time sending | timestamp | 8 |

за день имеем 1.6 млрд. свайпов
предположим, что хранить данные о свайпах будем 7 дней
Общий размер: 35 * 7 * 1.6 * 10^9 = 365 Гб

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

На 100 миллионов аккаунтов
Общий размер: 100 * 10^6 * 520 = 48 Гб

![image](https://user-images.githubusercontent.com/74594938/201471630-f0c3470e-6e22-4866-995f-5be257e057de.png)
#### Базы данных
PostgreSQL: Users, Matches, Swipes\
Redis: Sessions, Messages, Photos.\
Amazon S3: хранение фотографий.
#### Схема шардинга
Будем использовать шардирование по регионам, так как Tinder используется для локальных знакомств.\
Использоваться будет master-slave репликация - по 2 реплики на сервер.\
Схема работы: мастер принимает запросы на запись, реплики - на чтение, при выходе из строя мастера одна из реплик возьмет на себя запросы на запись, при выходе из строя всех реплик запросы будут приходить только на мастер.

## 5. Технологии

Для балансировки на Backend и отдачи статических файлов будет использован веб-сервер Nginx, который активно применяется в этих сферах.\
Для Backend части будет использован язык программирования Go, который используется для написания высоконагруженных веб-сервисов.\
Backend на микросервисной архитектуре. Общение между микросервисами будем организовывать при помощи gRPC.\
Как СУБД будут использованы PostgreSQL и Redis, которые обладают хорошими функциональностью и поддержкой.\
Для хранения изображений будем пользоваться Amazon S3 хранилищем.
Так как основная аудитория проекта - пользователи мобильных устройств, для разработки приложения используются Swift на IOS и Kotlin на Android как основные языки разработки для данных платформ.\
В web версии для frontend части будет использован фреймворк React, который обладает высокой популярностью и поддержкой.
## 6. Схема проекта
![image](https://user-images.githubusercontent.com/74594938/201544912-b87d1029-788b-41ba-8133-b1da8bc35345.png)\
Для балансировки применяется DNS балансировка, затем L7-балансировка на Nginx.\
JS-клиент и статические файлы раздаются Nginx, который также проксирует запросы на сами микросервисы.
## 7. Список серверов
#### Источники
1. https://vc.ru/services/68840-obzor-rynka-onlayn-znakomstv-skolko-zarabatyvayut-prilozheniya-i-kakie-biznes-modeli-ispolzuyut?ysclid=l879uyof0632857830
2. https://www.knowyourmobile.com/ru/news/tinder-stats-facts/
3. https://www.businessofapps.com/data/tinder-statistics/
4. https://www.huffingtonpost.co.uk/entry/how-to-get-tinder-matches_n_56a78f4be4b0172c659422da
5. https://www.reuters.com/article/us-match-group-results-idUSKBN2A22V1
