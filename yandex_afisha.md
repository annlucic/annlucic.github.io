## Аналитика в Яндекс.Афише

**Описание проекта:** в распоряжении есть данные (три датасета) от Яндекс.Афиши с июня 2017 по конец мая 2018 года:
- лог сервера с данными о посещениях сайта Яндекс.Афиши,
- выгрузка всех заказов за этот период,
- статистика рекламных расходов.

**В датасетах** - id пользователя, категория устройства, дата и время начала и окончания сессии, дата и время заказа, выручка Яндекс.Афиши с этого заказа, идентификатор рекламного источника, затраты на этот рекламный источник в этот день.

Проект Яндекс.Практикума

**Задачи:** помочь маркетологам оптимизировать маркетинговые затраты. 

Проанализировать продукт, продажи, маркетинг и определить: 
- Количество пользователей в день, неделю, месяц.
- Количество сессий в день и продолжительность сессии.
- Как часто люди возвращаются.
- Когда люди начинают покупать.
- Количество покупок за период.
- Средний чек.
- Выручку. 
- Затраты на маркетинг Всего / на каждый источник / по времени.
- Стоимость привлечения одного покупателя из каждого источника.
- На сколько окупились расходы.

### 1. Метрики

 В проекте рассчитываются:
 - метрики пользовательской активности - dau, mau, wau;
 - средняя продолжительность сессии - ASL;
 - Retention Rate (проводится когортный анализ);
 - средний чек (проводится когортный анализ);
 - «пожизненная» ценность клиента (LTV) по «возрастным» когортам;
 - валовая прибыль;
 - затраты на маркетинг по источникам;
 - стоимость привлечения пользователя (CAC);
 - ROI по источникам.

<img src="images/rr_heatmap.png?raw=true"/>

### 2. Пример расчета возрастного когортного отчета

```python

first_orders = orders.groupby('uid').agg({'buy_ts': 'min'}).reset_index()
first_orders.columns = ['uid', 'first_order_ts']

first_visits = visits.groupby('uid').agg({'start_ts': 'min'}).reset_index()
first_visits.columns = ['uid', 'first_visit_ts']

buyers = pd.merge(first_visits, first_orders, on='uid')
buyers['first_order_month'] = buyers['first_order_ts'].astype('datetime64[M]')

cohort_sizes = buyers.groupby('first_order_month').agg({'uid': 'nunique'}).reset_index()
cohort_sizes.rename(columns={'uid': 'n_buyers'}, inplace=True)

cohorts = pd.merge(orders, buyers, how='inner', on=['uid', 'first_order_month'])\
    .groupby(['first_order_month', 'order_month'])\
    .agg({'revenue': ['sum', 'count']}).reset_index()
    
# Считаем возраст каждой когорты

cohorts['age_month'] = ((cohorts['order_month'] - cohorts['first_order_month']) / np.timedelta64(1,'M')).round()
cohorts.columns = ['first_order_month', 'order_month', 'revenue', 'n_orders', 'age_month']
    
# Добавляем в когортный отчет количество покупателей в каждой когорте 
# и считаем выручку и количество заказов на каждого покупателя.

cohorts_report = pd.merge(cohort_sizes, cohorts, on = 'first_order_month')
cohorts_report['rev_per_buyer'] = cohorts_report['revenue'] / cohorts_report['n_buyers']
cohorts_report['orders_per_buyer'] = cohorts_report['n_orders'] / cohorts_report['n_buyers']

 # Возростной когортный отчет, показывающий накопительную выручку на покупателя

cohorts_age = cohorts_report.pivot_table(
index='first_order_month', 
columns='age_month', 
values='rev_per_buyer', 
aggfunc='sum'
).cumsum(axis=1)

```

### 3. Вывод

На основании результатов анализа, отделу маркетинга предложены рекомендации по оптимизации и перераспределению бюджета между каналами.

### 4. Используемые библиотеки

pandas, numpy, matplotlib, seaborn, plotly

Правилами Яндекс.Практикума запрещено выкладывать проекты в открытый доступ, поэтому [Полная версия проекта на github](https://nbviewer.jupyter.org/github/annlucic/yandex_praktikum/blob/master/yandex_afisha.ipynb) доступна по запросу.

