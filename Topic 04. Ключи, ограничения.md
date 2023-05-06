# Topic 04. Ключи, ограничения

#### PK

**Потенциальный ключ** - подмножество *атрибутов отношения*, которое удовлетворяет требованиям:

* **Уникальность**: не может быть двух кортежей данного отношения, в которых значения  этого подмножества атрибутов совпадают.
* **Минимальность**: нельзя выбрать из подмножества *потенц. ключа* меньшее подмножество, удовлетворяющее условию уникальности.

Типы *потенц. ключа*:

* Простой - состоит ровно из одного атрибута
* Составной - состоит из двух и более атрибутов

#### Первичный ключ (*PK*, *Primary key*)

**Любой** из *потенциальных ключей*, выбранный в качестве **основного**.

Приоритет выбора *PK*:

* тот, который занимает меньше места
* не утрачивает уникальность со временем

*PK* **всегда** **существует** в таблице - например, это все атрибуты отношения.

> **Альтернативными** **ключами** называются потенц. ключи, не выбранные в кач. основного.

Типы *PK*:

* Естественный - основанный на уже существующем поле.
* Интеллектуальный - основанный на естественном ключе путем добавления дополнительного поля.

#### Суррогатный ключ

\- дополнительное поле, выступающее в качестве первичного ключа, значения которого для каждой отдельной строки генерируются искусственно.

##### Плюсы

* Уникальность
* Оптимальность по памяти
* Неизменность ключа в конкретной строке
* Прост в замене
* Прост в реализации
* Удобство при создании ссылок на другие таблицы

##### Минусы

* Уязвимость генераторов
* Неинформативность
* Склонение к пропуску нормализации
* Необходимость поддержки и суррогатного и естественного ключа
* Проблемы связанные с тем, что ключ генерируется искусственно

#### Внешний ключ (*FK*, *Foreign* *Key*)

Пусть *A*, *B* - две переменные отношения, не обязательно различные.

***FK*** - в *B* является подмножество атрибутов *B* такое, что выполняются след. требования:

* В *A* имеется такой *PK*, что он совпадает с *FK* с точностью до переименования атрибутов.
* В каждый момент времени каждое значение *FK* в *B* идентично некоторому значению  *PK* в *A*.

**Терминология**:

***Родительское*** (*главное*, *целевое*) отношение - отношение *А*, содержащее *PK*.

**Дочернее** (*подчиненное*) отношение - отношение B, содержащее в себе ссылку на сущность, в которой находятся нужные нам атрибуты. (содержащее в себе *FK*)

#### Ссылочная целостность

**Ссылочная целостность** - качество реляционной базы данных, заключающееся в **отсутствии** в любом ее отношении (таблице) **внешних ключей** (*FK*), **ссылающихся на несуществующие кортежи**.

База данных **обладает** свойством **ссылочной целостности**, когда **для любой пары** связанных внешним ключом отношений в ней **условие** ссылочной целостности **выполняется**.

#### Ограничения

* `NOT NULL` - значение всегда известно, недопустимо значение `NULL`
* `UNIQUE` - значения должны быть уникальны
* `PRIMARY KEY` - первичный ключ таблицы
* `FOREIGN KEY` - внешний ключ, необходима ссылка на другую таблицу
* `CHECK` - проверка на соответствие критерию
* `DEFAULT` - значение по умолчанию, если пользователь не задал значения

```pos
-- NOT NULL, создание таблицы
CREATE TABLE PERSON (
    ID         INTEGER      NOT NULL,
    LAST_NAME  VARCHAR(255) NOT NULL,
    FIRST_NAME VARCHAR(255) NOT NULL,
    AGE        INTEGER
);
```

```pos
-- ДОБАВЛЯЕМ UNIQUE К ID
CREATE TABLE PERSON (
    ID         INTEGER      NOT NULL UNIQUE,
    LAST_NAME  VARCHAR(255) NOT NULL,
    FIRST_NAME VARCHAR(255) NOT NULL,
    AGE        INTEGER
);

ALTER TABLE PERSON ADD UNIQUE (ID);

ALTER TABLE PERSON
ADD CONSTRAINT UC_Person UNIQUE (ID, LAST_NAME);

ALTER TABLE PERSON
DROP CONSTRAINT UC_Person;

```

```pos
-- ДОБАВЛЯЕМ PRIMARY KEY
CREATE TABLE PERSON (
    ID         INTEGER      PRIMARY KEY,
    LAST_NAME  VARCHAR(255) NOT NULL,
    FIRST_NAME VARCHAR(255) NOT NULL,
    AGE        INTEGER
);

ALTER TABLE PERSON ADD PRIMARY KEY (ID);

------------------------------------------

CREATE TABLE PERSON (
    ID         INTEGER,
    LAST_NAME  VARCHAR(255),
    FIRST_NAME VARCHAR(255) NOT NULL,
    AGE        INTEGER,
    CONSTRAINT PK_Person PRIMARY KEY (ID, LAST_NAME)
);

ALTER TABLE PERSON
ADD CONSTRAINT PK_Person PRIMARY KEY (ID, LAST_NAME);

ALTER TABLE PERSON
DROP CONSTRAINT PK_Person;

```

