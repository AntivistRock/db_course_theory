### Общие табличные выражения (СТЕ)

#### Теор. справка

**CTE** (**Common Table Expression**) (синоним к подзапросу; живет ровно в момент исполнения одного конкретного запроса) - именованный временный набор данных, используемый в запросе.

##### Запросы `WITH`

\- предоставляет способ записывать дополнительные операторы для применения в больших запросах. Эти операторы, которые также называются **CTE**, можно представить как определения временных таблиц, существующих только для одного запроса. **Дополнительным оператором** в предложении `WITH` может быть:

* `SELECT`
* `INSERT`
* `UPDATE`
* `DELETE`

а сам `WITH` **присоединяется к основному оператору**, которым также могут быть перечисленные выше 4 оператора.

#### Синтаксис

```postgresql
WITH [RECURSIVE] cte_query_name
    AS (cte_query)
main_query;
```

Это способ задания временного набора результатов для использования в инструкциях DML.

Позволяет:

- Упростить запрос
- Реализовать рекурсивные запросы

Пример:

```postgresql
WITH 
    regional_sales AS (
        SELECT 
            region, 
            SUM(amount) AS total_sales
        FROM 
            orders
        GROUP BY 
            region
    ),
    top_regions AS (
        SELECT 
            region
        FROM 
            regional_sales
        WHERE 
            total_sales > (SELECT SUM(total_sales) / 10 FROM regional_sales)
    )
SELECT 
    region, 
    product, 
    SUM(quantity) AS product_units, 
    SUM(amount) AS product_sales
FROM 
    orders
WHERE 
    region IN (SELECT region FROM top_regions)
GROUP BY 
    region, 
    product;
```

#### Рекурсивные запросы

Необязательное указание `RECURSIVE` превращает `WITH` из просто удобной синтаксической конструкции в средство реализации того, что невозможно в стандартном `SQL`. Используя `RECURSIVE`, запрос `WITH` **может обращаться к собственному результату**.

##### Синтаксис

```postgresql
with recursive <cte_name> (<parameters>) as (
	<recursive base>    -- Нерекурсивное выражение
	union all           -- UNION или UNION ALL
	<recursion step>    -- Рекурсивное выражение, может ссылаться на результат запроса
)
<main query>;
```

##### Пример

```postgresql
WITH RECURSIVE t(n) AS (
    VALUES (1)      -- данные, для которых рекурсия не нужна
    
    UNION ALL
    
    SELECT 
        n + 1    -- рекурсивная часть
    FROM 
        t
    WHERE 
        n < 100
)
SELECT 
    sum(n)
FROM 
    t;
```

##### Вычисление рекурсивного CTE

1. Определить значение нерекурсивного выражения. Если указан `UNION`, то отбросить ряды-дубликаты. Поместить оставшиеся ряды в результат рекурсивного запроса, а также во временную рабочую таблицу.
2. До тех пор, пока временная таблица не пустая:
   1. Определить значение реrурсивного выражения, заменив текущее содержимое рабочей таблицы рекурсивной ссылкой на себя. Если указан `UNION`, то отбросить ряды-дубликаты. Включить все оставшиеся ряды в результат рекурсивного запроса, а также во временную таблицу
   2. Заменить содержимое рабочей таблицы содержимым промежуточной таблицы и очистить промежуточную таблицу

Рекурсивные запросы **обычно применяются для работы с иерархическими или древовидными структурами данных**. В качестве полезного примера можно привести запрос, находящий все непосредственные и косвенные составные части продукта, используя только таблицу с прямыми связями:

```postgresql
WITH RECURSIVE 
    included_parts(sub_part, part, quantity) AS (
        SELECT 
            sub_part, 
            part, 
            quantity
        FROM 
            parts
        WHERE 
            part = 'our_product'
        
        UNION ALL
        
        SELECT 
            p.sub_part, 
            p.part, 
            p.quantity
        FROM 
            included_parts pr,
            parts p
        WHERE 
            p.part = pr.sub_part
    )
SELECT 
    sub_part, 
    SUM(quantity) AS total_quantity
FROM 
    included_parts
GROUP BY 
    sub_part;

```

#### Изменение данных в `WITH`

