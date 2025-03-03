# Проектирование высоконагруженного сервиса на примере LinkedIn

[TOC]

## 1. Тема и целевая аудитория

LinkedIn -- социальная сеть для поиска и установления деловых контактов


### Целевая аудитория

LinkedIn является международным сервисом, распространенным по всему миру, общая аудитория которого более 1.1B пользователей[^1].
Большая часть целевой аудитории расположена в США (234М+ пользователей), преимущественно мужчины (57%)[^2]. 
Количество активных пользователей в месяц (MAU) составляет ~310М[^3].

![Распределение по странам](images/EarningsMembershipNumbersMap_FY25Q2.png) 
*Распределение пользователей по странам [^1]*

| Регион | Количество пользователей |
|--------------- | --------------- |
| Северная Америка | 260M+ |
| Европа |304M+  |
| Восточная Азия | 326M+  |
| Южная Америка | 188M+   |
| Ближний Восток и Африка | 73M+ |

### Функционал

Ключевой функционал сервиса заключается создания профессионального профиля, установления деловых контактов и поиска работы.

Функционал MVP:
- создание и управление профессиональным профилем
- добавление и управление деловыми контактами
- поиск людей, компаний и вакансий
- обмен сообщениями
- публикация и взаимодействие с контентом (лайки, комментарии, репосты)
- размещение и отклик на вакансии
- система рекомендации людей, компаний и вакансий

## 2. Расчет нагрузки
MAU ~310M[^3]  
DAU ~134M[^4]

### Продуктовые метрики

#### Размер хранилища пользователя

| Данные | Размер |
| --- | --- |
| Аватар | 30 kB |
| Информация | 1 kB |
| Публикации | 10 MB |

#### Количество действий пользователя по типам в день

В минуту просматривается более 1,7M обновлений ленты[^1] => в день ~2.5B просмотров ленты

В среднем, просмотров контента в 15 раз больше, чем объявлений о вакансиях[^10] => в среднем ~168M просмотров вакансий

Более 30M открытых вакансий в год[^5] => ~83K новых вакансий в день

Более 11K пользователей каждую минуту подают заявки на работу в LinkedIn[^1] => в день ~16M откликов на вакансии

Более 18К новых связей в минуту => ~26M новых связей в день

В среднем на LinkedIn ежедневно отправляется более 100M сообщений[^5]

Ежедневно в LinkedIn публикуется ~2М постов[^6]

В среднем каждый пост на LinkedIn имеет ~24 лайка[^9] => с учетом ежедневного количества постов (~2М) в среднем ~48M лайков в день

В среднем каждый пост на LinkedIn имеет ~3 комментария[^9] => с учетом ежедневного количества постов (~2М) в среднем ~6M комментариев в день

В течение последних 3х лет количество пользователей ежегодно увеличивается на ~100M пользователей[^8] => ~274K регистраций в день

В среднем, каждый пользователь имеет около 1300 связей[^7]

Количество действий рассчитывается относительно активного пользователя

| Тип действия | Среднее количество в день |
| --- | --- |
| Регистрация | 0.002 |
| Авторизация | 1 |
| Просмотр ленты | 19 |
| Лайк | 0.4 |
| Комментарий | 0.04 |
| Отправка сообщения | 0.75 |
| Публикация | 0.015 |
| Создание связи | 0.2 |
| Размещение вакансии | 0.0006 |
| Просмотр вакансий | 1.25 |
| Отклик на вакансию | 0.12 |

### Технические метрики

Значения рассчитаны с учетом DAU ~134M  
Максимальное RPS примерно в 3 раза больше среднего

| Запрос | Количество в день | Среднее RPS | Максимальное RPS | Средний размер запроса | Средний трафик, GB/s | Пиковый трафик, GB/s |
| --- | --- | --- | --- | --- | --- | --- |
| Регистрация | 274K | 3.17 | 9.51 | 1 kB | 0.00000295 | 0.00000886 |
| Авторизация | 134M | 1550.93 | 4652.78 | 10 B | 0.00001444 | 0.00004333 |
| Просмотр ленты | 2.5B | 28935.19 | 86805.56 | 20 MB | 538.96 | 1616.88 |
| Лайк | 48M | 555.56 | 1666.67 | 10 B | 0.00000517 | 0.00001552 |
| Комментарий | 6M | 69.44 | 208.33 | 200 B | 0.00001294 | 0.00003881 |
| Отправка сообщения | 100M | 1157.41 | 3472.22 | 100 B | 0.00010779 | 0.00032338 |
| Публикация | 2M | 23.15 | 69.44 | 1.5 MB | 0.03234 | 0.09701 |
| Создание связи | 26M | 300.93 | 902.78 | 10 B | 0.00000280 | 0.00000841 |
| Размещение вакансии | 83K | 0.96 | 2.88 | 20 kB | 0.00001789 | 0.00005368 |
| Просмотр вакансий | 168M | 1944.44 | 5833.33 | 1 MB | 1.81 | 5.43 |
| Отклик на вакансию | 16M | 185.19 | 555.56 | 1 kB | 0.00017247 | 0.00051740 |
| **Итог** |  | **34726.35** | **104179.06** |  | **540.80 GB/s** | **1622.41 GB/s** |

## Список источников

[^1]: https://news.linkedin.com/about-us
[^2]: https://blog.hootsuite.com/linkedin-demographics
[^3]: https://www.linkedhelper.com/blog/linkedin-demographics
[^4]: https://thesocialshepherd.com/blog/linkedin-statistics
[^5]: https://analyzify.com/statsup/linkedin
[^6]: https://www.cognism.com/blog/linkedin-statistics
[^7]: https://99firms.com/blog/linkedin-statistics
[^8]: https://www.businessofapps.com/data/linkedin-statistics
[^9]: https://www.intotheminds.com/blog/en/linkedin-statistics
[^10]: https://business.linkedin.com/content/dam/me/business/en-us/marketing-solutions/products/pdfs/Platform-Overview-v03.16.pdf
