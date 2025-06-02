# Intermediate
[Link](https://datalemur.com/sql-tutorial/intermediate-data-science-sql-intro)

## Aggregate Functions
Por ahora no se entra mucho en detalle, es su forma básica que viene acompañando al ``SELECT``.

| Function | Description                                                    |
|----------|----------------------------------------------------------------|
| `SUM`    | Adds together all the values in a particular column.           |
| `MIN`    | Returns the lowest value in a particular column.               |
| `MAX`    | Returns the highest value in a particular column.              |
| `AVG`    | Calculates the average of a group of selected values.          |
| `COUNT`  | Counts how many rows are in a particular column.               |

## GROUP BY
Se usa para organizar los datos en grupos según los valores de una o más columnas, y aplicar funciones como SUM, COUNT o AVG a cada grupo por separado.

> Todas las columnas que aparecen en el SELECT deben estar: en el GROUP BY, o dentro de una función de agregación.

Sintaxis basica
```sql
SELECT 
  column_to_group_by,
  aggregate_function(column)
FROM table_name
GROUP BY column_to_group_by;
```

Ejemplo
```sql
SELECT 
    ticker, 
    min(open) 
from stock_prices 
group by ticker 
order by min desc;
```

## HAVING
Se usa como un ``WHEN`` pero para filtrar después de aplicar el GROUP BY, normalmente el filtro se aplica a la función de agregación.

Sitaxis basica
```sql
SELECT columna, función_agregada
FROM tabla
GROUP BY columna
HAVING condición_sobre_la_función_agregada;
```

Ejemplo
```sql
SELECT ticker, min(open) 
FROM stock_prices
group by ticker
having min(open) > 100;
```

## DISTINCT
DISTINCT se usa junto con SELECT para devolver solo valores únicos, eliminando los duplicados en el resultado.

En este ejemplo, ``DISTINCT`` se usa dentro de ``COUNT()`` para contar los productos no repetidos. La función del código es mostrar las categorías agrupadas y cuántos productos únicos tiene cada una. Aunque acá se usa con ``COUNT()``, ``DISTINCT`` también puede usarse por separado en un SELECT para eliminar duplicados.

```sql
select category, count(DISTINCT product)
from product_spend 
group by category
```

## CASE
La sentencia CASE en SQL le permite dar forma, transformar, manipular y filtrar datos basándose en condiciones especificadas. Es una herramienta de expresión condicional que permite personalizar los resultados de las consultas, crear nuevas categorías y aplicar lógica condicional.

### CASE con SELECT
Se utiliza para crear nuevas columnas, categorizar datos o realizar cálculos basados en condiciones específicas. Ayuda a adaptar el resultado de la consulta para satisfacer requisitos específicos.

Sintaxis
```sql
SELECT
  column_1,
  column_2,
  CASE 
    WHEN condition_1 THEN 'result_1'
    WHEN condition_2 THEN 'result_2'
    -- Puedes agregar más condiciones si es necesario
    ELSE 'default_result' -- Si ninguna condición se cumple
  END AS new_column_name
FROM table_1;
```

En este **Ejemplo** crea la columna "popularity" y dependiendo del ``when`` los valores en las filas serian, Super likes, Good likes o Low likes.
```sql
select actor, character, platform, avg_likes,
CASE
  when avg_likes >= 15000 then 'Super Likes'
  when avg_likes between 5000 and 14999 then 'Good Likes'
  else 'Low Likes'
end as popularity
from marvel_avengers
order by avg_likes desc
```

### CASE con WHERE
Sintaxis
```sql
SELECT column_1, column_2
FROM table_1
WHERE 
  CASE 
    WHEN condition_1 THEN 1
    WHEN condition_2 THEN 1
    ELSE 0
  END = 1;
```

En este **Ejemplo** se filtra junto con AND y si se cumple la condición queda como verdadera y filtraria, por eso el 1(True), de lo contrario el Else al final indicando 0.
```sql
SELECT actor, character, platform, followers
FROM marvel_avengers
WHERE 
  CASE 
    WHEN platform = 'Instagram' AND followers >= 500000 THEN 1
    WHEN platform = 'Twitter' AND followers >= 200000 THEN 1
    WHEN platform NOT IN ('Instagram', 'Twitter') AND followers >= 100000 THEN 1
    ELSE 0
  END = 1;
```

### CASE con funciones de agregación
En entos ejemplos el `CASE` va dentro de la función de agregación, me funciona como un filtro para crear nuevas columnas, por ejemplo:
- Si la plataforma es instagram entonces me devuelve el valor de followers, y eso se va sumando, sino devuelve 0.
- Si la plataforma es instagram entonces me devuelve followers, y eso lo va promediando, sino, no devuelve nada.
- Si los followers son mayores a X cantidad me devuelve 1, para ir contando cuantos hay de este estilo, sino, no me devuelve nada.

> Fijense el unico que devuelve algo es ``SUM()``, porque si no se cumple suma un 0, ``AVG()`` no puede promediar 0 por eso devuelve nada, al igual que ``COUNT()``, que no puede devolver 0 porque lo contaria, por eso devuelve nada.

Ejemplo con ``SUM()``
```sql
SELECT
  SUM(CASE WHEN platform = 'Instagram' THEN followers ELSE 0 END) AS total_instagram,
  SUM(CASE WHEN platform = 'Twitter' THEN followers ELSE 0 END) AS total_twitter,
  SUM(CASE WHEN platform NOT IN ('Instagram', 'Twitter') THEN followers ELSE 0 END) AS total_otras
FROM marvel_avengers;
```

Ejemplo con ``AVG()``
```sql
SELECT 
  AVG(CASE WHEN platform = 'Instagram' THEN followers END) AS avg_instagram,
  AVG(CASE WHEN platform = 'Twitter' THEN followers END) AS avg_twitter
FROM marvel_avengers;
```

Ejemplo con ``COUNT()``
```sql
SELECT 
  COUNT(CASE WHEN followers >= 500000 THEN 1 END) AS superestrellas,
  COUNT(CASE WHEN followers BETWEEN 200000 AND 499999 THEN 1 END) AS famosos,
  COUNT(CASE WHEN followers < 200000 THEN 1 END) AS desconocidos
FROM marvel_avengers;
```

## JOINS
Un ``JOIN`` en SQL te permite combinar filas de dos o más tablas según una condición relacionada entre ellas, normalmente una clave (ID) que ambas comparten.

Existen distintos tipos de JOIN:
- ``INNER JOIN`` – Coincidencias en ambas tablas
- ``LEFT JOIN`` – 	Todo de la izquierda, más coincidencias de la derecha
- ``RIGHT JOIN`` - Todo de la derecha, más coincidencias de la izquierda
- `FULL OUTER JOIN` - Todo de ambas, coincidan o no

>Los más usados son los dos primeros.

Sintaxis básica
```sql
SELECT 
  t1.column_a, 
  t2.column_b
FROM table1 AS t1
JOIN table2 AS t2
  ON t1.common_column = t2.common_column;
```

**Ejemplo 1**, Necesitabamos mostrar las coincidencias entre ambas tablas según el `user_id`.
```sql
SELECT * FROM trades
INNER JOIN users on trades.user_id = users.user_id;
```

**Ejemplo 2**, Primero necesitabamos lo mismo de arriba, unir ambas tablas, pero ahora necesitamos mostrar TOP 3 ciudades con la mayor cantidad de tareas completadas, y ordenarlo descendente.
```sql
SELECT city, count(status) as completed_orders FROM trades
INNER JOIN users on trades.user_id = users.user_id
where status = 'Completed'
group by city
order by completed_orders DESC
limit 3
```

**Ejemplo 3**, nos dan dos tablas, la primera es `pages` que contiene `page_id` y `page_name`, esta tabla nos muestra la página con su id y su nombre. La segunda es `page_likes`, que contiene `user_id`, `page_id` y `liked_date`, esta tabla nos muestra el id de usuario la página y la fecha en que le dio like. Nos piden mostrar las páginas a las que los usuarios no le dieron like, entonces básicament es hacer un left join y las páginas donde los campos de `page_likes` salga null, significa que son páginas sin likes, y eso lo mostramos.
```sql
SELECT pages.page_id FROM pages
left join page_likes on pages.page_id = page_likes.page_id
where user_id is null
order by pages.page_id DESC
```

## DATE FUNCTIONS
Resumen de lo que se va a cubrir: 
- ``CURRENT_DATE``, ``CURRENT_TIME`` and ``CURRENT_TIMESTAMP`` to return current date, time and timestamp.
- Comparison operators ``<``, ``>``, ``=``, ``<=``, and ``>=`` to compare dates
- ``EXTRACT()`` and ``DATE_PART()`` to extract specific components of date.
- ``DATE_TRUNC()`` to round down date or timestamp into specific level of precision.
- ``INTERVAL`` to add or subtract time intervals in calculations.
- ``TO_CHAR()`` to convert date or timestamp into strings.
- ``::DATE``, ``TO_DATE()``, ``::TIMESTAMP``, and ``TO_TIMESTAMP()`` to convert strings into date or timestamp.

> Para dejar en claro:
> - Cuando hablamos de **date** es *2023-08-27*, sin el tiempo, cuando hablamos de **timestamp** es *2023-08-27 10:30:00*, con todo y tiempo.
> - Esta sintaxis es mas aplicada para Postgresql, para Mysql o Sql server, se usa otra distinta [link](https://datalemur.com/blog/mysql-postgresql-date-time-functions).

**Ejemplo** del uso de ``CURRENT_DATE``, ``CURRENT_TIME`` y ``CURRENT_TIMESTAMP``, los cuales me devuelve, la fecha actual, el tiempo actual, y ambas.
```sql
SELECT 
  message_id,
  sent_date,
  CURRENT_DATE AS current_date,
  CURRENT_TIME AS current_time,
  CURRENT_TIMESTAMP AS current_timestamp
FROM messages
LIMIT 3;
```

**Ejemplo** del uso de ``EXTRACT()`` y ``DATE_PART()``, se usan para el mismo propósito, pero tienen diferente sintaxis, se usa para extraer el año, mes, dia, hora, minutos, etc.
```sql
SELECT 
  message_id, 
  sent_date,
  EXTRACT(YEAR FROM sent_date) AS extracted_year,
  DATE_PART('year', sent_date) AS part_year,
  EXTRACT(MONTH FROM sent_date) AS extracted_month,
  DATE_PART('month', sent_date) AS part_month,
  EXTRACT(DAY FROM sent_date) AS extracted_day,
  DATE_PART('day', sent_date) AS part_day,
  EXTRACT(HOUR FROM sent_date) AS extracted_hour,
  DATE_PART('hour', sent_date) AS part_hour,
  EXTRACT(MINUTE FROM sent_date) AS extracted_minute,
  DATE_PART('minute', sent_date) AS part_minute
FROM messages
LIMIT 3;
```

**Ejemplo** del uso de ``DATE_TRUNC()``, esto me va a truncar según el parámetro que elija, por ejemplo, si mi valor en `sent_date` es *08/03/2022 16:43:00*, con month seria *08/01/2022 00:00:00*, con day *08/03/2022 00:00:00*, y con hour *08/03/2022 16:00:00*.
```sql
SELECT 
  message_id,
  sent_date,
  DATE_TRUNC('month', sent_date) AS truncated_to_month,
  DATE_TRUNC('day', sent_date) AS truncated_to_day,
  DATE_TRUNC('hour', sent_date) AS truncated_to_hour  
FROM messages
LIMIT 3;
```

**Ejemplo** del uso de ``INTERVAL``, me añade o resta dias o tiempo, según especifique.
```sql
SELECT 
  message_id,
  sent_date,
  sent_date + INTERVAL '2 days' AS add_2days,
  sent_date - INTERVAL '3 days' AS minus_3days,
  sent_date + INTERVAL '2 hours' AS add_2hours,
  sent_date - INTERVAL '10 minutes' AS minus_10mins
FROM messages
LIMIT 3;
```

**Ejemplo** del uso de ``TO_CHAR()``, solo me convierte de fecha a string según el formato que escoja.
```sql
SELECT 
  message_id,
  sent_date,
  TO_CHAR(sent_date, 'YYYY-MM-DD HH:MI:SS') AS formatted_iso8601,
  TO_CHAR(sent_date, 'YYYY-MM-DD HH:MI:SS AM') AS formatted_12hr,
  TO_CHAR(sent_date, 'Month DDth, YYYY') AS formatted_longmonth,
  TO_CHAR(sent_date, 'Mon DD, YYYY') AS formatted_shortmonth,
  TO_CHAR(sent_date, 'DD Month YYYY') AS formatted_daymonthyear,
  TO_CHAR(sent_date, 'Month') AS formatted_dayofmonth,
  TO_CHAR(sent_date, 'Day') AS formatted_dayofweek
FROM messages
LIMIT 3;
```

**Ejemplo** del uso de: 
- ``::DATE`` o ``TO_DATE()``: Convierte strings a fecha.
- ``::TIMESTAMP`` o ``TO_TIMESTAMP()``: Convierte strings a marcas de tiempo.
```sql
SELECT 
  sent_date,
  sent_date::DATE AS casted_date,
  TO_DATE('2023-08-27', 'YYYY-MM-DD') AS converted_to_date,
  sent_date::TIMESTAMP AS casted_timestamp,
  TO_TIMESTAMP('2023-08-27 10:30:00', 'YYYY-MM-DD HH:MI:SS') AS converted_to_timestamp
FROM messages
LIMIT 3;
```