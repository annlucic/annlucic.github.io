## Аналитика в Яндекс.Афише

**Описание проекта:** в распоряжении есть данные (три датасета) от Яндекс.Афиши с июня 2017 по конец мая 2018 года:
- лог сервера с данными о посещениях сайта Яндекс.Афиши,
- выгрузка всех заказов за этот период,
- статистика рекламных расходов.

В датасетах - id пользователя, категория устройства, дата и время начала и окончания сессии, дата и время заказа, выручка Яндекс.Афиши с этого заказа, идентификатор рекламного источника, затраты на этот рекламный источник в этот день.

Проект Яндекс.Практикума

**Задачи:** помочь маркетологам оптимизировать маркетинговые затраты. 
Проанализировать продукт, продажи и маркетинг и определить: 
- Сколько людей пользуются в день, неделю, месяц?
- Сколько сессий в день?
- Сколько длится одна сессия?
- Как часто люди возвращаются?
- Когда люди начинают покупать?
- Сколько раз покупают за период?
- Какой средний чек?
- Сколько денег приносят? 
- Сколько денег потратили? Всего / на каждый источник / по времени
- Сколько стоило привлечение одного покупателя из каждого источника?
- На сколько окупились расходы? 

### 1. Метрики

 В проекте рассчитываются:
 - метрики пользовательской активности - dau, mau, wau;
 - средняя продолжительность сессии - ASL;
 - Retention Rate (проводится когортный анализ)
 - Средний чек (проводится когортный анализ)
 - "Пожизненную" ценность клиента (LTV) по «возрастным» когортам
 - Валовую прибыль
 - Затраты на маркетинг по источникам
 - Стоимость привлечения пользователя (CAC)
 - ROI по источникам

<img src="images/rr_heatmap.png?raw=true"/>

### 2. Примеры кода

```python

```

### 3. Вывод

На основании результатов анализа, отделу маркетинга предложены рекомендации по оптимизации и перераспределению бюджета между каналами.

### 4. Используемые библиотеки

pandas, numpy, matplotlib, seaborn, plotly

Правилами Яндекс.Практикума запрещено выкладывать проекты в открытый доступ, поэтому [Полная версия проекта на github](https://github.com/annlucic/yandex_praktikum/blob/master/ab_testing.ipynb) доступна по запросу.
