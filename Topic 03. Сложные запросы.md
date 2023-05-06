### Сложные SQL запросы

#### Структура

```pos
SELECT
  [ALL | DISTINCT [ON (expression [, ...] )] ]
  [* | expression [AS output_name] [, ...] ]
[FROM from_item [, ...] ]
[WHERE condition]
[GROUP BY grouping_element [, ...]]
[HAVING condition]
[ORDER BY expression [ASC | DESC | USING operator] [NULLS {FIRST | LAST}]
     [, ...]]
[LIMIT {count | ALL}]
[OFFSET start [ROW | ROWS]]
[FETCH {FIRST | NEXT} [count] {ROW | ROWS} ONLY]

```

#### Операции JOIN

##### Как работает?

* `SELECT` говорит, какие столбцы будут в результирующей таблице
* Вид `JOIN` определяет сочетания значений каждого столбца в таблице
* Условие соединения определяет, какие ряды войдут в результирующую таблицу

##### Виды соединений и их описание

* `[INNER] JOIN`

  ![img](https://gitlab.atp-fivt.org/courses-public/db2023-supplementary/global/-/raw/main/practice/seminars/03-complex-sql/img/img_0.png)

  1. Каждая строка одной таблицы сопоставляется с каждой строкой второй таблицы

  2. Для полученной соединённой строки проверяется условие соединения

  3. Если условие истинно, в набор результатов добавляется соединённая строка

* `LEFT [OUTER] JOIN` | `RIGHT [OUTER] JOIN` | `FULL [OUTER] JOIN`

  ![img](https://gitlab.atp-fivt.org/courses-public/db2023-supplementary/global/-/raw/main/practice/seminars/03-complex-sql/img/img_3.png)

  В набор результатов входят все ряды либо одной, либо обеих таблиц.

* `CROSS JOIN` - декартово произведение 2-х таблиц

##### `USING`

Если в `join_condition` входят столбцы с одинаковым именем, можно использовать сокращенный синтаксис `USING`:

```pos
ON left_table.a = right_table.a AND left_table.b = right_table.b
USING (a, b)
```

##### `NATURAL`

```pos
SELECT select_list 
FROM T1 NATURAL JOIN T2;
```

Принцип работы `NATURAL`:

- Аналогичен `USING` с указанием всех одноимённых столбцов
- Если одноимённых столбцов нет, то аналогичен `ON TRUE` - аналогично `CROSS JOIN`

#### Подзапросы

\- это запрос, содержащийся в другом SQL-запросе.

Запрос, содержащий другой подзапрос, называется *содержащим выражением*.

##### Свойства

* Всегда заключен в круглые скобки и обычно выполняется до содержащего выражения
* Могут вкладываться друг в друга
* В `SELECT` можно использовать во всех разделах, кроме `GROUP BY`.

##### Классификация

**По взаимодействию с содержащим выражением**:

* Связанные - ссылаются на столбцы основного запроса
  * Полезно использовать *aliases* (`SELECT ... AS ...`)
  * Обязательно использовать *aliases*, если в запросе и подзапросе одна и та же таблицв
  * Выполняется для каждой строки содержащего выражения
* Несвязанные - полностью не зависят от основного запроса.
  * Выполняются перед выполнением содержащего выражения.

**По результату выполнения**:

* Скалярные - 1 столбец, 1 строка
* Не скалярные

![img](https://gitlab.atp-fivt.org/courses-public/db2023-supplementary/global/-/raw/main/practice/seminars/03-complex-sql/img/img_10.png)

##### Предикаты подзапросов для одного атрибута

* `EXISTS` - *true* <=> в результате подзапроса есть хотя бы одна строка,  *false* иначе.

  ```pos
  SELECT 
      SupplierName
  FROM 
      Suppliers
  WHERE 
      EXISTS(
          SELECT --Подзапрос с 1 атрибутом
              ProductName
          FROM 
              Products
          WHERE 
              SupplierId = Suppliers.supplierId
              AND Price < 20
      );
  
  ```

* `IN` - проверка наличия значения в результате подзапроса

  ```pos
  SELECT 
      emp_id, 
      fname, 
      lname, 
      title
  FROM 
      employee
  WHERE 
      emp_id IN(
          SELECT 
              superior_emp_id
          FROM 
              employee
      );
  ```

* `ALL` - *true*, если результат пуст, либо результат предиката *true* для всех строчек подзапроса.

  ```pos
  SELECT 
      EMP_NO
  FROM 
      EMP
  WHERE 
      DEPT_NO = 65
      AND EMP_SAL >= ALL(
          SELECT 
              EMP1.EMP_SAL
          FROM 
              EMP AS EMP1
          WHERE 
              EMP.DEPT_NO = EMP1.DEPT_NO
      );
  ```

* `ANY` - аналогично `ALL`, но *false* вместо *true*.

##### Создание таблицы по результатам подзапроса

* `CREATE TABLE AS` - создает таблицу и наполняет ее результатом `SELECT`

  Имена столбцов можно переопределить, явно добавив список новых имен

  ```pos
  CREATE TABLE NEW_TABLE AS
      SELECT 
          *
      FROM 
          OLD_TABLE;
  ```