## Анализ результатов A/B теста крупного интернет магазина

**Описание проекта:** В распоряжении есть список гипотез 9 гипотез по увеличению выручки интернет-магазина с указанными параметрами Reach, Impact, Confidence, Effort. Файлы с результатами A/B теста (два датасета с данными о визитах и выручке)
В датасетах - id заказа и пользователя, дата заказа, выручка, группа A/B-теста, в которую попал заказ, количество пользователей в указанную дату в указанной группе A/B-теста

**Задача:** приоритизировать гипотезы, провести анализ результатов A/B теста

### 1. Приоритезация гипотез

Для приоритезации гипотез используются фреймворки ICE и RICE

### 2.  Анализ результатов 

В проекте проводится анализ:
- стабильности кумулятивных метрик (выручки, среднего чека, конверсии) по группам.
- относительного изменения кумулятивных метрик группы B к группе A.
- выбросов и всплесков, определяется граница для определения аномальных пользователей

Рассчитывается статистическая значимость различий в конверсии и среднем чеке между группами по «сырым» и «очищенным» данным.
Проверка проводится критерием критерием Манна-Уитни. Примеры кода из проекта ниже.

```python
# cтатистическая значимость различий в среднем чеке заказа между группами по «сырым» данным
print("{0:.3f}".format(stats.mannwhitneyu(sampleA, sampleB)[1]))
print("{0:.3f}".format(sampleB.mean()/sampleA.mean()-1))

# cтатистическая значимость различий в среднем чеке заказа между группами по «очищенным» данным

usersWithManyOrders = pd.concat([ordersByUsersA[ordersByUsersA['orders'] > 2]['userId'], ordersByUsersB[ordersByUsersB['orders'] > 2]['userId']], axis = 0)
usersWithExpensiveOrders = orders[orders['revenue'] > 28000]['visitorid']
abnormalUsers = pd.concat([usersWithManyOrders, usersWithExpensiveOrders], axis = 0).drop_duplicates().sort_values()

print("{0:.3f}".format(stats.mannwhitneyu(
    orders[np.logical_and(
        orders['group']=='A',
        np.logical_not(orders['visitorid'].isin(abnormalUsers)))]['revenue'],
    orders[np.logical_and(
        orders['group']=='B',
        np.logical_not(orders['visitorid'].isin(abnormalUsers)))]['revenue'])[1]))

print("{0:.3f}".format(
    orders[np.logical_and(orders['group']=='B',np.logical_not(orders['visitorid'].isin(abnormalUsers)))]['revenue'].mean()/
    orders[np.logical_and(
        orders['group']=='A',
        np.logical_not(orders['visitorid'].isin(abnormalUsers)))]['revenue'].mean() - 1))
```

### 3. Вывод

На основании результатов анализа, принято решение остановить тест и зафиксировать победу группы B. Так как есть статистически значимое различие по конверсии между группами, относительный прирост конверсии группы B около 17%.

<img src="images/ab_conv.png?raw=true"/>

### 4. Используемые библиотеки

pandas, numpy, matplotlib, scipy

Правилами Яндекс.Практикума запрещено выкладывать проекты в открытый доступ, поэтому [Полная версия проекта на github](https://github.com/annlucic/yandex_praktikum/blob/master/ab_testing.ipynb) доступна по запросу.

