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

```sql
select count(*) CustomerCountDistinct, co.country_name Country_name from customer cu
join countries co on cu.country_code = co .country_code
where co.country_name = 'France' or co.country_name = 'Italy'
group by co.country_name;
```

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

```sql
select cu.customer_name, sum(o.quantity * i.item_price) Revenue from orders o
join customers cu on o.customer_id = cu.customer_id
join items i on o,item_id = i.item_id
group by cu.customer_name
order by revenue DESC
limit 10;
```

### 3) Общая выручка USD по странам, если нет дохода, вернуть NULL

| **Country_name** | **RevenuePerCountry** |
| ------------------------- | --------------------- |
| Italy                     | #                     |
| France                    | NULL                  |
| Mexico                    | #                     |
| Germany                   | #                     |
| Tanzania                  | #                     |

```sql
select co.country_name, 
    case
        when sum(o.quantity * i.item_price) > 0 then sum(o.quantity * i.item_price)
        else null
    end RevenuePerCountry
from countries co
left join customers cu on cu.country_code = co.country_code
left join orders o on o.customer_id = c.customer_id
join items i on o.item_id = i.item_id
group by co.country_name
```

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

```sql
with MaxPricePerCustomer AS (
    select 
        o.customer_id,
        MAX(i.item_price) AS max_price
    from Orders o
    join Items i on o.item_id = i.item_id
    group by 
        o.customer_id
),
MostExpensiveItems AS (
    select 
        o.customer_id,
        cu.customer_name,
        i.item_name,
        i.item_price
    from 
        Orders o
    join 
        Items i on o.item_id = i.item_id
    join 
        Customers cu on o.customer_id = cu.customer_id
    join 
        MaxPricePerCustomer mp on o.customer_id = mp.customer_id and i.item_price = mp.max_price
)
SELECT DISTINCT ON (customer_id)
    customer_id,
    customer_name,
    item_name AS MostExpensiveItemName
FROM 
    MostExpensiveItems;


```

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

```sql
select to_char(o.date_time, 'MM') Month,
    SUM(o.quantity * i.item_price) TotalRevenue
from orders o
join items i ON o.item_id = i.item_id
group by to_char(o.date_time, 'MM')
order by to_char(o.date_time, 'MM');
```

### 6) Найти дубликаты

Во время передачи данных произошел сбой, в таблице orders появилось несколько 
дубликатов (несколько результатов возвращаются для date_time + customer_id + item_id). 
Вы должны их найти и вернуть количество дубликатов.

```sql
select 
    o.date_time,
    o.customer_id,
    o.item_id,
    count(*) - 1 AS DuplicateCount
from Orders o
group by o.date_time, o.customer_id, o.item_id
having count(*) > 1;

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

```sql
select 
    o.customer_id,
    COUNT(*) TotalOrdersCount
from orders o
join 
    (select 
        customer_id, 
        min(date_time) first_order_date
     from orders
     group by customer_id) first_orders on o.customer_id = first_orders.customer_id
where o.date_time > first_orders.first_order_date
group by o.customer_id
order by TotalOrdersCount DESC;
```

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

```sql
with MonthlyRevenue as (
    select 
        o.customer_id,
        TO_CHAR(o.date_time, 'YYYY-MM') Month,
        SUM(o.quantity * i.item_price) Revenue
    from orders o
    join items i on o.item_id = i.item_id
    group by o.customer_id, TO_CHAR(o.date_time, 'YYYY-MM')
),
AvgMonthlyRevenue as (
    select 
        customer_id,
        AVG(Revenue) AvgRevenue
    from MonthlyRevenue
    group by customer_id
)
select 
    mr.customer_id,
    mr.Revenue TotalRevenue
from MonthlyRevenue mr
join AvgMonthlyRevenue amr on mr.customer_id = amr.customer_id
where mr.Month = to_char(date_trunc('month', current_date) - interval '1 month', 'YYYY-MM')
    and mr.Revenue > amr.AvgRevenue;
```
