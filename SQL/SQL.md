# Содержание
[CASE](#CASE)  \
[CAST, DATEADD, CONVERT, преобразовать, конвертировать, добавить, убавить](#CAST,-DATEADD,-CONVERT,-преобразовать,-конвертировать,-добавить,-убавить) \
[Индексы](#Индексы)  \

## CASE
Когда проблемы с кодировкой, придется добавлять коллейт. THEN может быть множество.
```SQL
CASE   
   WHEN SSS.PlanPro = 0  
      THEN SUMine.nameUnit collate Cyrillic_General_CI_AS
   ELSE SU.nameUnit  collate Cyrillic_General_CI_AS
END AS nameUnit, 
```

## CAST, DATEADD, CONVERT, преобразовать, конвертировать, добавить, убавить

### Преобразовываем формат датаВремя в дату
```SQL
CAST(GETDATE() AS DATE)
```

### Отнимаем 4 часа
```SQL		
DATEADD(HOUR,-4, CAST(@DateBegin AS DATETIME))
```

### Конвертируем дату время в строку и ставим формат времени
```SQL		
CONVERT(NVARCHAR, @DateBegin, 21)
```

## Индексы
```SQL	
CREATE CLUSTERED INDEX clustered_date_eqmt_id ON dbo.table_name (date_time_begin, eqmt_id, id);
-- 
CREATE NONCLUSTERED INDEX IX_ ON Sales.table_name (SalesQuota, SalesYTD);
```
