# Яндекс Go

<!-- ## О -->

## 1. Тема и целевая аудитория

Яндекс Go – сервис для заказа такси, доставки, каршеринга, аренды электросамокатов.
Изначально возник как сервис для заказа такси, поэтому имеет смысл рассматривать это за основной функционал.

### MVP

1. Регистрация/авторизация для пользователя/водителя
2. Выбор точки подачи/назначения
3. Выбор типа транспорта (легковой, грузовой) и их классы (эконом, комфорт и тд.)
4. Возможность принять заказ для водителя
5. Отслеживание положения автомобиля на карте
6. Чат между пассажиром и водителем
7. Рассчёт стоимости поездки
8. Оплата заказа
9. История поездок
10. Рейтинговая система. Возможность оценить поездку, водителя (для пассажира) и пассажира (для водителя)

### Целевая аудитория

![Страны, в которых работает Яндекс Такси](https://upload.wikimedia.org/wikipedia/commons/9/98/Yandex_Taxi_available_countries.png)

* 42M+ MAU в СНГ [[1], [2], [3]](./README.md#источники)
* ~70% поездок в Яндекс Такси совершается в РФ [[2]](./README.md#источники)
* 91% жителей мегаполисов используют мобильный клиент [[5]](./README.md#источники) 
* 30% пассажиров Яндекс Такси — молодежь до 25 лет [4]
* 20% пользователей Яндекс Такси — семьи с детьми [4]
* 25% пользователей Яндекс Такси используют сервис для работы (деловые встречи и тп.) [4]
* ~94% рынка такси в РФ [7]

## 2. Расчёт нагрузки

### Продуктовые метрики

#### Аудитория

| Метрика | млн. пользователей |
| --- | --- |
| MAU | 42 |
| DAU (2019) [[6]](./README.md#источники) | 4.5 |

#### Действия

| Тип | Среднее в день на пользователя
| --- | --- |
| Регистрация | 0.0004 |
| Авторизация | 1.83 |
| Выбор точки подачи/назначения | 1.64 |
| Выбор типа транспорта | 1.64 |
| Возможность принять заказ | 11.6 |
| Отслеживание положения автомобиля на карте | 2.92 |
| Чат | 0.36 |
| Оплата заказа | 1.64 |
| История поездок | 0.034 |
| Оценки | 1.17 |

$DAU$ = $DAU_{пасс}$ (количество пассажиров в сутки) + $DAU_{вод}$ (количество водителей в сутки).
Согласно [8] в РФ 604 тыс. работающих водителей, а доля рынка Яндекс Такси [7] около 94%.
Таким образом, в Яндекс Такси задействовано примерено $DAU_{вод}=0,94 * 604 \approx 567$ тыс. водителей.
Водителям невыгоден простой, поэтому уместно считать, что все они ежедневно выезжают на заказы.
Что примерно бьётся, с данными Яндекса по числу водителей, совершающих поездки в месяц[9]  

Поездок в год `2.4B`. Усредняя, получаем, что в день совершается `6.575M` поездок. 
Тогда на активного пассажира
$DAU_{пасс} \approx 4 млн$, выходит $\frac{6.575M}{4.0M DAU} = 1.64$ заказов.  

Предположим, что каждый раз, когда пользователь решает заказать такси, он заходит в приложение
и происходит аутентификация, количество которых можно принять числу, равному количеству пассажиров на долю заказов
и утроенное количество водителей (учитываем бытовые нужды, когда смартфон заблокирован),

Учитывая, что пользователь достаточно редко меняет тип и класс машины, положим, что запрос
на расчет маршрута и стоимости поедки меняется редко, и мобильный клиент запоминает последний тип транспортного средства.
Тогда число таких запросов естественно оставить равным $1.64$ шт/пользователь.

Количество принятых водитилем заказов, оценить можно так: общее количество заказов поделенное на количество водителей.
$6.575  млн. / 0.567  млн \approx 11,6$

Достаточно естественно положить, что каждую поездку водитель отслеживает в навигаторе, а пассажир
смотрит место движения машины до момента его прибытия к месту подачи, тогда такое действие оценим как среднее их арифметического
$\frac{4*1.64 + 11.6*0.567}{4.5} \approx 2.92$  

Примем, что во время каждого четвертого заказа осуществляется взаимодействие в чате между водителем и пассажиром.
$0.25 * 6,575 / 4.5 \approx 0.36$

История поездок. Довольно сложно определить, как часто пользуются этой фичей.
Понятно, что она может применяться для разных целей: например для компенсации деловых поездок.
Примем, что пользователь хотя бы раз в месяц заходит в историю поездок, тогда это будет $1/30 \approx 0.034$ действия в день

Оценивают пользователи как водителей, так и пассажиров. Сложно указать, какую долю составляют оцененные поездки
от обещего количества поездок, но исходя из личного опыта, примем 10% оценок от пассажиров и 70% от водителей.
Тогда можно подсчитать $\frac{0.1*6.575+0.7*6.575}{4.5} \approx 1.17$

### Технические метрики

#### Объём хранилища

| Данные | объём |
| --- | --- |
| Аватар пользователя* | 160 КБ |
| Личные данные** | 1.5 КБ |
| История поездок*** | 532 Б |
| Чат**** | 1632 Б |
| **Итого** | ~164 КБ |

\* сжимается до 200x200 px формата webp  
\** Телефон + ФИО + водит. удостоверение + СТС + паспорт (кодировка UTF-8)
Получаем 4 * (12 + (6 + 5 + 9) + 89 + 99 + 116) ~ 1.5 КБ  
\*** Откуда + куда + время начала + время окончания + цена + водитель/пассажир + авто + оценка ~532 Б  
\**** Обычно сообщений в чате немного, он используется скорее для уточнения различных моментов
Пусть таких сообщений в среднем штуки 4, длинной около 100 символов, тогда для переписки получим ~4 * (4 * 100 + 8) = 1632Б

#### RPS

Можно считать, что в часы пик (вечер пятницы и субботы) RPS возрастает в 1,5 - 2 раза [10].
Метрики рассчитаем на основании действий пользователя.

| Действие | RPS средний | RPS пик |
| --- | --- | --- |
| Регистрация | 0.02 | 0.02 |
| Авторизация | 96 | 190 |
| Выбор точки подачи/назначения | 85 | 170 |
| Выбор типа транспорта | 85 | 170 |
| Возможность принять заказ водителем | 76 | 152 |
| Отслеживание положения на карте | 152 | 304 |
| Чат | 19 | 38 |
| Расчёт стоимости | 76 | 152 |
| Оплата заказа | 76 | 152 |
| Просмотр истории поездок | 2 | 3 |
| Оценка | 61 | 122 |
| **Итого** | 728 | 1456 |


#### Трафик

- Регистрация и авторизация  
    * Пиковый: 190 * 32 Б = 5.94 КБ/с
    * Суточный: 96 * 86400 * 32 = 0,25 ГБ/сутки
- Выбор точки подачи и назначения
    * Пиковый: 170 * (128 + 128) = 42,5 КБ/с
    * Суточный: 85 * 86400 * (128 + 128) = 1.75 ГБ/сутки
- Возможность принять заказ
    * Информация по заказу: 532 Б
    * Пиковый: 152 * 532 Б = 78,96 КБ/с
    * Суточный: 76 * 86400 * 532 = 3,25 ГБ/сутки
- Отслеживание положения на карте (навигация)
    * В среднем, на поездку минут в 20-25 уходит ~5 МБ трафика,
    значит это примерно 4,3 КБ/с
    * Пиковый: 304 * 4,3 КБ/с = 1297 КБ/с
    * Суточный: 152 * 86400 * 4,3 КБ/с = 53,43 ГБ/сутки
- Чат
    * Объём сообщений: 1632 Б
    * Пиковый: 38 * 1632 = 60,56 КБ/с
    * Суточный: 19 * 86400 * 1632 = 2,5 ГБ/сутки
- Оплата заказа
    * Пусть сообщение об оплате передается в формате JSON по HTTP.
    Объект содержит информацию о стоимости, валюту, номер карты, дату истечения действия и CVC.
    Это будет примерно 200 Б. Кроме того, отправляются HTTP заголовки, что составляет ~350Б.
    Значит данные на одно сообщение ~550Б.
    * Пиковый: 152 * 550 Б = 81,6 КБ/с
    * Суточный: 76 * 86400 * 550 = 3,36 ГБ/сутки
- Просмотр истории
    * пусть человек смотрит последние 15 заказов, тогда для них имеем объём трафика
    15 * 532 Б = 7980 Б
    * Пиковый: 3 * 7980 Б = 23,4 КБ/с
    * Суточный: 2 * 86400 * 7980 = 1,28 ГБ/сутки
- Оценки:
    * пусть отсылается ID поезки и оценка, тогда одно сообщение будет ~ 16Б + 4Б = 20Б
    * Пиковый: 122 * 20Б = 2,4 КБ/с
    * Суточный: 61 * 86400 * 20 = 0,1 ГБ/сутки

Учитывая, что каждый запрос ещё оборачивается HTTP, который потребляет ~ 350 байт на заголовки, получим
пиковую скорость (350 * 1456) / 1024 + 1591,8 = 2090 КБ/с = 0,016 Гб/с.  
Аналогично, для суточного потребления (350 * 728 * 86400) / 1024^3 + 65,92 = 86,42 ГБ/сутки.




## Источники
1. https://dev.go.yandex/blog/yandex-taxi-reliability-2024-05-30
2. https://tass.ru/ekonomika/18379431
3. https://ir.yandex.ru/blog/pro-rezultaty-4-kvartala-i-polnogo-2022-goda
4. https://dzen.ru/a/ZyItzK4YLArwvc53
5. https://www.rbc.ru/technology_and_media/10/10/2024/670681329a7947cf6806f69b
6. https://incountry.com/blog/interview-with-yandex-cloud-expert/
7. https://uscapital.ru/media/content_files/Prezentation__TM_4q_2023.pdf
8. https://ac.gov.ru/uploads/publications/taxi_app.pdf
9. https://highload.ru/spring/2021/abstracts/7219
10. https://yandex.ru/company/researches/2015/moscow/taxi