```pos
-- ДОБАВЛЯЕМ FOREIGN KEY
CREATE TABLE ORDER (
    ORDER_ID     INTEGER,
    ORDER_NUMBER INTEGER NOT NULL,
    PERSON_ID    INTEGER,
    
    PRIMARY KEY (ORDER_ID),
    CONSTRAINT FK_PersonOrder FOREIGN KEY (PERSON_ID) REFERENCES PERSON(PERSON_ID)
);

ALTER TABLE ORDER ADD CONSTRAINT FK_PersonOrder
FOREIGN KEY (PERSON_ID) REFERENCES PERSON(PERSON_ID);

ALTER TABLE ORDER DROP CONSTRAINT FK_PersonOrder;

------------------------------------------

CREATE TABLE ORDER (
    ORDER_ID     INTEGER PRIMARY KEY,
    ORDER_NUMBER INTEGER NOT NULL,
    PERSON_ID    INTEGER REFERENCES PERSON(PERSON_ID)
);

ALTER TABLE ORDER
ADD FOREIGN KEY (PERSON_ID) REFERENCES PERSON(PERSON_ID);

```

При определении *FK* у нас должно **совпадать количество атрибутов** в обеих таблицах, которые мы "мэтчим". **Также** в таблице, для которой атрибуты являются *PK*, должны быть заданы соответствующие ограничения – иначе будет ошибка `there is no unique constraint matching given keys for referenced table`.

#### Поддержание ссылочной целостности

* `CASCADE` - при удалении / изменении строки главной таблицы соответствующая запись дочерней таблицы будет удалена / изменена

* `RESTRICT` - строка не может быть удалена / изменена, если на нее имеется ссылка; Значение не может быть удалено / изменено, если на  него есть ссылка

* `NO ACTION`

  * Имеет сходства с `RESTRICT`, но проверка происходит в конце транзакции

  * Для разницы с `RESTRICT` нужно явно прописать в транзакции выражение (`SET CONSTRAINTS`)

    ```pos
    SET CONSTRAINTS { ALL | name [, ...] } { DEFERRED | IMMEDIATE }
    ```

* `SET NULL` - при удалении записи главной таблицы, соотв. значение дочерней таблицы становится `NULL`

* `SET DEFAULT` - аналогично `SET NULL`, но вместо `NULL` устанавливается некоторое значение по умолчанию

  ```pos
  CREATE TABLE ORDER (
      ORDER_ID     INTEGER,
      ORDER_NUMBER INTEGER NOT NULL,
      PERSON_ID    INTEGER,
      
      PRIMARY KEY (ORDER_ID),
      CONSTRAINT FK_PersonOrder FOREIGN KEY (PERSON_ID)
          REFERENCES PERSON(PERSON_ID)
              ON DELETE RESTRICT
              ON UPDATE RESTRICT
  );
  ```

* `CHECK`

  ```pos
  CREATE TABLE PERSON (
      ID         INTEGER      NOT NULL,
      LAST_NAME  VARCHAR(255) NOT NULL,
      FIRST_NAME VARCHAR(255) NOT NULL,
      AGE        INTEGER      CHECK (AGE >= 18)
  );
  
  ALTER TABLE PERSON ADD CHECK (AGE >= 18);
  
  ------------------------------------------
  
  CREATE TABLE PERSON (
      ID          INTEGER      NOT NULL,
      LAST_NAME   VARCHAR(255) NOT NULL,
      FIRST_NAME  VARCHAR(255) NOT NULL,
      AGE         INTEGER,
      CITY        VARCHAR(255),
      CONSTRAINT CHK_Person CHECK (AGE >= 18 AND CITY = 'Moscow')
  );
  
  ALTER TABLE PERSON ADD CONSTRAINT CHK_Person
  CHECK (AGE >= 18 AND CITY = 'Moscow');
  
  ALTER TABLE PERSON DROP CONSTRAINT CHK_Person;
  ```

* `DEFAULT`

  ```pos
  CREATE TABLE ORDER (
      ORDER_ID     INTEGER PRIMARY KEY,
      ORDER_NUMBER INTEGER NOT NULL,
      ORDER_DATE   DATE    DEFAULT now()::date
  );
  
  ALTER TABLE ORDER;
  ALTER COLUMN ORDER_DATE DROP DEFAULT;
  ```

#### Посмотреть ограничение в таблице

```pos
-- Список колонок, попадающих под ограничение:
SELECT * FROM information_schema.constraint_column_usage;

-- Все имеющиеся в базе ограничения:
SELECT * FROM information_schema.table_constraints;

-- Уникальные и ключевые (PK, FK) поля таблиц
SELECT * FROM information_schema.key_column_usage;

-- Информация по ограничениям с типом `CHECK`
SELECT * FROM information_schema.check_constraints;

-- Информация по ограничениям с типом `DEFAULT` и `NOT NULL`
SELECT * FROM information_schema.columns;
```

#### Подробнее про ограничения

https://metanit.com/sql/postgresql/2.4.php
