## Задания для собеседования на позицию аналитика

## Описание БД для заданий по sql

Есть менеджеры, отвечающие на звонки клиентов. В рамках звонка менеджер должен не только ответить на вопросы, но и сделать какое-то предложение - upsale.
Набор предложений (factor), которые можно сделать, стандартный и фиксированный. Для простоты будем считать, что менеджер делает предложение при каждом обращении и при этом только одно (из некоторого списка).
Для оценки эффективности этой деятельности, факты контактов с клиентами и результативность этих контактов в части сделанных предложений заносятся в базу.
Конверcия это отношение количества принятых предложений к общему количеству предложений.

##### Таблицы
managers: список менеджеров

![Image alt](https://github.com/plakidinsv/assignment_xxx/blob/main/managers.jpg)

upsale: факты контакта с клиентом

![Image alt](https://github.com/plakidinsv/assignment_xxx/blob/main/upsale.jpg)

## Блок заданий на sql
 1. Отобразить сколько предложений сделал каждый из менеджеров за всю историю

```
select 	distinct "NAME",
	count("CALL_ID") over(partition by "Manager_ID") AS total_calls
from upsale AS u
left join managers m
on u."Manager_ID" = m."ID"
order by 1;
```

![Image alt](https://github.com/plakidinsv/assignment_xxx/blob/main/1.jpg) 

2.  Отобразить конверсию предложений в подключения для менеджеров за указанный период времени.
```
with cte as (
select 	distinct "Manager_ID",
			count("CALL_ID") over(partition by "Manager_ID") AS total_calls,
			count("RESULT") filter(where "RESULT" = true) over(partition by "Manager_ID") as res
from upsale AS u
order by 1
)
select "NAME",
		res * 100 / total_calls as manager_cr
from cte
left join managers m
on cte."Manager_ID" = m."ID";
```
![Image alt](https://github.com/plakidinsv/assignment_xxx/blob/main/2.jpg) 

3.  Вывести конверсии тех менеджеров, у которых было больше 100 звонков за указанный период времени.

```
with cte as (
select 	distinct "Manager_ID",
			count("CALL_ID") over(partition by "Manager_ID") AS total_calls,
			count("RESULT") filter(where "RESULT" = true) over(partition by "Manager_ID") as res
from upsale AS u
order by 1
)
select "Manager_ID",
		res * 100 / total_calls as manager_cr
from cte
left join managers m
on cte."Manager_ID" = m."ID"
where total_calls > 100; 
```
**В данном случае запрос не выведет записей, поскольку тестовая база данных содержит всего 12 строк**

4.  Вывести офисы, отсортированные в порядке убывания средней конверсии менеджеров из офиса за всю историю.

```
with cte as (
select 	distinct "Manager_ID",
			count("CALL_ID") over(partition by "Manager_ID") AS total_calls,
			count("RESULT") filter(where "RESULT" = true) over(partition by "Manager_ID") as res
from upsale AS u
order by 1
),
cte_2 as (
select "Manager_ID",
		res * 100 / total_calls as manager_cr
from cte
)
select distinct "OFFICE",
		avg(manager_cr) over(partition by "OFFICE") as avg_cr
from cte_2
left join managers m
on cte_2."Manager_ID" = m."ID"
order by 2 desc;
```
![Image alt](https://github.com/plakidinsv/assignment_xxx/blob/main/4.jpg) 

5.  Вывести минимальный порядковый номер звонка клиента с RESULT = Yes, перед которым был RESULT = No для каждого менеджера.

```
with cte as (
select "Manager_ID" ,
		"DT" ,
		"RESULT" ,
		"CALL_ID" ,
		row_number() over (partition by "Manager_ID" order by "DT", "CALL_ID") as call_rank,
		lag("RESULT") over(partition by "Manager_ID")
from upsale u
)
select m."NAME" ,
		"CALL_ID",
		first_value(call_rank) over (partition by cte."Manager_ID" order by call_rank)
from cte 
left join managers m
on cte."Manager_ID" = m."ID"
where "RESULT" = true and lag = false;
```
![Image alt](https://github.com/plakidinsv/assignment_xxx/blob/main/5.jpg) 

6. Получить для каждого менеджера первое принятое предложение.

```
select distinct m."NAME",
		first_value("FACTOR_ID") over(partition by "Manager_ID" order by "DT") as first_accepted_factor_id
from upsale u
left join managers m 
on u."Manager_ID" = m."ID"
where "RESULT" = true
order by 1;
```
![Image alt](https://github.com/plakidinsv/assignment_xxx/blob/main/6.jpg) 

7. Предложить запрос, показывающий ранее не использованные операторы SQL:

Получить для каждого менеджера первое принятое предложение и дату такого предложения в формате:
"Наименование дня недели, название месяца, число месяца, полное наименование месяц, год"

```
select distinct m."NAME",
		first_value("FACTOR_ID") over(partition by "Manager_ID" order by "DT") as first_accepted_factor_id,
		to_char("DT", 'FMDay, FMDD, FMMonth, YYYY') as first_accepted_factor_date
from upsale u
left join managers m 
on u."Manager_ID" = m."ID"
where "RESULT" = true
order by 1;
```
![Image alt](https://github.com/plakidinsv/assignment_xxx/blob/main/7.jpg) 

8.  Предложить запрос, который покажет интересную или полезную информацию из этих данных:

Определить топ-3 клиентов, чаще всего принимающих предложение менеджеров

```
select distinct "CLIENT_ID",
		count("RESULT") over(partition by "CLIENT_ID") 
from upsale u 
where "RESULT" = true
limit 3;
```
![Image alt](https://github.com/plakidinsv/assignment_xxx/blob/main/8.jpg) 
