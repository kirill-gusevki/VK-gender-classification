# VK-gender-classification
Датасеты можно найти на сайте - https://cups.online/ru/tasks/1923, (загрузка невозможно из-за большого веса файлов)

## 📋 Описание проекта

Проект был выполнен в рамках тестового задания. Цель — разработать модель машинного обучения, которая предсказывает пол пользователя на основе данных о его поведении в социальных сетях VK.

> **Бизнес-задача:** Повышение эффективности рекламных кампаний за счёт точного таргетирования по полу пользователя  
> **Техническая задача:** Бинарная классификация (пол пользователя: `1` / `0`)  
> **Метрика качества:** AUC-ROC

## 🏆 Результаты

- **Качество модели:** AUC-ROC = `0.8904`
- **Ключевые факторы:** векторные представления URL, время активности, браузер и ОС пользователя

## 🛠 Технологический стек

| Категория | Технологии |
|-----------|------------|
| **Язык программирования** | Python 3.9+ |
| **Обработка данных** | Pandas, NumPy |
| **Машинное обучение** | Scikit-learn, LightGBM |
| **Визуализация** | Matplotlib, Seaborn |
| **Работа с данными** | JSON, CSV |
| **Утилиты** | Jupyter Notebook, Joblib |

## 📁 Структура проекта
```python
VK-Gender-Prediction/
├── data/ # Исходные данные (не включены в репозиторий)
├── notebooks/
│ └── VK.ipynb # Полный Jupyter Notebook с исследованием
├── src/
│ ├── data_processing.py # Функции предобработки данных
│ ├── feature_engineering.py # Генерация признаков
│ └── modeling.py # Построение и оценка моделей
├── models/
│ ├── gender_prediction_model.joblib # Обученная модель
│ └── feature_names.json # Список признаков
├── results/
│ └── gender_prediction_submission.csv # Предсказания
└── README.md
```

## 🧮 Этапы работы

### 1. Предобработка данных

```python
# Загрузка и объединение данных из различных источников
train_labels = pd.read_csv("train_labels.csv", sep=";")
train = pd.read_csv("train.csv", sep=";", on_bad_lines='skip')
geo_info = pd.read_csv("geo_info.csv", sep=";", dtype=str)
referer_vectors = pd.read_csv("referer_vectors.csv", sep=";")

# Парсинг user_agent
def parse_user_agent(ua_str):
    ua_str = ua_str.replace("'", '"').replace('None', '"None"')
    return literal_eval(ua_str)
```

### 2. Feature Engineering
**Временные признаки**: час, день недели, время суток (ночь/утро/день/вечер)

**Признаки из User Agent**: браузер, ОС, версия, мобильные устройства

**Географические признаки**: страна, регион, часовой пояс

**Признаки из URL**: домен, путь, сложность пути

**Векторные представления**: 10 компонент из referer_vectors

### 3. Агрегация на уровне пользователя
```python
# Агрегация событий по пользователям
user_features = df.groupby('user_id').agg({
    'is_ios': ['mean', 'sum'],
    'browser': ['nunique', lambda x: x.mode()[0]],
    'hour': ['mean', 'std'],
    # ... другие агрегации
})
```

### 4. Построение и оценка модели
```python
# Обучение LightGBM с кросс-валидацией
params = {
    'objective': 'binary',
    'metric': 'auc',
    'boosting_type': 'gbdt',
    'num_leaves': 63,
    'learning_rate': 0.05,
}

model = lgb.train(params, train_data, num_boost_round=1000,
                 valid_sets=[val_data], callbacks=[lgb.early_stopping(50)])
```

### 5. Интерпретация результатов
5 наиболее важных признаков:

- **component2_mean** (0.089)

- **component3_mean** (0.083)

- **component8_mean** (0.079)

- **component1_mean** (0.078)

- **avg_hour** (0.071)

## Контакты
**Автор:** Гусев Кирилл
**Telegram:** @KiKmEp
**Email:** kirill-gusevki@yandex.ru
