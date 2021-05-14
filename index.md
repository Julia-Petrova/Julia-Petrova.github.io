---
title: Документация
nav_order: 2
---

DTM : SELECT  

1.  [DTM](index.html)

DTM : SELECT
============

Created by Vladimir Vinnikov, last modified by Yulia Petrova on апр. 20, 2021

DRAFT
{: .label .label-blue }

OSS
{: .label .label-green }
EE
{: .label .label-red }

Запрос позволяет выбрать данные из [логических таблиц](https://arenadata.atlassian.net/wiki/spaces/DTM/pages/354945309) и [представлений](https://arenadata.atlassian.net/wiki/spaces/DTM/pages/361070885). Запрос можно использовать как самостоятельный запрос для [чтения данных](https://arenadata.atlassian.net/wiki/spaces/DTM/pages/356321401), а также в качестве подзапроса в [запросах на выгрузку данных](https://arenadata.atlassian.net/wiki/spaces/DTM/pages/557089214/INSERT+INTO+download_external_table) и запросах на [создание](https://arenadata.atlassian.net/wiki/spaces/DTM/pages/544965567/CREATE+VIEW) и [обновление](https://arenadata.atlassian.net/wiki/spaces/DTM/pages/545292823/ALTER+VIEW) логических представлений.

Для выбора доступны следующие данные:

*   актуальные данные;
    
*   архивные данные, которые были актуальны в указанный момент времени;
    
*   изменения данных, выполненные в рамках указанного диапазона [дельт](https://arenadata.atlassian.net/wiki/spaces/DTM/pages/354946089).
    

Запрос обрабатывается в порядке, описанном в разделе [Порядок обработки запросов на чтение данных](https://arenadata.atlassian.net/wiki/spaces/DTM/pages/559383099).

В ответе возвращается:

*   объект ResultSet c выбранными записями при успешном выполнении запроса;
    
*   исключение при неуспешном выполнении запроса.
    

#### Синтаксис
``` sql
SELECT column_list 
FROM [db_name.]entity_name 
[FOR SYSTEM_TIME time_expression [AS alias_name]]
[DATASOURCE_TYPE = 'datasource_alias']
```
Описание параметров запроса см. [ниже](#select_parameters).

В запросе можно использовать следующие секции, которые должны быть указаны в порядке их перечисления:

*   `FOR SYSTEM_TIME` — для указания момента времени или периода, за который выбираются данные или изменения данных. Если секция не указана, данные выбираются по состоянию на дату и время обработки запроса;
    
*   `JOIN ON` — для соединения данных нескольких логических таблиц или представлений;
    
*   `WHERE` — для указания условий выбора данных;
    
*   `GROUP BY` — для группировки данных;
    
*   `HAVING` — для указания условий выбора сгруппированных данных;
    
*   `ORDER BY` — для сортировки данных;
    
*   `LIMIT` — для ограничения количества возвращаемых строк;
    
*   `DATASOURCE_TYPE` — для указания [СУБД](https://arenadata.atlassian.net/wiki/spaces/DTM/pages/354944467) [хранилища](https://arenadata.atlassian.net/wiki/spaces/DTM/pages/361071530), из которой выбираются данные.
    

Для имен таблиц, представлений и столбцов можно использовать псевдонимы. В запросе доступны оператор `DISTINCT`, агрегатные функции, а также соединение нескольких логических таблиц или логических представлений из одной или нескольких [логических БД](https://arenadata.atlassian.net/wiki/spaces/DTM/pages/354945300).

Поддерживаются следующие типы соединений:

*   `[NATURAL]` — соединения по всем столбцам с одинаковыми именами,
    
*   `[INNER]` — внутреннее соединение,
    
*   `LEFT [OUTER]` — левое внешнее соединение,
    
*   `RIGHT [OUTER]` — правое внешнее соединение,
    
*   `FULL [OUTER]` — полное внешнее соединение,
    
*   `CROSS` — декартово произведение таблиц или представлений, ключи соединения не указываются.
    

**Примечание:** некоторые агрегатные функции и типы соединений недоступны для исполнения в определенных СУБД хранилища. Список доступных возможностей см. в разделе [Поддержка SQL](https://arenadata.atlassian.net/wiki/spaces/DTM/pages/354944477).

#### Параметры

*   `column_list` — список выбираемых столбцов таблицы или представления. Допустимо указывать символ `*` для выбора всех столбцов;
    
*   `db_name` — имя логической базы данных, из которой выбираются данные. Указывается опционально, если выбрана логическая БД, [используемая по умолчанию](https://arenadata.atlassian.net/wiki/spaces/DTM/pages/401279070);
    
*   `entity_name` — имя таблицы или представления, из которого выбираются данные;
    
*   `time_expression` — выражение, которое задает момент или период времени, за который выбираются данные или изменения данных. Формат выражения см. [ниже](#select_for_system_time);
    
*   `alias_name` — псевдоним таблицы или представления. Может включать латинские буквы, цифры и символы подчеркивания;
    
*   `datasource_alias` — системный псевдоним СУБД хранилища, из которой выбираются данные. Возможные значения: adb, adqm, adg.
    

#### Синтаксис директивы FOR SYSTEM\TIME

Директива `FOR SYSTEM_TIME` позволяет указать момент, по состоянию на который запрашиваются данные, или период (диапазон дельт), за который запрашиваются изменения. Директива относится к логической таблице или представлению, после имени которого она следует. Если в запросе соединяется несколько логических таблиц и представлений, для каждой логической сущности можно указать свою директиву, при этом значения директив могут различаться (см. пример [ниже](#EX_select_with_different_system_times)).

Директива указывается в формате `FOR SYSTEM_TIME time_expression`, где выражение `time_expression` принимает одно из следующих значений:

*   `AS OF 'YYYY-MM-DD HH:MM:SS'` — для запроса данных, актуальных на указанную дату и время;
    
*   `AS OF DELTA_NUM delta_num` — для запроса данных, актуальных на дату и время закрытия дельты с номером `delta_num`;
    
*   `AS OF LATEST_UNCOMMITTED_DELTA` — для запроса данных на текущий момент, включая данные, загруженные в рамках открытой (горячей) дельты. По горячей дельте возвращаются записи, загруженные в рамках непрерывного диапазона завершенных операций записи (см. параметры `cn_from` и `cn_to` в разделе [https://arenadata.atlassian.net/wiki/pages/resumedraft.action?draftId=551192787](https://arenadata.atlassian.net/wiki/pages/resumedraft.action?draftId=551192787));
    
*   `STARTED IN (delta_num1, delta_num2)` — для запроса данных, добавленных или измененных в период между дельтой `delta_num1` и дельтой `delta_num2` (включительно);
    
*   `FINISHED IN (delta_num1, delta_num2)` — для запроса данных, удаленных в период между дельтой `delta_num1` и дельтой `delta_num2` (включительно).
    

#### Ограничения

*   Не допускается комбинирование подзапросов к логическим базам данных с подзапросами к системным представлениям `INFORMATION_SCHEMA`.
    
*   Если ключами секции `JOIN` выступают поля типа Nullable, то строки, где хотя бы один из ключей имеет значение NULL, не соединяются.
    

#### Примеры

Запрос с неявным указанием столбцов и секцией `WHERE`:
``` sql
SELECT * FROM sales.sales
WHERE store_id = 1234
```
Запрос с явным указанием столбцов и выбором данных из определенной СУБД хранилища (ADQM):
``` sql
SELECT sold.store_id, sold.product_amount 
FROM sales.stores_by_sold_products AS sold
DATASOURCE_TYPE = 'adqm'
```
Запрос с агрегацией, группировкой и сортировкой данных, а также выбором первых 20 строк:
``` sql
SELECT s.store_id, SUM(s.product_units) AS product_amount 
FROM sales.sales AS s
GROUP BY (s.store_id)
ORDER BY product_amount DESC
LIMIT 20
```
Запрос записей, актуальных на момент закрытия дельты с номером 9:
``` sql
SELECT * FROM sales.sales FOR SYSTEM_TIME AS OF DELTA_NUM 9
```
Запрос с соединением данных двух логических таблиц из двух различных логических БД:
``` sql
SELECT 
  st.identification_number, 
  st.category, 
  s.product_code 
FROM sales.stores FOR SYSTEM_TIME AS OF LATEST_UNCOMMITTED_DELTA AS st
INNER JOIN sales2.sales FOR SYSTEM_TIME AS OF LATEST_UNCOMMITTED_DELTA AS s
  ON st.identification_number = s.store_id
```
Запрос с соединением записей логической таблицы, добавленных и измененных в двух различных диапазонах дельт:
``` sql
use sales

SELECT
  p1.product_code,
  p1.price as feb_price,
  p2.price as march_price,
  (p2.price - p1.price) as diff
FROM 
  (SELECT product_code, 
          price from sales.prices 
          FOR SYSTEM_TIME STARTED IN(3,6)) AS p1
FULL JOIN (select product_code, 
          price from sales.prices 
          FOR SYSTEM_TIME STARTED IN(7,10)) AS p2
  ON p1.product_code = p2.product_code
WHERE p1.product_code is NOT NULL
ORDER BY diff DESC
LIMIT 50
DATASOURCE_TYPE = 'adb'
```
Document generated by Confluence on апр. 23, 2021 17:11

[Atlassian](http://www.atlassian.com/)
