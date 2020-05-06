## Анализ оттока клиентов банка

**Описание проекта:** В распоряжении данные о клиентах банка за месяц. Банк располагается в Ярославле и областных городах: Ростов Великий и Рыбинск. Клиенты стали уходить, необходимо провести анализ и разобраться в причинах оттока.

**В датасете** - id пользователя, баллы кредитного скоринга, город, пол, возраст, количество объектов в собственности, баланс на счёте, количество продуктов, которыми пользуется клиент, наличие кредитной карты, лояльность, заработная плата клиента, данные об оттоке.

Проект Яндекс.Практикума

**Задачи:** 

Провести анализ клиентов регионального банка и сегментировать пользователей по количеству потребляемых продуктов

- Провести исследовательский анализ данных
- Сегментировать пользователей на основе данных о количестве потребляемых продуктов
- Сформулировать и проверить статистические гипотезы

### 1. Исследовательский анализ данных

- Исследование и анализ распределения признаков, выявление закономерностей
<img src="images/age_churn.png?raw=true"/>
<img src="images/gender_churn.png?raw=true"/>
- Матрица корреляций 
<img src="images/corr_m.png?raw=true"/> 

### 2. Модель для предсказания оттока клиентов и кластеризация.

Проводится обучение моделей для прогнозирования оттока клиентов четырьмя алгоритмами бинарной классификиции:
- Логистическая регрессия (sklearn.linear_model.LogisticRegression())
- Дерево принятия решений (sklearn.tree.DecisionTreeClassifier())
- Случайный лес	(sklearn.ensemble.RandomForestClassifier())
- Градиентный бустинг (sklearn.ensemble.GradientBoostingClassifier(), xgboost.XGBClassifier)

Выбор лучшей на основании метрик  Accuracy, Precision, Recall, F1, ROC_AUC

```python
# разделим данные на признаки (матрица X) и целевую переменную (y)
X = df.drop(['churn', 'city_int', 'userid'], axis = 1)
y = df['churn']

# разделим модель на обучающую и валидационную выборку
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=0)

# стандартизируем данные 
scaler = StandardScaler()
scaler.fit(X_train)
X_train_st = scaler.transform(X_train)
X_test_st = scaler.transform(X_test)

# Модель на основе алгоритма случайного леса
rf_model = RandomForestClassifier(n_estimators = 100, random_state = 0) 

rf_model.fit(X_train_st, y_train)
rf_predictions = rf_model.predict(X_test_st) 
rf_probabilities = rf_model.predict_proba(X_test_st)[:,1] 
```

### Кластеризация клиентов.

С помощью построения дендрограммы определили рекомендуемое число кластеров
```python
from scipy.cluster.hierarchy import dendrogram, linkage

# Построим матрицу расстояний функцией linkage()
linked = linkage(x_sc, method = 'ward')

# Дендрограмма
plt.figure(figsize=(15, 10))  
dendrogram(linked, orientation='top')
plt.title('Hierarchial clustering')
plt.show()
```
<img src="images/dendro.png?raw=true"/> 

Проводится кластеризация методом `Kmeans`

```python
# задаём модель k_means с числом кластеров 4
km = KMeans(n_clusters = 4, random_state=0)
# прогнозируем кластеры для наблюдений
labels = km.fit_predict(x_sc)
```
Выделены закономерности в кластерах и предположения о признаках наиболее влияющих на отток
<img src="images/clusters.png?raw=true"/> 

### 3. Сегментация пользователей по потреблению

Проводится сегментация по количеству продуктов и анализ сегментов
<img src="images/ch.png?raw=true"/> 

### 4. Проверка гипотез
Проверено 9 гипотез о равенстве средних двух генеральных совокупностей методом `st.ttest_ind()`

### 5. Вывод

На основании результатов анализа, выявлены признакие наиболее влияющие на отток и даны рекомендации.

### 6. Используемые библиотеки

pandas, numpy, matplotlib, seaborn, plotly, pylab, sklearn, scipy

Правилами Яндекс.Практикума запрещено выкладывать проекты в открытый доступ, поэтому [Полная версия проекта на github](https://github.com/annlucic/yandex_praktikum/blob/master/ab_testing.ipynb) доступна по запросу.

