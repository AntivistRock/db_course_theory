### Транзакции

#### Теор. справка

**Транзакция** – это объект, **группирующий последовательность операций**, которые **должны быть выполнены как единое целое**. Обеспечивает переход БД из одного целостного состояния в другое.

Транзакции обеспечивают целостность БД в условиях:

- Параллельной обработки данных
- Физических отказов диска
- Аварийного сбоя электропитания
- И других

Транзакции обладают **4 характеристиками**, удовлетворяющими парадигме **ACID**:

1. *Atomic* – атомарные;
2. *Consistent* – согласованные;
3. *Isolated* – изолированные;
4. *Durable* – долговечные, устойчивые.

##### Атомарность транзакций (Atomicity)

- Транзакция должна представлять собой атомарную (неделимую) единицу работы;
- Должны быть выполнены либо все операции, входящие в транзакцию, либо ни одна из них;
- Следовательно, в случае невозможности выполнить все операции, все внесённые изменения должны быть отменены:
  - Commit – совершение транзакции
  - Rollback – отмена транзакции

##### Согласованность транзакций (Consistency)

- По завершению транзакции все данные должны остаться в согласованном состоянии
- При выполнении транзакции необходимо выполнить все правила реляционной СУБД:
  - Проверки выполнения ограничений (домены, индексы уникальности, внешние ключи, проверки, правила и т.д.)
  - Обновление индексов;
  - Выполнение триггеров
  - И другие

##### Изоляция транзакций (Isolation)

- Изменения в данных, выполняемые в пределах транзакции, должны быть изолированы от всех изменений, выполняемых в других транзакциях, до тех пор, пока транзакция не совершена;
- Выделяют различные уровни изоляции – для достижения компромисса между степенью распараллеливания работы с БД и строгостью выполнения принципа непротиворечивости:
  - Чем выше уровень изоляции, тем выше степень непротиворечивости данных;
  - Чем выше уровень изоляции, тем ниже степень распараллеливания и тем ниже степень доступности данных.

##### Стандартная классификация проблем с уровнями изоляции:

- **P1**: dirty read – **«грязное» чтение**

  > 1. Транзакция *Т1* вносит изменения в ряд таблицы.
  > 2. *Т2* читает этот ряд после внесения изменений *Т1*, но до совершения *Т1*.
  > 3. Если *Т1* будет отменена, то данные, считанные *Т2*, будут некорректными.

- **P2**: non-repeatable read – **невоспроизводимое чтение**

  > 1. Транзакция *Т1* читает ряд.
  > 2. Транзакция *Т2* после этого вносит изменения в этот ряд или удаляет его.
  > 3. Если *Т1* после этого считает этот же ряд снова, то получит новый результат по сравнению с первым считыванием.

- **P3**: phantom read – **фантомное чтение**

  > 1. Транзакция *Т1* читает набор рядов N, удовлетворяющих некоторому условию.
  > 2. После этого *Т2* выполняет SQL запросы, создающие новые ряды, удовлетворяющие этому условию.
  > 3. Если *Т1* повторит запрос с тем же условием, то получит другой набор рядов.

##### Долговечность, устойчивость (прочность) транзакций (Durability)

- Если транзакия была совершена, её результат должен сохраниться в системе, несмотря на сбой системы;
- Если транзакиция не была совершена, её результат может быть полностью отменён вслед за сбоем системы.

#### Transaction Controe Language

```postgresql
BEGIN TRANSACTION transaction_mode [, ...]
   ISOLATION LEVEL { SERIALIZABLE | REPEATABLE READ | READ COMMITTED | READ UNCOMMITTED }
   READ WRITE | READ ONLY
   [NOT] DEFERRABLE

BEGIN / START
   COMMIT
   ROLLBACK

SAVEPOINT name
   ROLLBACK TO SAVEPOINT name
   RELEASE SAVEPOINT name
```

##### **Границы транзакций**

```postgresql
BEGIN;
INSERT ...;
UPDATE ...;
DELETE ...;
COMMIT;

BEGIN;
INSERT ...;
UPDATE ...;
DELETE ...;
ROLLBACK;
```

#### Уровни изолированности 

В идеальном мире параллельные транзакции изолированны (идеально уживаются друг с другом в базе), но в реальном мире такое невозможно. **Характеристика соответствия базы свойству изолированности называется уровнем изолированности**. Выделяют **4 уровня изолированности**:

