# Проекты по анализу данных
В данном репозитории хранится портфолио моих проектов.

Проекты в основном представлены в виде файлов, подготовленных в Jupyter Notebook на языке Python, а также в виде дашбордов на платформе Yandex DataLens.

№|Название проекта|Описание|Инструменты и библиотеки
-|----------------|--------|---------------------------------------
1|[Исследование аудитории фанатов киберспорта в рамках стажировки в ТГУ по специальности "Аналитик данных"](https://github.com/Lenupcik/portfolio/blob/main/Cyber.ipynb) |Составила портрет пользователя, интересующегося киберспортом и состоящего в сообществе по киберспорту|pandas, matplotlib, seaborn
2|Дашборд с портретом фаната киберспорта в рамках стажировки в ТГУ по специальности "Аналитик данных"|Создала дашборд на Yandex.DataLens ["Cоциально-демографический портрет российского киберспортсмена"](https://datalens.yandex/daflqs6wae7i5)|datalens.yandex
3|[Конкурс по визуализации данных ИНИД на Yandex DataLens](https://diagram-contest.ru/)| Создала [дашборд](https://datalens.yandex/1wwanbydjzsmt) по российским ВУЗам для нескольких категорий пользователей   (администрация губернаторов — выбор ВУЗа для включения в перечень на финансирование,   ректоры ВУЗов — определение точек роста,   абитуриенты— выбор ВУЗа для поступления)|datalens.yandex
4|[Построение моделей машинного обучения "Предсказание выживет или нет пассажир Титаника" в рамках стажировки в ТГУ по специальности "Аналитик данных"](https://github.com/Lenupcik/portfolio/blob/main/ML.ipynb) |Построила модели Logistic Regression, Decision Tree, Random Forest, Gradient Boosting, MLPClassifier|pandas, sklearn, numpy, matplotlib
5|[Обработка естественного языка "Тематическое разделение групп ВК" в рамках стажировки в ТГУ по специальности "Аналитик данных"](https://github.com/Lenupcik/portfolio/blob/main/NLP.ipynb) |Разбила датасет с описанием групп ВК на тематические области двумя способами(DecisionTreeClassifier и Latent Dirichlet Allocation)|pandas, numpy, matplotlib, nltk, sklearn, warnings, multiprocessing, pymorphy2

# Примеры запросов к демонстрационной базе данных для СУБД PostgreSQL “Авиаперевозки”
## Подготовка:
1. [Описание демонстрационной базы](https://postgrespro.ru/education/demodb)
2. Скачиваем  демонстрационную базу и импортируем в PostgreSQL через командную строку windows
+ Переходим в каталог, где находится скачанная БД  
cd >cd C:\Users\Лена\Desktop\БД
+ Далее выполняем команду для загрузки БД из sql-файла<br/>
"C:\Program Files\PostgreSQL\13\bin\psql" -U postgres -f demo_small.sql

## Задание:
Нужно написать запрос, который покажет, по каким дням недели, в какие города и какое количество должно вылететь рейсов из города Москва с “2017-08-15 15:00:00” по “2017-08-25 15:00:00” для пассажиров, купивших билеты в эконом классе на самолет “Аэробус А321-200”, но не прошедших регистрацию.

## Входные данные
+ Самолет Аэробус А321-200
+ Билет эконом класса
+ Город вылета Москва
+ Дата вылета с “2016-10-13 19:00:00” по “2016-10-23 19:00:00”  (включая начальное и конечное время)
+ Регистрация не пройдена

## Выходные данные
+ Номер строки (отдельная колонка). Название колонки rn
+ Список номеров дней недели. Название колонки day_of_week
+ Название города прилета. Название колонки arr_city
+ Количество рейсов вылета. Название колонки cnt

## Требования
+ В запросе должна использоваться таблица cities
+ Запрос должен быть “обернут” либо в CTE, либо во временную таблицу
+ Нужно использовать условие where и оператор ->>
+ В выборке данных для json-а нужно использовать функцию lang

## Выполнение:
### 1. создаем таблицу cities<br/>
create table cities(<br/>
 city_id serial not null,<br/>
 name jsonb not null<br/>
);

alter table cities<br/>
add constraint pk__id primary key (city_id);

alter table airports_data add column city_id int;

alter table airports_data<br/>
add constraint fk__airports__city_id<br/>
foreign key (city_id) references cities (city_id);

insert into cities (name)<br/>
select distinct city from airports_data;

update airports_data ad<br/>
set city_id = c.city_id<br/>
from cities c<br/>
where ad.city = c.name;
![Схема БД после создания таблицы cyties](https://github.com/Lenupcik/portfolio/blob/main/new_chart.jpg)
![Скрин создания таблицы cyties]()
### 2. Выполняем сам запрос, согласно задания

create temp table mosсow_flights as<br/>
select to_char(scheduled_departure, 'ID'::text)::int as day_of_week, c_2.name as city_name<br/>
from flights f<br/>
join aircrafts_data ac on ac.aircraft_code=f.aircraft_code

join airports_data ap_d on ap_d.airport_code=f.departure_airport<br/>
join cities c_1 on c_1.city_id=ap_d.city_id<br/>
join airports_data ap_a on ap_a.airport_code=f.arrival_airport<br/>
join cities c_2 on c_2.city_id=ap_a.city_id
	
join ticket_flights tf on tf.flight_id=f.flight_id<br/>
left join boarding_passes bp on tf.ticket_no=bp.ticket_no and tf.flight_id=bp.flight_id
	
where ac.model ->>lang() = 'Аэробус A321-200' and c_1.name ->>lang() = 'Москва'<br/> 
and bp.ticket_no is NULL and bp.flight_id is NULL<br/>
and fare_conditions = 'Economy'<br/>
and scheduled_departure between bookings.now() and bookings.now()+interval '10 day'<br/>
group by scheduled_departure, c_2.name

select row_number () over () as rn, city_name ->>lang() as arr_city, count(*) as cnt, array_agg(distinct day_of_week) as day_of_week<br/> 
from mosсow_flights<br/>
group by city_name
![Скрин запроса к БД по заданию](https://github.com/Lenupcik/portfolio/blob/main/temp_table.png)




