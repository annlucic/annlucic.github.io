# Описание проекта

## Анатика в авиакомпании (парсинг, SQL, python)

**Описание проекта:** Вы аналитик компании «F9» — это российская авиакомпания, выполняющая внутренние пассажирские авиаперевозки. Сотни перелётов каждый день. Важно понять предпочтения пользователей, покупающих билеты на те или иные направления.
Вам предстоит изучить базу данных и проанализировать спрос пассажиров на рейсы в города, где проходят крупнейшие фестивали.

**В датасете** - База данных об авиаперевозках:
Таблица airports — информация об аэропортах:
airport_code — трёхбуквенный код аэропорта
airport_name — название аэропорта
city — город
timezone — временная зона

Таблица aircrafts — информация о самолётах:
aircraft_code — код модели самолёта
model — модель самолёта
range — количество самолётов

Таблица tickets — информация о билетах:
ticket_no — уникальный номер билета
passenger_id — персональный идентификатор пассажира
passenger_name — имя и фамилия пассажира

Таблица flights — информация о рейсах:
flight_id — уникальный идентификатор рейса
departure_airport — аэропорт вылета
departure_time — дата и время вылета
arrival_airport — аэропорт прилёта
arrival_time — дата и время прилёта
aircraft_code – id самолёта

Таблица ticket_flights — стыковая таблица «рейсы-билеты»
ticket_no — номер билета
flight_id — идентификатор рейса

Таблица festivals — информация о фестивалях
festival_id — уникальный номер фестиваля
festival_date — дата проведения фестиваля
festival_city — город проведения фестиваля
festival_name — название фестиваля

Пояснение: В базе данных нет прямой связи между таблицами airports и festivals, а также festivals и flights. Но вы можете писать JOIN и связывать эти таблицы по городу проведения фестиваля (festival_city) и городу аэропорта (city). Потребуется некоторое преобразование дат в flights, и тогда данную таблицу также можно будет связать по дате проведения фестиваля (festival_date) в запросах с JOIN.

Проект Яндекс.Практикума

**Задачи:** 

Инструкция по выполнению проекта

Проект состоит из двух частей: тренажёр и Jupyter Notebook для отправки на проверку. Шаги 1, 2 и 3 выполните в тренажёре, а шаг 4 — в Jupyter Notebook.



Шаг 3. 
Шаг 
### 1. Парсинг

Напишите парсер для сбора с сайта данных о 11 крупнейших фестивалях 2018 года. Сохраните данные в датафрейм festivals и выведите на экран.
Ссылка на сайт: https://code.s3.yandex.net/learning-materials/data-analyst/festival_news/index.html

```python
import requests

URL = 'https://code.s3.yandex.net/learning-materials/data-analyst/festival_news/index.html'

req = requests.get(URL)
#print(req.text)

from bs4 import BeautifulSoup
soup=BeautifulSoup(req.text, 'lxml')

table = soup.find('table')
# применим метод find к тегу table

heading_table = [] 
for row in table.find_all('th'): 
        heading_table.append(row.text) 

content=[] 
for row in table.find_all('tr'): 

    if not row.find_all('th'): 
# Эта проверка необходима, чтобы пропустить первую строку таблицы с заголовками
            content.append([element.text for element in row.find_all('td')])


import pandas as pd
festivals = pd.DataFrame(content, columns=heading_table) 

print(festivals)
```

### 2. Исследовательский анализ данных

Найдите количество рейсов с вылетом в сентябре 2018 года на каждой модели самолёта. Назовите получившееся поле flights_amount и выведите его, также выведите поле model.

```SQL
SELECT
    aircrafts.model AS model,
    COUNT(flights.flight_id) AS flights_amount
FROM
    flights
LEFT JOIN
    aircrafts ON aircrafts.aircraft_code = flights.aircraft_code
WHERE
    CAST(departure_time AS date) < '2018-10-01' AND
    CAST(departure_time AS date) > '2018-08-31'
GROUP BY
    model
```

Посчитайте количество рейсов по всем моделям самолетов Boeing и Airbus в сентябре. Назовите получившуюся переменную flights_amount и выведите ее.

```SQL
SELECT
   COUNT(flights.flight_id) AS flights_amount,
   CASE WHEN model LIKE 'Airbus%' THEN 'Airbus'
        WHEN model LIKE 'Boeing%' THEN 'Boeing'
        ELSE 'other'
        END AS type_aircraft
FROM
   flights
INNER JOIN
    aircrafts ON aircrafts.aircraft_code = flights.aircraft_code
WHERE
    CAST(departure_time AS date) BETWEEN '2018-09-01' AND '2018-09-30'
GROUP BY
    type_aircraft
```
Посчитайте среднее количество прибывающих рейсов в день для каждого города за август 2018 года. Назовите получившееся поле average_flights , вместе с ней выведите столбец city.