![img](https://gitlab.atp-fivt.org/courses-public/db2023-supplementary/global/-/raw/main/practice/seminars/09-tcl-dcl/img/img_1.png)

* **READ UNCOMMITTED**: помогает бороться с потерянными обновлениями при изменениях одного и того же блока данных несколькими транзакциями.

* **READ COMMITTED**: помогает бороться с чтением "грязных" данных, измененных впоследствии откатившейся транзакцией

* **REPEATABLE READ**: помогает бороться с изменением в данных при повторном чтении одного и того же блока данных в рамках одной транзакции

* **SERIALIZABLE**: аналогично **REPEATABLE READ**, но применимо к `INSERT`, а не `UPDATE`

### Разграничение доступов в PostgreSQL

**Роль** – множество пользователей БД

- Может владеть объектами в базе
- Может иметь определенный доступ к некоторым объектам базы, не являясь их владельцем
- Может выдавать доступы на некоторые объекты в базе другим ролям
- Может предоставлять членство в роли другой роли

#### Создание пользователя

```postgresql
CREATE USER == CREATE ROLE

CREATE USER name [[WITH] option [...]]
where option can be:
SUPERUSER | NOSUPERUSER
| CREATEDB | NOCREATEDB
| CREATEROLE | NOCREATEROLE
| INHERIT | NOINHERIT
| LOGIN | NOLOGIN
| REPLICATION | NOREPLICATION
| BYPASSRLS | NOBYPASSRLS
| CONNECTION LIMIT connlimit
| [ENCRYPTED] PASSWORD 'password' | PASSWORD NULL
| VALID UNTIL 'timestamp'
| IN ROLE role_name [, ...]
| IN GROUP role_name [, ...]
| ROLE role_name [, ...]
| ADMIN role_name [, ...]
| USER role_name [, ...]
| SYSID uid
```

```postgresql
CREATE ROLE jonathan LOGIN;

CREATE USER davide
WITH PASSWORD 'jw8s0F4';

CREATE ROLE Miriam
WITH LOGIN PASSWORD 'jw8s0F4'
VALID UNTIL '2005-01-01';
```

#### Data Control Language (DCL)

* Позволяет настраивать доступы к объектам

* Поддерживает 2 типа действий:

  - `GRANT` – выдача доступа к объекту

  - `REVOKE` – отмена доступа к объекту

**Права, которые можно выдать на объект:**

- `SELECT`
- `INSERT`
- `UPDATE`
- `DELETE`
- `TRUNCATE`
- `REFERENCES`
- `TRIGGER`
- `CREATE`
- `CONNECT`
- `TEMPORARY`
- `EXECUTE`
- `USAGE`

##### Синтаксис `GRANT`

```postgresql
GRANT {{SELECT | INSERT | UPDATE | DELETE | TRUNCATE |
    REFERENCES | TRIGGER}
[, ...] | ALL [PRIVILEGES]}
ON {[ TABLE] table_name [, ...]
    | ALL TABLES IN SCHEMA schema_name [, ...]}
TO role_specification [, ...] [WITH GRANT OPTION]
```

##### Синтаксис `REVOKE`

```postgresql
GRANT {{SELECT | INSERT | UPDATE | DELETE | TRUNCATE |
    REFERENCES | TRIGGER}
[, ...] | ALL [PRIVILEGES]}
ON {[ TABLE] table_name [, ...]
    | ALL TABLES IN SCHEMA schema_name [, ...]}
TO role_specification [, ...] [WITH GRANT OPTION]
```

```postgresql
GRANT ALL PRIVILEGES ON kinds TO manuel;
REVOKE ALL PRIVILEGES ON kinds FROM manuel;

GRANT SELECT ON kinds TO manuel WITH GRANT OPTION; -- manuel может выдавать права на SELECT на табличку kinds
```

*Примечание:* Если мы хотим забрать `grant option` у `manuel`, то нужно использовать `CASCADE`, чтобы забрать права у всех, кому он выдавал права. Иначе при попытке отозвать права у `manuel`, запрос упадет с ошибкой.

#### Пример

```postgresql
CREATE USER davide WITH PASSWORD 'jw8s0F4';

GRANT CONNECT ON DATABASE db_prod TO davide;

GRANT USAGE ON SCHEMA my_schema TO davide;

GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA my_schema
TO davide;
```

