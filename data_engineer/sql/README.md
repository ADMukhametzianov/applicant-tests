# Тест SQL

На основе таблиц базы данных, напишите SQL код, который возвращает необходимые результаты
Пример: 

Общее количество товаров
```sql
select count (*) from items
```

## Структура данных

Используемый синтаксис: Oracle SQL или другой

| Сustomer       | Description           |
| -------------- | --------------------- |
| customer\_id   | customer unique id    |
| customer\_name | customer name         |
| country\_code  | country code ISO 3166 |

| Items             | Description       |
| ----------------- | ----------------- |
| item\_id          | item unique id    |
| item\_name        | item name         |
| item\_description | item description  |
| item\_price       | item price in USD |

| Orders       | Description                 |
| ------------ | --------------------------- |
| date\_time   | date and time of the orders |
| item\_id     | item unique id              |
| customer\_id | user unique id              |
| quantity     | number of items in order    |

| Countries     | Description           |
| ------------- | --------------------- |
| country\_code | country code          |
| country\_name | country name          |
| country\_zone | AMER, APJ, LATAM etc. |


| Сonnection\_log         | Description                           |
| ----------------------- | ------------------------------------- |
| customer\_id            | customer unique id                    |
| first\_connection\_time | date and time of the first connection |
| last\_connection\_time  | date and time of the last connection  |

## Задания

### 1) Количество покупателей из Италии и Франции

| **Country_name** | **CustomerCountDistinct** |
| ------------------------- | ----------------------------- |
| France                    | #                             |
| Italy                     | #                             |

**Ответ:**

SELECT c.country_name, COUNT(DISTINCT cust.customer_id) as customer_count

FROM Customer cust JOIN Countries c using(country_code)

WHERE c.country_name IN ('Italy', 'France') GROUP BY c.country_name;

### 2) ТОП 10 покупателей по расходам

| **Customer_name** | **Revenue** |
| ---------------------- | ----------- |
| #                      | #           |
| #                      | #           |
| #                      | #           |
| #                      | #           |
| #                      | #           |
| #                      | #           |
| #                      | #           |

**Ответ:**

SELECT c.customer_name, SUM(i.item_price * o.quantity) as revenue

FROM Orders o, Customer c, Items i where o.customer_id = c.customer_id and o.item_id = i.item_id

GROUP BY c.customer_name ORDER BY revenue DESC LIMIT 10;

### 3) Общая выручка USD по странам, если нет дохода, вернуть NULL

| **Country_name** | **RevenuePerCountry** |
| ------------------------- | --------------------- |
| Italy                     | #                     |
| France                    | NULL                  |
| Mexico                    | #                     |
| Germany                   | #                     |
| Tanzania                  | #                     |

**Ответ:**

SELECT c.country_name, SUM(i.item_price * o.quantity) as revenue

FROM Countries c JOIN Customer ct ON c.country_code = ct.country_code

LEFT JOIN Orders o ON ct.customer_id = o.customer_id JOIN Items i ON o.item_id = i.item_id

GROUP BY c.country_name ORDER BY revenue DESC;

### 4) Самый дорогой товар, купленный одним покупателем

| **Customer\_id** | **Customer\_name** | **MostExpensiveItemName** |
| ---------------- | ------------------ | ------------------------- |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |

**Ответ:**

SELECT o.customer_id, c.customer_name, i.item_name AS MostExpensiveItemName

FROM Orders o JOIN Customer c ON o.customer_id = c.customer_id

JOIN Items i ON o.item_id = i.item_id

WHERE o.customer_id IN (
    SELECT o2.customer_id
    FROM Orders o2
    GROUP BY o2.customer_id
    HAVING COUNT(DISTINCT o2.item_id) = 1
)

AND o.quantity * i.item_price = (
    SELECT MAX(o3.quantity * i3.item_price)
    FROM Orders o3
    JOIN Items i3 ON o3.item_id = i3.item_id
    WHERE o3.customer_id = o.customer_id
)

ORDER BY o.quantity * i.item_price DESC

### 5) Ежемесячный доход

| **Month (MM format)** | **Total Revenue** |
| --------------------- | ----------------- |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |

**Ответ:**

SELECT EXTRACT(MONTH FROM o.date_time) AS Month, SUM(i.item_price * o.quantity) AS Total_Revenue

FROM Orders o JOIN Items i ON o.item_id = i.item_id GROUP BY 1 ORDER BY Month

### 6) Найти дубликаты

Во время передачи данных произошел сбой, в таблице orders появилось несколько 
дубликатов (несколько результатов возвращаются для date_time + customer_id + item_id). 
Вы должны их найти и вернуть количество дубликатов.

```sql
-- result here
```

### 7) Найти "важных" покупателей

Создать запрос, который найдет всех "важных" покупателей,
т.е. тех, кто совершил наибольшее количество покупок после своего первого заказа.

| **Customer\_id** | **Total Orders Count** |
| --------------------- |-------------------------------|
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |

**Ответ:**

WITH FirstOrder AS (
    SELECT customer_id, MIN(date_time) AS first_order_date
    FROM Orders
    GROUP BY customer_id
)

SELECT o.customer_id AS Customer_id, COUNT(*) AS Total_Orders_Count FROM Orders o JOIN FirstOrder fo ON o.customer_id = fo.customer_id

WHERE o.date_time > fo.first_order_date GROUP BY o.customer_id ORDER BY Total_Orders_Count DESC

### 8) Найти покупателей с "ростом" за последний месяц

Написать запрос, который найдет всех клиентов,
у которых суммарная выручка за последний месяц
превышает среднюю выручку за все месяцы.

| **Customer\_id** | **Total Revenue** |
| --------------------- |-------------------|
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |

**Ответ:**

WITH MonthlyRevenue AS (
    SELECT EXTRACT(MONTH FROM date_time) AS Month, customer_id, SUM(item_price * quantity) AS Revenue
    FROM Orders
    JOIN Items ON Orders.item_id = Items.item_id
    GROUP BY EXTRACT(MONTH FROM date_time), customer_id
),

AverageRevenue AS (
    SELECT AVG(Revenue) AS Avg_Revenue
    FROM MonthlyRevenue
)

SELECT mr.customer_id, SUM(mr.Revenue) AS Total_Revenue
FROM MonthlyRevenue mr
JOIN AverageRevenue ar ON 1=1

WHERE mr.Month = EXTRACT(MONTH FROM CURRENT_DATE - INTERVAL '1 month')
GROUP BY mr.customer_id

HAVING SUM(mr.Revenue) > ar.Avg_Revenue
ORDER BY Total_Revenue DESC
