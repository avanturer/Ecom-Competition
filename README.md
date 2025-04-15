# Ecom-Competition
# Прогнозирование следующего заказа

Этот репозиторий содержит реализацию задачи **Прогнозирования следующего заказа**, которая направлена на предсказание, закажет ли пользователь определённую категорию товаров в следующем заказе. Проект использует комбинацию инженерии признаков и методов машинного обучения для достижения высокой точности.

---

## Описание задачи

Сервисы доставки еды уже давно перестали быть просто курьерами, которые привозят заказ. Индустрия e-grocery стремительно идёт к аккумулированию и использованию больших данных, чтобы знать о своих пользователях больше и предоставлять более качественные и персонализированные услуги. Одним из шагов к такой персонализации может быть разработка модели, которая понимает привычки и нужды пользователя и, например, может угадать, что и когда пользователь захочет заказать в следующий раз.

Такая модель может принести значительную ценность для клиента:
- Сэкономить время при сборке корзины.
- Помочь ничего не забыть в заказе.
- Убрать необходимость планировать закупки и следить за заканчивающимися запасами продуктов.

В рамках данной задачи необходимо решить проблему предсказания следующего заказа пользователя (без привязки ко времени). Заказ состоит из списка уникальных категорий товаров, независимо от их количества.

### Данные

#### train.csv
- **user_id**: уникальный идентификатор пользователя.
- **order_completed_at**: дата заказа.
- **cart**: список уникальных категорий (category_id), из которых состоял заказ.

#### sample_submission.csv
- **id**: идентификатор строки в формате `{user_id};{category_id}`.
- **target**: 1, если категория будет присутствовать в следующем заказе пользователя, иначе 0.

### Метрика оценки

Оценка проводится с использованием **F1-меры**, которая учитывает баланс между точностью (precision) и полнотой (recall). Формула F1-меры:

F₁ = 2 × (precision × recall) / (precision + recall)

где:
precision = TP / (TP + FP)
recall = TP / (TP + FN)

Задача состоит в нахождении оптимального баланса между precision и recall.

---

## Общий обзор проекта

### Этапы реализации

#### 1. Создание тренировочного датасета и инженерия признаков
Ключевые признаки:
- **order_number**: номер заказа для каждого пользователя.
- **user_id**: уникальный идентификатор пользователя.
- **category**: категория товара.
- **ordered**: количество покупок определённой категории пользователем.
- **orders_total**: общее количество покупок пользователя.
- **rating**: доля категории в общих покупках пользователя.
- **total_ordered**: общее количество покупок категории всеми пользователями.
- **id**: комбинация `user_id` и `category`.
- **target**: целевая переменная (наличие категории в следующем заказе).

#### 2. Обучение модели
Используется алгоритм **CatBoostClassifier** с кросс-валидацией (K-Fold) для повышения устойчивости результатов. Основные параметры:
- **Итерации**: 5000
- **Глубина**: 10
- **Метрика**: F1-Score
- **Веса классов**: автоматически вычисляются для баланса данных.

#### 3. Создание тестового датасета
Аналогичные шаги по инженерии признаков применяются к тестовому набору данных для обеспечения совместимости с тренировочной выборкой.

#### 4. Агрегация предсказаний из 5 фолдов
Предсказания усредняются по всем фолдам для повышения стабильности.

#### 5. Оптимизация порога вероятности
Оптимальный порог определяется на основе:
- Соответствия среднего уровня предсказанных вероятностей уровню в тренировочной выборке.
- Адаптации к реальным распределениям данных.

#### 6. Итоговые предсказания и сабмит
Финальные предсказания экспортируются в формате, требуемом для сабмита, с **публичным F1-Score = 0.49166** (16-е место на лидерборде).

---

## Методология

### Инженерия признаков
Разработаны признаки, отражающие поведение пользователя и тенденции для конкретных категорий. Ключевые преобразования включают:
- Расчёт частот и средних значений покупок.
- Группировка и преобразование данных для соответствия задаче классификации.

### Обучение модели
Алгоритм CatBoost выбран за способность работать с категориальными признаками и дисбалансом классов. Использовалась стратегия кросс-валидации на 5 фолдах для повышения устойчивости модели.

### Оценка
Основной метрикой выступает **F1-Score**, отражающий баланс между точностью и полнотой в задаче бинарной классификации.

---

## Результаты

- **Публичный F1-Score**: 0.49166
- **Место в лидерборде**: #16

---

## Установка и использование

### Требования
- Python 3.8+
- Библиотеки: `numpy`, `pandas`, `scikit-learn`, `catboost`

### Запуск проекта
1. Установите необходимые зависимости:
   ```bash
   pip install -r requirements.txt
   ```
2. Поместите файлы train.csv и sample_submission.csv в корневую директорию.
3. Запустите ноутбук или скрипт:
   ```bash
   train_and_predict.ipynb
   ```
4. Финальные предсказания будут сохранены в файле `submission_baseline_cb_5_folds.csv`.

---