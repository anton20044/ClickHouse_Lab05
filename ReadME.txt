Вариант домашней работы №2

1) Установил PostgreSQL, загрузил тестовый датасет. Создалась БД demo, схема bookings и таблицы с данными.

2) В качестве данных для словаря выбрал таблицу с названиями и условными обозначениями аэропортов (airports_data). На стороне Clickhouse словарь:
CREATE DICTIONARY airports_dt
(
	airport_code String,
	airport_name String
)
PRIMARY KEY airport_code
SOURCE(POSTGRESQL(HOST '192.168.50.111' PORT 5432 USER 'cl' PASSWORD 'cl' DB 'demo' SCHEMA 'bookings' TABLE 'airports_data' 
invalidate_query 'SELECT airport_code as airport_code, airport_name as airport_name FROM airports_data'
))

LIFETIME(300)
LAYOUT(complex_key_hashed())

Попробовал получить данные из словаря SELECT dictGet('airports_dt', 'airport_name', 'KHV') - все прошло успешно.

Из встретившихся "проблем":
- т.к. данные лежали не в схеме public необходимо было добавить параметр SCHEMA 
- при создании словаря пытался назвать его 'airports' или 'airports_data', получил ошибку. Название 'airports_dt' было принято Clickhouse. На стороне PostgreSQL существует таблица 'airports_data' и вьюха 'airports'. Отсюда делаю вывод что имена словарей в Clickhouse не могут пересекаться с именами таблиц/вьюх в PostgreSQL.

3) Выгрузил в Yandex Object Storage таблицу из Clickhouse (Учетные данные скрыл). (Скрин загруженного файла - Рисунок 1)

INSERT INTO FUNCTION s3('https://storage.yandexcloud.net/lab05/20240211.csv.gz', '***', '***', 
'CSVWithNames', 'actor_id UInt32, movie_id UInt32, role String, created_at DateTime', 'gzip') SELECT *
FROM roles

