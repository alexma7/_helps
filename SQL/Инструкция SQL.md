Инструкция
========================
Содержание
-------------------------
1. [INSERT, DELETE, UPDATE, CREATE](#ins-del-upd-create)
2. [CURSOR, WHILE](#CURSOR-WHILE)
3. [TRANSACTIONS](#TRANSACTIONS)
4. [CREATE PROCEDURE](#CREATE-PROCEDURE) 
5. [Функции: "CAST, DATEADD, CONVERT, DATEDIFF, CEILING, FLOOR"](#CAST,-DATEADD,-CONVERT,-DATEDIFF)
6. [Математические функции: ABS, CEILING, FLOOR, SQRT, SQUARE, POWER, LOG](#math-func)
6. [TRIGGER](#TRIGGER)
6. [TRY, CATCH, @@TRANCOUNT, THROW](#TRY,-CATCH,-@@TRANCOUNT,-THROW)
6. [Обобщенное табличное выражение "CTE" - WITH](#WITH)
6. [ROW_NUMBER](#ROW_NUMBER)
6. [ERROR, INTO, OUTPUT, INSERTED](#ERROR,-INTO,-OUTPUT,-INSERTED)
6. [CASE](#)
0. [Объединения](#UNION)
0. [Представление VIEW](#VIEW)
0. [](#)
0. [](#)
0. [](#)
0. [](#)
0. [](#)
0. [](#)


# INSERT, DELETE, UPDATE, CREATE, DROP, <a id="ins-del-upd-create"></a>
```SQL
INSERT INTO dbo.tbl_name (table_field2) 
VALUES (CONVERT(Point, '3,4')); // Вместо VALUES можно вставить SELECT

UPDATE tbl_name
SET table_field1 = @table_field1
WHERE table_field2 = @table_field2

--апдейт при сравнении двух итоговых таблиц, 	
UPDATE tbl_name2 
SET tbl_name2.table_field1 = TN.table_field1
FROM #tbl_name TN
WHERE  tbl_name2.table_field2 = TN.table_field2 
   AND tbl_name2.table_field3 = TN.table_field3 

--Версия с LEFT JOIN 
UPDATE t2
SET t2.email = t1.email, t2.podr = t1.email 
FROM #tbl_name t1
    LEFT JOIN tbl_name2 t2
	ON t1.idUser = t2.idUser


DELETE FROM baza
WHERE Cod = @cod

## Удаление таблиц
DROP TABLE tbl_name ;
--Удаление временной таблицы, если она существует
IF OBJECT_ID('tempdb..#tbl_name') IS NOT NULL DROP TABLE #tbl_name

--Обновление через JOIN
UPDATE  tbl2
SET tbl2.column = 1
FROM tbl_name tbl1
   LEFT JOIN tbl_name2 tbl2 ON tbl1.id = tbl2.id


--Создание таблицы
CREATE TABLE #tbl_name (
    id INT,
    name NVARCHAR(2000)
);


```

# CURSOR, WHILE, курсор<a id="CURSOR-WHILE"></a>
```SQL
--Описываем и открываем курсор
DECLARE my_coursor CURSOR FOR
--Вставляем в курсор номера из таблицы SprParentMobil, что бы произвести их перечисление в последующих операциях
SELECT tbl_field1 FROM tbl_name WHERE actual = 1
--Открываем курсор, в котором находятся номера из предыдущего запроса
OPEN my_coursor
--Вставляем данные из курсора в переменную, для дальнейшего использования
FETCH FROM my_coursor INTO @tbl_field1

WHILE @@FETCH_STATUS = 0
BEGIN
	--Выполняем код с полученным значением от курсора
	--КОД
	--После использования значения из курсора, нам необходимо получить следующее, для этого используется следующая строка
	FETCH NEXT FROM my_coursor INTO @tbl_field1;
END
--Закрываем курсор
CLOSE my_coursor; 
--Уничтожаем его
DEALLOCATE my_coursor;
```



# TRANSACTIONS <a id="TRANSACTIONS"></a>
```SQL
--Начало транзакции
BEGIN TRAN
--Подтверждение транзакции
COMMIT TRAN;
--Откат транзакции (обычно ставиться в блок CATCH)
ROLLBACK TRAN;
```

# CREATE PROCEDURE <a id="CREATE-PROCEDURE"></a>
```SQL
USE [База данных]
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE PROCEDURE [dbo].[Название процедуры]  
	@param AS INT, --параметр 1
	@date_begin AS DATETIME = NULL -- параметр 2 значение null, если параметр не передан
AS
BEGIN
	--Сама процедура
END
```

# CAST, DATEADD, CONVERT, DATEDIFF <a id="CAST,-DATEADD,-CONVERT,-DATEDIFF"></a>
### Преобразовываем формат датаВремя в дату
```SQL
CAST(GETDATE() AS DATE)
```

### Отнимаем 4 часа. Параметры: week, hour, minute, second, millisecond
```SQL		
DATEADD(HOUR,-4, CAST(@date_begin AS DATETIME))
```

### Разница двух дат.Параметры: week, hour, minute, second, millisecond
```SQL		
DATEDIFF(day, date_begin, date_end)
```

### Конвертируем дату время в строку и ставим формат времени
```SQL		
CONVERT(NVARCHAR, @date_begin, 21)
```

# TRIGGER <a id="TRIGGER"></a>
Имя триггера может быть только уникально в пределах базы данных

```SQL
USE [DB]
GO

SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TRIGGER [dbo].[TABLE_trig_journal]  
ON [dbo].[TABLE]
AFTER UPDATE, INSERT, DELETE      
AS  
BEGIN
	DECLARE
		@name_table NVARCHAR(255)
		,@num_operation_ins SMALLINT = 1
		,@num_operation_del SMALLINT = 2 
		,@str_operation_ins NVARCHAR(255) = ''
		,@str_operation_del NVARCHAR(255) = ''

	--Получаем имя таблицы
	SELECT  @name_table = tbl.name 
	FROM sysobjects AS TRG
		INNER JOIN sysobjects tbl ON trg.parent_obj = tbl.id
	WHERE  trg.id = @@PROCID


	-- SELECT * FROM pc_journal     SELECT * FROM pc_spr_journal_operation

	-- Если обновление 
	IF EXISTS( SELECT 1 FROM  DELETED) AND EXISTS( SELECT 1 FROM  INSERTED)
	BEGIN
		SET @num_operation_ins = 0
		SET @num_operation_del = 0
		SET @str_operation_ins = 'Стало: '
		SET @str_operation_del = 'Было: '
	END

	IF EXISTS(SELECT 1 FROM  DELETED)
		INSERT INTO pc_journal(
		  id_parent_shop, id_parent_mobil, id_operation, txt, name_table, updated_user, updated_by)
		SELECT
			id_parent_shop
			,NULL
			,@num_operation_del
			,@str_operation_del + 'id = ' + CAST(id AS NVARCHAR(10)) + ', name = ' + name 
			,@name_table
			,SUSER_SNAME()
			,GETDATE()
		FROM DELETED

	IF EXISTS( SELECT 1 FROM  INSERTED)
		INSERT INTO pc_journal(
		  id_parent_shop, id_parent_mobil, id_operation, txt, name_table, updated_user, updated_by)
		SELECT 
			id_parent_shop
			,NULL
			,@num_operation_ins
			,@str_operation_ins + 'id = ' + CAST(id AS NVARCHAR(10)) + ', name = ' + name 
			,@name_table 
			,SUSER_SNAME()
			,GETDATE()
		FROM INSERTED
END
```

# TRY, CATCH, @@TRANCOUNT, THROW <a id="TRY,-CATCH,-@@TRANCOUNT,-THROW"></a>
```SQL
BEGIN TRY
  --Выполняем какой-то запрос
END TRY 
BEGIN CATCH
  --Заходим сюда, если появляется ошибка в запросе
  IF @@TRANCOUNT > 0 
  	ROLLBACK; --Проверяем открытые транзакции и, если они не закрыты, то закрываем их
	
	--Производим запись ошибок в журнал
	INSERT journal (txt, error_txt, error, date_time, user_name, id)
	SELECT 
		Ошибка в процедуре + CAST(ERROR_PROCEDURE() AS NVARCHAR(200)) + На линии + CAST(ERROR_LINE() AS NVARCHAR(200)),
		ERROR_MESSAGE(), 
		1, 
		GETDATE(), 
		SUSER_SNAME() AS user_name,  
		@id 
	THROW;  -- команда выполняет вывод сообщений об ошибках в окно с сообщениями
END CATCH
```
# Обобщенное табличное выражение "CTE" - WITH  <a id="WITH"></a>
```SQL
	WITH (CTE1) AS -- первый запрос
		(
			SELECT f1, f2, f3
			FROM table_field1
		),
	CTE2 AS
		(
			SELECT f1, f2
			FROM table_field2
		)	
	-- После объявления, мы можем работать с выражениями, как с обыными таблицами
	SELECT T1.f1, T2.f1
	FROM CTE1 T1
		LEFT JOIN CTE2 T2 ON T1.id = T2.t1_id
	WHERE T1.id > 15

	--Пример. Получаем уникальные значения почты
	WITH SRC 
	AS 
	(	
		SELECT email FROM User_email WHERE podr IN
			(	--Заносим список в функцию, котоая конвертирует строку в таблицу
				SELECT *
				FROM dbo.ConverStrToTbl(@list_podr, ',', 1)
			)
	)
	--Формируем строку
	SELECT @list_user_email = ISNULL(@list_user_email + '; ','') + email 
	FROM SRC;

			--Удалим первые 2 символа
	SET @list_user_email = STUFF( @list_user_email, 1, 2, '' )
```
## ОКОННЫЕ ФУНКЦИИИ <a id="ROW_NUMBER"></a>

Создает виртуальное окно, в котором можно 
```SQL
OVER () 
```

## кадрирование в агрегатных функциях
1. Определяется упорядочение внутри секции ORDER BY
2. Затем можно выделить набор строк между двемя границами (ROWS или RANGE). Обязательное присутствие упорядочивания окна.



## ранжирующие функции
1. ROW_NUMBER - функция нумерации строк
```SQL
-- Нумеруем записи по equipment_id, заново начинаем нумерацию, при изменении этого поля  и сортируем с последней по времени записи
ROW_NUMBER() OVER(PARTITION BY equipment_id ORDER BY date_begin DESC) AS row_num 
```
2. RANK - возвращает ранг каждой строки. В отличие от ROW_NUMBER, тут идет анализ значений
```SQL
-- Нумеруем записи по equipment_id, если date_begin равны и данные находятся в одной части, то устанавливаются одинаковые порядковые числа. А последующие порядковые числа пропускаются. Пример: если есть 4 записи, 2 из них с одинаковой датой, то номера будут 1,2,2,4
RANK() OVER(PARTITION BY equipment_id ORDER BY date_begin DESC) AS row_num 
```
3. DENSE_RANK - возвращает ранг каждой строки. В отличие от RANK в случае нахождения одинаковых значений возвращает ранг без пропуска следующего
```SQL
-- Нумеруем записи по equipment_id, если date_begin равны и данные находятся в одной части, то устанавливаются одинаковые порядковые числа. А последующие порядковые числа НЕ пропускаются, в отличие от RANK(). Пример: если есть 4 записи, 2 из них с одинаковой датой, то номера будут 1,2,2,3
DENSE_RANK() OVER(PARTITION BY equipment_id ORDER BY date_begin DESC) AS row_num 
```
4. NTILE - делит данные на группы по определенному столбцу. В функцию передаетя параметр кол-ва групп. Если кол-во строк не будет четно кол-ву групп, то в первых группах строк будет больше. Если мы применим эту функцию к запросу из 6 строк, то получим 1 первых двух, номер 1, у 3, 4 строки, номер 2 И у 5,6 строки номер 3
```SQL

NTILE(3) OVER( ORDER BY date_begin DESC) AS row_num 
```

## функции смещения
1. LEAD - функция обращается к данным из след. строки. Имеет 3 парметра:столбец, который необходимо вернуть; кол-во строк для смещения (по умолч. = 1); значение, которое необходимо вернуть, если после смещеиня NULL
```SQL
-- Нумеруем записи по equipment_id, заново начинаем нумерацию, при изменении этого поля  и сортируем с последней по времени записи
ROW_NUMBER() OVER(PARTITION BY equipment_id ORDER BY date_begin DESC) AS row_num 
```
2. LAG - обращается к данным из предыдущей строки. Параметры, как в LEAD
3. FIRST_VALUE - возвращает первое значение из набора данных
4. LAST_VALUE - возвращает последнее значение из набора данных
### В предложении оконного кадры указывают (ROWS или RANGE):
#### Параметры ROWS:
1. UNBOUNDED PRECEDING - начало секции
2. UNBOUNDED FOLLOWING - конецсекции
3. CURRENT ROW - это текущая строка
5. <n> PRECEDING - <n> строк перед
6. <n> FOLLOWING - <n> строк после

## Аналитические функции
1. CUME_DIST – вычисляет и возвращает интегральное распределение значений в наборе
данных. Иными словами, она определяет относительное положение значения в наборе;
2. PERCENT_RANK – вычисляет и возвращает относительный ранг строки в наборе данных;
3. PERCENTILE_CONT – вычисляет процентиль на основе постоянного распределения значения
столбца. В качестве параметра принимает процентиль, который необходимо вычислить; 
4. PERCENTILE_DISC – вычисляет определенный процентиль для отсортированных значений в
наборе данных. В качестве параметра принимает процентиль, который необходимо
вычислить.
У функций PERCENTILE_CONT и PERCENTILE_DISC синтаксис немного отличается, столбец, по
которому сортировать данные, указывается с помощью ключевого слова WITHIN GROUP.
Вот пример использования аналитических оконных функции в T-SQL.
```SQL
SELECT product_id, product_name, category, price,
	CUME_DIST() OVER (PARTITION BY category ORDER BY price) AS [CUME_DIST],
	PERCENT_RANK() OVER (PARTITION BY category ORDER BY price) AS [PERCENT_
	PERCENTILE_DISC(0.5) WITHIN GROUP(ORDER BY product_id) OVER(PARTITION BY
	PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY product_id) OVER(PARTITION BY
FROM goods;
```
#### Примеры
#### Два запроса делающие одно и то же, UNBOUNDED PRECEDING можно опустить, по умолчанию окно и так до текущей записи.
```SQL
SUM([поля из кадра]) OVER(ORDER BY [поле сортировки] ROWS  UNBOUNDED PRECEDING ) 
SUM([поля из кадра]) OVER(ORDER BY [поле сортировки] ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)   
```

#### Делим окно на секции по **году **с сортировкой по **дате**. Создаем кадр от первой записи в секции до текущей. Получаем накопительную сумму в каждой секции, сумма строки = сумме строк выше + текущая в пределах секции.
```SQL
SUM([поля из кадра]) OVER(PARTITION BY YEAR(Дата) ORDER BY [Дата] ROWS  UNBOUNDED PRECEDING ) 

--можно сделать расчет по каждому месяцу, 
SUM([поля из кадра]) OVER(PARTITION BY YEAR(Дата), MONTH(Дата) ORDER BY [Дата] ROWS  UNBOUNDED PRECEDING )   
```


#### суммируем записи по IdTechnology, заново начинаем сумму, при изменении этого поля
```SQL
SUM(столбец) OVER (PARTITION BY IdTechnology) AS [SUM]
```

#### Получаем последнюю запись в таблице
```SQL
ROW_NUMBER() OVER(ORDER BY dataBegin) AS OBJECT_ID
```


# ERROR, INTO, OUTPUT, INSERTED <a id="ERROR,-INTO,-OUTPUT,-INSERTED"></a>
### Функции обработки ошибок
* функция ERROR_NUMBER() возвращает номер ошибки;
* функция ERROR_SEVERITY() возвращает степень серьезности ошибки;
* функция ERROR_STATE() возвращает код состояния ошибки;
* функция ERROR_PROCEDURE() возвращает имя хранимой процедуры или триггера, в котором произошла ошибка;
* функция ERROR_LINE() возвращает номер строки, которая вызвала ошибку, внутри подпрограммы;
* функция ERROR_MESSAGE() возвращает полный текст сообщения об ошибке. Текст содержит значения подставляемых параметров, таких как длина, имена объектов или время.

### Производим запись ошибок в журнал
```SQL
INSERT journal (txt, errorTxt, error, dTime, userName, idParentMobil)
SELECT 
   'Ошибка в процедуре '+ CAST(ERROR_PROCEDURE() AS NVARCHAR(200)) +
' На линии '+ CAST(ERROR_LINE() AS NVARCHAR(200)),
   ERROR_MESSAGE(), 
   1, 
   GETDATE(), 
   SUSER_SNAME() AS userName,  
   @idParentMobil 
```

### Во временные таблицы можно заносить данные, которые вставились в другую таблицу
```SQL
INSERT INTO table_name (*)
    OUTPUT inserted.event_id,  @event_type_new, inserted.[OBJECT_ID] 
    INTO @temp_tbl
SELECT *
FROM 
```
### Заносим информацию при вставке в журна, такая же конструкция будет работать с DELETE и UPDATE, для получения старых данных используем delete.field
```SQL
INSERT table (field1, field2)
   OUTPUT inserted.field1
   INTO journal(field1, field2)
SELECT field1, field2

UPDATE SM
SET name_mobil = @name_mobil
   ,id_type_mobil = @id_type_mobil
   ,actual = @actual
   ,id_group_equip = @id_group_equip
		--SELECT * 
   OUTPUT	 @id_SM
		,inserted.id_parent_mobil
		,inserted.id_mobil
   INTO pc_spr_mobil_journal (
		 id_SM
		,id_parent_mobil)	
FROM pc_spr_mobil AS SM
WHERE id  = @id_SM
```

### Вставить данные во временную или новую не созданную таблицу
```SQL
SELECT поля таблицы
INTO #имя таблицы, либо временная, либо создастся автоматически
FROM Имя таблицы откуда брать данные
```

### Вход из процедуры и из цикла
```SQL
RETURN -- из процедуры
BREAK -- из цикла WHILE
```


# CASE, IIF, ISNUUL, COALESCE  <a id="TRIGGER"></a>
```SQL
--Когда проблемы с кодировкой, придется добавлять коллейт. THEN может быть множество.
CASE   
   WHEN field = 0  
      THEN field2 collate Cyrillic_General_CI_AS
   ELSE field3  collate Cyrillic_General_CI_AS
END AS name_row

IIF(field = 0 , field2, field3)

ISNULL(field, 0)  -- Если NULL, то 0

COALESCE(field1, field2.., 'Value') -- возвращает первое значение, которое не равно NULL, если все NULL, то значение Value
```

a
# Математические функции: ABS, CEILING, FLOOR, SQRT, SQUARE, POWER, LOG <a id="math-func"></a>
- ABS - обсолютное (положительное) число
- CEILING - округление до целого в большую сторону
- FLOOR - округление до целого в меньшую сторону
- SQRT - кв. корень
- SQUARE - возведение в квадрат
- POWER - возведение в степень (второй параметр)
- LOG - натуральный логарифм

# Объединение UNION, INTERSECT и EXCEPT <a id="UNION"></a>
- UNION - объединяет две таблицы с одинаковыми столбцами, берутся только уникальные значения (работает медленнее)
- UNION ALL - объединяет две таблицы с одинаковыми столбцами, берутся все значения из таблиц и просто соединяются (работает быстрее, если не устанавливать сортировку, то порядок будет такой же, как и инструкции в запросе)
- INTERSECT - объединяет таблицы, выводит только те строки, которые ЕСТЬ и в верхнем и в нижнем запросах
- EXCEPT - объединяет таблицы, выводит только те строки, которых НЕТ в нижнем запросе


# ПредставлениеVIEW <a id="VIEW"></a>
Объект БД, который хранит в себе инсрукцию SELECT. Можно просто просматривать данные. Но, можно еще производить изменения, если столбец в представлении указан напрямую из рузультирующей таблицы, без агрегатных функций и тд. 
```SQL
CREATE VIEW count_pc
AS
	SELECT C.category_name, 
		(
			SELECT COUNT(*)
			FROM goods
			WHERE category = C.category_id) AS cnt_goods

	FROM categories C
-- Теперь, при запросе SELECT * FROM count_pc, мы будем получать кол-во пк в каждой категории

-- Для изменения предстваления можно использовать следующую коснтрукцию с ALTER VIEW
ALTER VIEW count_pc
AS
	SELECT 	C.category_id
		   	,C.category_name
			,(
				SELECT COUNT(*)
				FROM goods
				WHERE category = C.category_id) AS cnt_goods

	FROM categories C
```
# Индексы <a id="INDEX"></a>
 - Индекс - это объект БД, 
 - Типы индексов
	- кластеризованный (Clustered) - по возможности, каждыя таблица должна иметь этот индекс. Обычно применяется к ключам.
	- некластеризованный (nonclustered)  - значение ключа и указатель на строку данных содержащую значеине ключа. Их может быть несколько.
	- фильтруемый (Filtered) - это оптимизированный некластеризованный индекс, который использует предикат фильтра для индексирования части строк в таблице.
	- уникальный (Unique) - обеспечивает отсутствие повторяющихся ключей индекса. Могут быть класт. и некласт. Можно сделать только для уникальных значений столбца(либо столбцов).
	- колоночный (Columnstore) - оснван на технологии хранения индекса в виде стобцов. Эффективено использовать для больших хранилищ данных. Данные там сжимаются. 
	- полнотекстовый (Full-text) - обеспечивае быстрый поиск слов в символьных, строковых данных.
	- пространственный (Spatian) - специальный индекс, обеспечивает возомжность более продуктивного индекса с типами geometry или geography.
	- XML - специальный индекс, который создан для стобцов XML - типа.
	- 

```SQL
--Создание
CREATE NONCLUSTERED INDEX IX_non_clustered ON goods
(
	category ASC
)

-- Удаление индекса
DROP INDEX IX_non_clustered ON goods

-- Добавление индекса
CREATE NONCLUSTERED INDEX IX_non_clustered ON goods
(
	category ASC
	,product_name ASC
)
	-- добавим не ключевые поля
	INCLUDE (price)

-- При добавлении новых индексов, можно не удалять предыдущие индексы, а в конце прописать:
WITH (DROP_EXISTING = ON) -- удаляет индекс перед созданием, если такой существует
--

```

# Проектирование индексов <a id="TRIGGER"></a>
- Одним из самых эффективных индексов является индекс для целочисленных столбцов,
которые имеют уникальные значения, поэтому по возможности создавайте индексы для
таких столбцов;
- Если таблица очень интенсивно обновляется, то не рекомендуется создавать большое
количество индексов, так как это снижает производительность инструкций INSERT, UPDATE,
DELETE и MERGE. Потому что после изменений данных в таблице, SQL сервер автоматически
вносит соответствующие изменения во все индексы;
- Если таблица с большим объемом данных обновляется редко, при этом она активно
используется в инструкциях SELECT, т.е. на выборку данных, то большое количество индексов
может улучшить производительность, так как у оптимизатора запросов будет больший выбор
индексов при определении наиболее эффективного и быстрого способа доступа к данным;
- Для таблиц с небольшим объемом данных создание некластеризованных индексов с целью
повышения производительности может оказаться абсолютно бесполезно, да еще и с
затратами на их поддержание. Так как оптимизатору может потребоваться больше времени
на поиск данных в индексе, чем просмотр данных в самой таблице. Поэтому не создавайте
индексы для таблиц, в которых очень мало данных;
- Кластеризованный индекс необходимо создавать для столбца, который является
уникальным и не принимает значения NULL, также длина ключа должна быть небольшой,
другими словами, ключ индекса не нужно составлять из нескольких столбцов;
- Некластеризованные индексы лучше всего создавать для всех столбцов, которые часто
используются в условиях (WHERE) и в объединениях (JOIN);
- По возможности не стоит создавать индексы, в которых очень много ключевых столбцов, так как это влияет на размер индекса и на ресурсы его поддержания;
- Эффективно использовать покрывающие индексы, т.е. индексы, которые включают все
столбцы, используемые в запросе. Благодаря этому оптимизатор запросов может найти все
значения столбцов в индексе, при этом не обращаясь к данным таблиц, что приводит к
меньшему числу дисковых операций ввода-вывода. Этого можно достичь с помощью
включения в индекс неключевых столбцов (включенные столбцы), но также следует принять
во внимание, что это влечет за собой увеличение размера индекса;
- Если есть возможность, то рекомендовано заменять неуникальный индекс уникальным для
той же комбинации столбцов, это обеспечивает оптимизатору запросов дополнительные
сведения, что может сделать индекс более эффективным;
- При создании индекса учитывайте порядок ключевых столбцов, это повышает
производительность индекса. Например, столбцы, которые используются в предложении
WHERE в условиях поиска равно (=), больше (>), меньше (<) или находящихся в интервале
(BETWEEN) или участвуют в соединении (JOIN), должны стоять первыми. Если таких
несколько, то упорядочивайте их по уровню различности, т.е. от наиболее четкого к наименее
четкому

# Ограничения столбцов <a id="TRIGGER"></a>
- NOT NULL - устанавливаем ограничение на добавление NULL
- PRIMARY KEY - устанавливает первичный ключ. Можно указать двумя способами (ниже в коде пример)
- FOREIGN KEY - внешний ключ для связи с данными из другой таблицы. Используется для целостности данных
- UNIQUE - устанавливает запрет, на запись одинаковых записей
- CHECK - это проверочное ограничение, которое обеспечивает выполнение определенных условий
- DEFAULT - значение по умолчанию
```SQL
ALTER TABLE table1 ALTER COLUMN field1 INT NOT NULL; -- Устанавливаем ограничение на добавление NULL

PRIMARY KEY
```

# SEQUENCE последовательность <a id="SEQUENCE"></a>
Создает объект БД, в котором можно указывать последовательность, а далее, ее использовать в запросах
```SQL

```

# Пакет <a id="ИМЯ"></a>
 Это команды или ннструкции языка, которые передаются на сервер как одно целое, он не будет их анализировать и компилировать как единую инструкцию. Пакеты разделяются оператором GO, т.е. мы описали переменную, ниже написали GO, пакет закрылся и мы больше не сможем использовать эту переменную ниже. CREATE VIEW - доложна быть первой инструкцией в пакете
```SQL

```

# PIVOT, UNPIVOT <a id="PIVOT"></a>
Оператор языка "переворачивет" таблицу, преобразуя значение столбца в стобцы.
```SQL
-- Исходные данные
SELECT C.category_name, AVG(G.price) AS avg_price
FROM goods G
	LEFT JOIN categories C ON G.category_id = C.category_id
	GROUP BY C.category_name
```
Вывод

| category_name | avg_price  |
|:------------- |:----------:|
| TV     	    | 20000 	 | 
| mobile        | 10000      | 


## PIVOT запрос

```SQL
	-- Значения в столбце category_name
	SELECT 'Средняя цена' AS avg_price, TV, mobile
	-- Запрос с исходным данными source_tbl
	FROM (	SELECT C.category_name, AVG(G.price) AS avg_price
			FROM goods G
				LEFT JOIN categories C ON G.category_id = C.category_id ) AS source_tbl
	-- Вызываем PIVOT, передаем price в функцию AVG. FOR - указываем столбец из которого берем имена столбцов. IN - перечисляем значение столбцов из category_name. 
	PIVOT (AVG(price) FOR category_name IN (TV, mobile)
	) AS pivot_table  -- Псевдоним обязателен
```
| avg_price    | TV  	 | mobile |
|:-------------|:-------:|:------:|
| Средняя цена | 20000   | 10000  |





## UNPIVOT запрос редко используется
### исходные данные
| type_equp | column1 | column2 | column3 |
|:----------|:-------:|:-------:|:-------:|
| TV        | Samsung | LG      | AOC     |

```SQL
	-- Наименования столбцов
	SELECT type_equp, column_name, name_tv
	-- Запрос с исходным данными source_tbl
	FROM table_name
	-- Вызываем PIVOT, передаем price в функцию AVG. FOR - указываем столбец из которого берем имена столбцов. IN - перечисляем значение столбцов из category_name. 
	UNPIVOT (name_tv FOR column_name IN (column1, column2, column3)
	) AS unpivot_table  -- Псевдоним обязателен
```

### Вывод
| type_equp    | column_name | name_tv |
|:-------------|:-----------:|:-------:|
| TV   		   | column1     | Samsung |
| TV   		   | column2     | LG 	   |
| TV   		   | column3     | AOC 	   |

# Аналитические операторы: ROLLUP, CUBE и GROUPING SETS  
 <a id="ИМЯ"></a>
ROLLUP - формирует промежуточные итоги для указаных стобцов и общий итог
```SQL
SELECT category, SUM(price) AS sum_price
FROM goods
GROUP BY 	
ROLLUP (category)
```

CUBE - формирует результаты для всех возможных перекрестных вычислений
```SQL
SELECT product_name, category, SUM(price) AS sum_price
FROM goods
GROUP BY 	
ROLLUP (category, product_name)
```

GROUPING SETS  - формирует результаты нескольких группировок в один набор данных. Эксвиввалент UNION ALL
```SQL
SELECT product_name, category, SUM(price) AS sum_price
FROM goods
GROUP BY 	
GROUPING SETS  (category, product_name)
```



# СЛИЯНИЕ ДАНЫХ MERGE <a id="MERGE"></a>
MERGE - выполняет операции добавления, удаления, обновления для таблицы на основе результатов соединения с другой таблицей.

tbl
| equpment_id  | equpment_name | price |
|:-------------|:-----------:|:-------:|
| 1   		   | PC	         | 0 	   |
| 2   		   | print       | 0 	   |
| 4   		   | monitor	 | 0 	   |

tbl_temp
| equpment_id  | equpment_name | price |
|:-------------|:-----------:|:-------:|
| 1   		   | PC	         | 500 	   |
| 2   		   | print       | 300	   |
| 3   		   | monitor	 | 400 	   |

```SQL
MERGE tbl AS t_base 
	USING ( SELECT equpment_id, equpment_name, price
			FROM tbl_temp) AS t_source (equpment_id, equpment_name, price)
	ON (t_base.equpment_id = t_source.equpment_id) -- условие объединения

	WHEN MATCHED THEN -- если совпадение, то обновляем строки
		UPDATE SET equpment_name = t_source.equpment_name
				   ,price = t_source.price
	WHEN NOT MATCHED THEN -- если не истина, то делаем вствку
		INSERT (equpment_id, equpment_name, price) 
		VALUES (t_source.equpment_id, t_source.equpment_name, t_source.price)
	
	WHEN NOT MATCHED BY SOURCE THEN -- Если нет соападений с истоником, то удаляем строки
		DELETE
	-- можно вывести, то, что сделалось при слиянии
	OUTPUT $action 				AS operation
		,inserted.equpment_id	AS equpment_id
		,inserted.equpment_name AS equpment_name_new
		,inserted.price 		AS price_new
		,deleted.equpment_name 	AS equpment_name_old
		,deleted.price  		AS price_old; -- ; обязательна
```
OUTPUT
| operation | equpment_id | equpment_name_new | price_new | equpment_name_old | price_old |
|:----------|:-----------:|:-----------------:|:---------:|:-----------------:|:---------:|
| UPDATE    | 1	          | PC 	   			  | 500		  | PC 				  | 0         |
| UPDATE    | 2	          | print  			  | 300		  | print			  | 0         |
| INSERT    | 3	          | monitor			  | 400		  | NULL 			  | NULL      |
| DELETE    | NULL        | NULL   			  | NULL	  | monitor			  | 0         |

tbl
| equpment_id  | equpment_name | price |
|:-------------|:-----------:|:-------:|
| 1   		   | PC	         | 500 	   |
| 2   		   | print       | 300	   |
| 3   		   | monitor	 | 400 	   |

tbl_temp
| equpment_id  | equpment_name | price |
|:-------------|:-----------:|:-------:|
| 1   		   | PC	         | 500 	   |
| 2   		   | print       | 300	   |
| 3   		   | monitor	 | 400 	   |



# OUTPUT <a id="ИМЯ"></a>
OUTPUT - позволяет получить изменившиеся строки после INSERT, UPDATE, DELETE, MERGE. 
```SQL 
```

# ROW_COUNT <a id="ИМЯ"></a>
ROW_COUNT - системная функция, которая возвращает кол-во затронутых строк при выполнении последней инструкции.
### Важно:
1. необходимо использовать сразу после инстркции, результат который, вы хотите узнать
2.  после MERGE, фунеция выведет сумму вставленных, обновленных и удаленных строк 
```SQL 
UPDATE tbl SET price = price * 1.1
WHERE category = 2

SELECT @@ROWCOUNT AS count_upd_rows
```

# Пользовательские функции <a id="FUNCTION"></a>
Есть 2 типа функций, которые можно создать
1. Скалярные - возвращают одно значение
```SQL 
CREATE FUNCTION test_func
(
	@product_id INT -- Объявляем входящие параметры
)
RETURNS VARCHAR(100) -- Тип возвр. результата
AS
BEGIN
	DECLARE @product_name VARCHAR(100);

	--Получим наименование товара по его идентификатору
	SELECT @product_name = product_name
	FROM goods
	WHERE product_id = product_id

	--Возвращаем результат
	RETURN product_name

END

GO

--Вызов функции
SELECT dbo.test_func(1) AS name_product
```
2. табличные - возвращают данные в виде таблицы
```SQL 
CREATE FUNCTION prod_in_category
(
	@category_id INT -- Объявляем входящие параметры
)
RETURNS TABLE-- Тип возвр. результата
AS
RETURN
(
	--Получим наименование товара по его идентификатору
	SELECT 	product_id
			,product_name
			,price
			,category
	FROM goods
	WHERE category = @category_id

	--Возвращаем результат
	RETURN product_name

)	

GO

--Вызов функции
SELECT * FROM dbo.name_product(1) 
```
3. табличная функция, в которой можно использовать условия и др. операторы. Для создания такой функции необходимо указать таблицу в возвращающемся значении
```SQL 
CREATE FUNCTION prod_in_category2
(
	@category_id INT -- Объявляем входящие параметры
	,@price MONEY
)
RETURNS @returns_tbl TABLE ( product_id INT
							,product_name VARCHAR(100)
							,price MONEY
							,category INT )
AS
RETURN
(
	--Получим наименование товара по его идентификатору
	SELECT 	product_id
			,product_name
			,price
			,category
	FROM goods
	WHERE category = @category_id

	--Возвращаем результат
	RETURN product_name

)	

GO

--Вызов функции
SELECT * FROM dbo.name_product(1) 
```


# APPLY <a id="APPLY"></a>
APPLY - оператор, который позволяет запускать табличную функцию для каждой строки источника данных и на выходе получать все результирующие наборы, объединенные в одном. Есть 2 вида: CROSS и OUTER
```SQL 
--Смысл примерно такой же, как с жойнами, только тут мы передаем параметром данные из таблицы category. Будет выведены строки, которые есть и в той и втой таблице
SELECT C.category, G.*
FROM category C
	CROSS APPLY dbo.product_in_category(c.category_id) AS G

--Будет выведены все строки, если в таблице category, будет категории, которых нет в функции, то будет выведено имя категории, а данные будут NULL
SELECT C.category, G.*
FROM category C
	OUTER APPLY dbo.product_in_category(c.category_id) AS G


```

# Псевдонимы типа данных <a id="ИМЯ"></a>
```SQL 
--Создает псевдоним строки с разменостью 150
CREATE TYPE test_type FROM VARCHAR(150) NULL;

--Создает табличный тип
CREATE TYPE test_tbl_type AS TABLE
(
	product_name VARCHAR(100) NULL
	,price MONEY NULL
)

-- Псевдонимы можно использовать при передаче парамтеров в процедуру. Если это таблица, то надо указать READONLY
CREATE PROCEDURE test_procedure @text_comment test_type
								,@tmp_table test_tbl_type READONLY
AS
BEGIN
	SELECT TMP.*, @txt_comment AS txt_comment
	FROM @tmp_table tmp
END

-- Другой пакет
GO

DECLARE 
	@text_comment test_type
	,@test_table test_tbl_type

SET @text_comment = 'Заказ номер 10'

INSERT INTO @test_table(product_name, price)
VALUES 	 ('print', 5000)
		,('monitor', 8000)

EXEC test_procedure @text_comment, ,@test_table
```

# ИМЯ <a id="ИМЯ"></a>
```SQL 
```










