# Tinder
## 1. Сервис для знакомств

### целевая аудитория 
|Распределение аудитории |
|------------------------|
|Северная Америка        |
|Европа                  |

* 75 миллионов активных пользователей в месяц
* 1.6 млрд свайпов в день
* более 30 млн совпадений в день

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
| Фото                                             | 7.5 * 10^5 Тб ( 10Mбайт * 75 млн )|
| Текст                                            | 1.1 * 10^5 Тб                     |

#### Сетевой трафик

* Суммарный суточный (Гбайт/сутки):\
В день происходит 1.6 млрд. свайпов\
При просмотре страницы происходит запрос за фотографией и информацией о просматриваемом человеке.\
В среднем пользователь просматривает 2-3 фотографии каждой страницы\
1.6 * 10^9 * (6Мб + 1Мб) =  87.5 * 10^6 Гбит в день - 1012 Гбит/c.\

* Пиковое потребление в течение суток (в Гбит/с):\
Большинство пользователей, активны в приложении в вечернее время с 21:00 до 22:00
Допустим, что 60 процентов от кол-ва свайпов в день приходится на это вечернее время , тогда пиковая нагрузка - около 600 ГБит/c.


#### RPS в разбивке по типам запросов

* Свайп:\
`1.6 * 10^9 / (24 * 60 * 60) ≈ 18520 RPS`
* Фото:\
`1.6 * 10^9 * 3 / (24 * 60 * 60) ≈ 55555 RPS`
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
Всего: 74758 RPS

|              |               |
|--------------|---------------|
|Свайпы        | 18520 RPS     |
|Фото          | 55555 RPS     |
|Мэтчи         | 150 RPS       |
|Чат на запись | 115 RPS       |
|Чат на чтение | 150 RPS       |
|Список чатов  | 150 RPS       |
|Сообщения     | 116 RPS       |
|Профиль       | 2 RPS         |
|Итого         | 74758 RPS     |
## 3. Логическая схема БД
 ![image](https://user-images.githubusercontent.com/74594938/199004619-14e64377-31fb-4468-af1b-3915008cb9b9.png)
## 4. Физическая схема БД
Photo:
| Поле | Тип | Размер |
| --- | --- | --- |
| Photo id | bigint | 8 |
| User id | bigint | 8 |
| S3 link | varchar(256)  | 256 |

Match:
| Поле | Тип | Размер |
| --- | --- | --- |
| Match id | bigint | 8 |
| First user id | bigint | 8 |
| Second user id | bigint | 8 |

Message:
| Поле | Тип | Размер |
| --- | --- | --- |
| Message id | bigint | 8 |
| User sender id | bigint | 8 |
| User receiver id | timestamp | 8 |
| Text | text | 256 |
| Time sending | timestamp | 8 |

Session:
| Поле | Тип | Размер |
| --- | --- | --- |
| Session id | bigint | 8 |
| User id | bigint | 8 |
| Session token | varchar(256) | 256 |

Swipe:
| Поле | Тип | Размер |
| --- | --- | --- |
| Swipe id | bigint | 8 |
| User id | bigint | 8 |
| User target id | bigint | 8 |
| value | boolean | 1 |

User:
| Поле | Тип | Размер |
| --- | --- | --- |
| User id | bigint | 8 |
| Name | timestamp | 8 |
| Age | varchar(20) | 20 |
| Sex | varchar(20) | 20 |
| City | varchar(20) | 20 |
| Hobby | varchar(100) | 100 |
| Job  | varchar(100) | 100 |
| Education | varchar(100) | 100 |
| Social network accounts | varchar(100) | 100 |

![image](https://user-images.githubusercontent.com/74594938/201211128-c59cc06d-bab7-429b-a5ab-1dd8f8026674.png)
#### Базы данных
PostgreSQL: User, Match. Выбрана как наиболее надежная и популярная реляционная БД.\
Redis: Session, Swipe, Message, Photo. Выбрана из-за быстрой обработки запросов.\
Amazon S3: хранение фотографий. Выбрана из-за высокой надежности, доступности, производительности и безопасности в отрасли хранения изображений.
#### Источники
1. https://vc.ru/services/68840-obzor-rynka-onlayn-znakomstv-skolko-zarabatyvayut-prilozheniya-i-kakie-biznes-modeli-ispolzuyut?ysclid=l879uyof0632857830
2. https://www.knowyourmobile.com/ru/news/tinder-stats-facts/
3. https://www.businessofapps.com/data/tinder-statistics/
4. https://www.huffingtonpost.co.uk/entry/how-to-get-tinder-matches_n_56a78f4be4b0172c659422da
5. https://www.reuters.com/article/us-match-group-results-idUSKBN2A22V1
