# goit-rdb-hw-05

Домашнє завдання 5 з курсу реляційних баз даних. Тема: CTE, recursive CTE і Materialized View.

У цій роботі я зробила customer-level ознаки (features) для клієнтів інтернет-магазину. Робота виконана в Google Colab у файлі `hw5_Kostiuk.ipynb`.

## Датасет

Я використовувала датасет Olist Brazilian E-commerce з Kaggle:
https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

Це дані про приблизно 100 тисяч замовлень у Бразилії за 2016–2018 роки. Мені знадобились 5 таблиць:

- customers - клієнти
- orders — замовлення
- order_items — товари в замовленнях
- products — товари і їх категорії
- sellers — продавці

## Як отримати дані

CSV-файли я не завантажувала в репозиторій, бо вони великі. Notebook сам скачує їх з Kaggle через бібліотеку `kagglehub`:

```python
import kagglehub
dataset_dir = kagglehub.dataset_download('olistbr/brazilian-ecommerce')
```

Якщо Colab попросить доступ до Kaggle, треба створити токен на сайті Kaggle (Settings → API → Create New Token) і додати `KAGGLE_USERNAME` і `KAGGLE_KEY` в Secrets у Colab (значок ключа зліва).

## Raw і typed шар

Спочатку я завантажила CSV у raw-таблиці (`olist_customers_raw`, `olist_orders_raw` і т.д.). Вони такі самі, як CSV, без перевірок.

Потім я створила typed-таблиці з префіксом `olist_` (`olist_customers`, `olist_orders`, `olist_order_items`, `olist_products`, `olist_sellers`). У них правильні типи даних і є обмеження:

- PRIMARY KEY і FOREIGN KEY між таблицями
- NOT NULL для важливих колонок
- CHECK, наприклад статус замовлення тільки з дозволеного списку, а ціна не може бути від'ємною

Дані з raw перенесла в typed через `INSERT ... SELECT`. Кількість рядків у raw і typed однакова, тобто нічого не загубилось.

## CTE-stages

Pipeline з Завдання 2 складається з 8 кроків (stages):

| Stage | Що робить |
|---|---|
| Stage 0 `reference_date` | бере дату останнього замовлення + 1 день, від неї рахується recency |
| Stage 1 `stage1_orders_total` | рахує суму кожного замовлення і чи воно скасоване |
| Stage 2 `stage2_rfm` | рахує RFM: recency_days, frequency, monetary |
| Stage 3 `stage3_cohort` | визначає когорту клієнта (квартал першого замовлення) |
| Stage 4 `stage4_rolling_aov` | рахує середній чек за останні 30 днів через window function |
| Stage 5 `stage5_cancel_rate` | рахує частку скасованих замовлень |
| Stage 6 `stage6_category_revenue` | рахує, скільки клієнт витратив у кожній категорії |
| Stage 7 `stage7_preferred_category` | вибирає улюблену категорію клієнта |
| Stage 8 `stage8_segment` | ділить клієнтів на сегменти: active, cooling, churn_risk |

Скасовані замовлення я не враховувала в monetary і в середньому чеку, бо це не реальні гроші клієнта.


- Завдання 3: таблиця `hw5_referrals` (штучні реферали у вигляді дерева) і recursive CTE, який рахує глибину і шлях. Є захист від циклів і приклад з циклом A → B → C → A.
- Завдання 4: Materialized View `mv_olist_customer_features` з унікальним індексом на `customer_id` і `REFRESH MATERIALIZED VIEW CONCURRENTLY`.
- Завдання 5: EXPLAIN (ANALYZE, BUFFERS) для повного pipeline і для пошуку в MV, і Reflection.

## Як запустити

1. Відкрити `hw5_Kostiuk.ipynb` у Google Colab.
2. Якщо треба, додати Kaggle-ключі в Secrets (див. вище).
3. Натиснути Runtime → Restart session and run all.
4. Почекати, поки виконаються всі клітинки. Все встановлюється і завантажується автоматично.

Бібліотеки, які використовуються: `pgserver` (PostgreSQL прямо в Colab), `sqlalchemy`, `psycopg2-binary`, `pandas`, `kagglehub`, `jupysql` (для клітинок `%%sql`).
