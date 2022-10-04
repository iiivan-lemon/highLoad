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
|Кол-во свайпов в день                       | 0.8 млрд.              |
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
* Пиковое потребление в теченнии суток (в Гбит/с):\
количество пользователей составляет 10 млн., пиковое время использования сервиса 21:00-22:00
Примем,что время использования сервиса в этом интервале 10 мин, тогда для 6.5 млн пользователей - 1.1 млн одновременных сессий при равномерном распределении времени использования сервиса

`Статика - 500 КБ * 1.1е6 / 60 ≈ 70 Гбит/сек`
`Фото - 3 МБ * 1.1e6 / 60 ≈ 429 Гбит/сек`

* Суммарный суточный (Гбайт/сутки):\
 за сутки в среднем 13 млн. пользователей, 20 мин используется сервис\

`Статика - 500 КБ * 20 * 13e6 ≈ 130 000 Гбайт/сутки`
`Фото - 3 МБ * 20 * 13e6 ≈ 780 000 Гбайт/сутки`

#### RPS в разбивке по типам запросов

* Свайп:\
`800e6 / (24 * 60 * 60) ≈ 9260 RPS`
* Фото:\
При среднем кол-ва фото у пользователя - 3\
`800e6 * 3 / (24 * 60 * 60) ≈ 27777 RPS`
* Мэтчи:\
Новый мэтч\
`10e6 / (24 * 60 * 60) ≈ 115 RPS`

Список мэтчей\
`13e6 / (24 * 60 * 60) ≈ 150 RPS`

* Чаты:\
Пусть 80% совпавших пользователей начинают общение в чате -  8 млн.\
`8e6 / (24 * 60 * 60)` ≈ 92 RPS`

Список чатов\
`13e6 / (24 * 60 * 60) ≈ 150 RPS`

* Профиль:\
пусть информацию в своем профиле изменяет 1% активных пользователей - 130 тыс.\
регистрируется 100 тыс. новых пользователей\
`230e3 / (24 * 60 * 60) ≈ 3 RPS`


##### Итог 
Всего: 37407 RPS

|        |               |
|--------|---------------|
|Свайп   | 9260 RPS      |
|Фото    | 27777 RPS     |
|Мэтчи   | 265 RPS       |
|Чаты    | 242 RPS       |
|Профиль | 3 RPS         |
 
#### Источники
1. https://vc.ru/services/68840-obzor-rynka-onlayn-znakomstv-skolko-zarabatyvayut-prilozheniya-i-kakie-biznes-modeli-ispolzuyut?ysclid=l879uyof0632857830
2. https://www.knowyourmobile.com/ru/news/tinder-stats-facts/
3. https://www.businessofapps.com/data/tinder-statistics/
4. https://www.huffingtonpost.co.uk/entry/how-to-get-tinder-matches_n_56a78f4be4b0172c659422da
5. https://www.reuters.com/article/us-match-group-results-idUSKBN2A22V1