```SQL
SELECT
    subq.city AS city,
    AVG(subq.flight_amount) AS average_flights
FROM
(SELECT
     COUNT(flights.flight_id) AS flight_amount,
     airports.city AS city,
     EXTRACT (day from CAST(flights.arrival_time AS date)) AS day
 FROM
     flights
 INNER JOIN airports
     ON airports.airport_code = flights.arrival_airport
 WHERE
     CAST(flights.departure_time AS date) BETWEEN '2018-08-01' AND '2018-08-31'
 GROUP BY
     city,
     day) AS subq

GROUP BY 
    city;
```

### 3. SQL

Проверьте гипотезу, что количество рейсов во время фестивалей увеличивается

Установите фестивали, которые проходили с 23 июля по 30 сентября 2018 года в Москве, и номер недели, в которую они проходили. Выведите название фестиваля festival_name и номер недели festival_week.

```SQL
SELECT
    festival_name,
    EXTRACT(WEEK FROM CAST(festival_date AS date)) AS festival_week
FROM
    festivals
WHERE
    festival_city = 'Москва' AND festival_date BETWEEN  '2018-07-23' AND '2018-09-30'

```

Для каждой недели с 23 июля по 30 сентября 2018 года посчитайте количество билетов, купленных на рейсы в Москву (номер недели week_number и количество рейсов flights_amount). Получите таблицу, в которой будет информация о количестве купленных за неделю билетов, отметка, проходил ли в эту неделю фестиваль, название фестиваля festival_name и номер недели week_number.


```SQL
SELECT
    EXTRACT(week FROM CAST(flights.arrival_time AS date)) AS week_number,
    COUNT(ticket_flights.ticket_no) AS ticket_amount,
    sub.festival_week AS festival_week,
    sub.festival_name AS festival_name
    
FROM tickets
    LEFT JOIN ticket_flights ON ticket_flights.ticket_no  = tickets.ticket_no
    LEFT JOIN flights ON flights.flight_id  = ticket_flights.flight_id
    LEFT JOIN airports ON airports.airport_code = flights.arrival_airport
    LEFT JOIN
        (SELECT
             festival_name,
             EXTRACT(WEEK FROM CAST(festival_date AS date)) AS festival_week
         FROM
             festivals
         WHERE
             festival_city = 'Москва' AND festival_date BETWEEN '2018-07-23' AND '2018-09-30') as sub ON sub.festival_week = EXTRACT(week FROM CAST(flights.arrival_time AS date))
WHERE
    airports.city = 'Москва' AND
    CAST(flights.arrival_time AS date) BETWEEN '2018-07-23' AND '2018-09-30'
GROUP BY
    EXTRACT(week FROM CAST(flights.arrival_time AS date)), 
    festival_week, 
    festival_name
```


### 4. Аналитика средствами Python
У вас есть файлы, в которых содержатся результаты запросов из предыдущих заданий.

/datasets/query_1.csv — результат первого запроса. В нём содержится информация о:
model — модели самолета
flights_amount — количество рейсов для каждой модели самолетов model в сентябре 2018 года

/datasets/query_3.csv — результат третьего запроса. В нём содержится информация о:
city — городах
average_flights — среднем количестве рейсов, прибывающих в город city за день в сентябре 2018 года

```python

```

Для этих двух наборов данных нужно:

- импортировать файлы;
- изучить данные в них;
проверить типы данных на корректность;
выбрать топ-10 городов по количеству рейсов;
построить графики: модели самолетов и количество рейсов, города и количество рейсов, топ-10 городов и количество рейсов;
сделать выводы по каждому из графиков, пояснить результат.


<img src="images/.png?raw=true"/> 

### 5


### 6

На основании результатов анализа, выявлены признакие наиболее влияющие на отток и даны рекомендации.

### 6. Используемые библиотеки

pandas, numpy, matplotlib, seaborn, requests, bs4

Правилами Яндекс.Практикума запрещено выкладывать проекты в открытый доступ, поэтому [Полная версия проекта на github](https://nbviewer.jupyter.org/github/annlucic/ya_praktikum/blob/master/churn_bank.ipynb) доступна по запросу.

