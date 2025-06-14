# Advanced
[Link](https://datalemur.com/sql-tutorial/advanced-sql-tutorial-intro)

## CTE (Common table expressions) vs SUBQUERY
Una **CTE** (con WITH) crea una tabla temporal dentro de una consulta, lo que mejora la legibilidad y facilita mantener consultas complejas. Solo existe durante la ejecución de la consulta principal.

Una **SUBQUERY** es una consulta anidada entre paréntesis dentro de otra. Sirve para calcular o filtrar datos intermedios, permitiendo mayor precisión en el análisis.

### Ejemplo 1 

Este [ejercicio](https://datalemur.com/questions/sql-swapped-food-delivery) es mejor si lo ven desde datalemur, porque es dificil de explicar, voy a resumirlo brevemente. 

Las órdenes se deben reordenar alternando de dos en dos: la segunda va primero, la primera va segunda, y así sucesivamente. Si la cantidad total de pedidos es impar, el último mantiene su posición; si es par, todos cambian. En otras palabras, a los impares se les suma 1 y a los pares se les resta 1, excepto el ultimo valor, en caso de que sea impar se deja como está.

>Este ejercicio también se puede resolver con una **subquery**, pero usar un **CTE** lo hace más claro y eficiente, así que conviene ir acostumbrándose.
```sql
with total_orders as (
select count(order_id) as total_orders from orders
)

select CASE
  when order_id = t.total_orders then order_id
  when order_id%2=0 then order_id-1
  else order_id+1
end as corrected_order_id,
item
from orders
cross join total_orders as t
order by corrected_order_id
```

Existe uan solución usando el tema de ***Window Functions*** que veremos más adelante
```sql
select CASE
  when order_id = COUNT(order_id) OVER () THEN order_id
  when order_id%2=0 then order_id-1
  else order_id+1
end as corrected_order_id,
item
from orders
order by corrected_order_id
```

### Ejemplo 2

Este [ejericio](https://datalemur.com/questions/supercloud-customer) también es bastante enredado la primera vez, y más si no sabias que se podia hacer el ``distinct`` en ``count``. La idea es encontrar qué usuarios han comprado al menos un producto de cada categoría disponible. 

Primero se agrupa por ``customer_id``, y para cada uno se cuenta cuántas categorías distintas de producto ha comprado.

Después, se compara ese número con el total de categorías existentes en la tabla ``products``. Si coinciden, significa que ese cliente ha comprado al menos un producto de cada categoría, y se considera "super cloud customer"

```sql
with category_products_count as (
  select c.customer_id, count(distinct p.product_category) as category_count 
  from customer_contracts as c 
  inner join products as p on p.product_id = c.product_id
  group by c.customer_id
)

select customer_id from category_products_count as cpc
where cpc.category_count = (select count(DISTINCT product_category) from products)
```

## Window Functions
Una window function me crea una nueva columna, donde realiza un cálculo (como suma, promedio, ranking, etc.) sobre un conjunto de filas relacionadas (si es el caso). Este conjunto de filas se llama “ventana”.

Sintaxis básica, usando ``PARTITION BY`` y ``ORDER BY``
```sql
SELECT
agg_función() OVER (
  PARTITION BY ... 
  ORDER BY ...
) AS ...
FROM
```

Diferencias del uso de solo ``OVER`` con o sin ``PARTITION BY`` y ``ORDER BY``, para este caso la agg_funcion es ``SUM()``, pero vale para todas, hay que agarrarle la lógica.

| ¿Incluye `PARTITION BY`? | ¿Incluye `ORDER BY`? | ¿Qué hace?                                                                 |
|--------------------------|----------------------|-----------------------------------------------------------------------------|
| ❌ No                    | ❌ No                | Muestra el **mismo valor total** en todas las filas.   |
| ✅ Sí                    | ❌ No                | Muestra el **total por grupo**, repetido para cada fila del grupo.         |
| ✅ o ❌                  | ✅ Sí               | Muestra un **acumulado progresivo** (running total), según la partición dada. |

**Ejemplo**, para este [ejercicio](https://datalemur.com/questions/card-launch-success), la idea es mostrar la cantidad de problemas que tuvo la tarjeta de crédito el primer mes de su lanzamiento, por lo tanto, mostramos el nombre de la tarjeta y su primer valor separado por ``card_name`` y ordenado por ``issue_year``, ``issue_month``, esto en un **CTE**, como nos dan varios valores y solo queremos uno, podemos hacer uso de un ``group by``, y en este caso usé un ``max()``

```sql
with card_issues as (
  select card_name,
  FIRST_VALUE(issued_amount) over (
  partition by card_name
  ORDER BY issue_year, issue_month
  ) as issued_amount
  from monthly_cards_issued
)

select card_name, max(issued_amount) as issued_amount from card_issues
group by card_name
order by issued_amount desc
```

> Alguans funciones más comunes aparte de las ya conocidas son:
> - `ROW_NUMBER()`
> - `RANK()` / `DENSE_RANK()`
> - `FIRST_VALUE()` / `LAST_VALUE()`
> - `LAG()` / `LEAD()`

## Ranking
Es una función de ventana que asigna un número de ranking a cada fila dentro de una partición, según el orden que vos le des. Si hay empates, les da el mismo número… pero salta el siguiente.

Sintaxis basica
```sql
SELECT 
  RANK() / DENSE_RANK() / ROW_NUMBER() OVER ( -- Compulsory expression
    PARTITION BY partitioning_expression -- Optional expression
    ORDER BY order_expression) -- Compulsory expression
FROM table_name;
```

Diferencias entre ``ROW_NUMBER()``, ``RANK()`` y ``DENSE_RANK()``

| Función        | ¿Salta números si hay empates? | ¿Da un número único por fila? | ¿Qué hace si hay empate?                                              |
| -------------- | ------------------------------ | ----------------------------- | --------------------------------------------------------------------- |
| `ROW_NUMBER()` | ❌ No                           | ✅ Sí                          | Asigna números únicos, **sin importar empates**                       |
| `RANK()`       | ✅ Sí                           | ❌ No (puede repetir)          | **Empates comparten posición**, y se **saltan** números después       |
| `DENSE_RANK()` | ❌ No                           | ❌ No (puede repetir)          | **Empates comparten posición**, pero **no se saltan** números después |

Ejemplo visual
```sql
SELECT 
  nombre, score,
  ROW_NUMBER() OVER (ORDER BY score DESC) AS row_num,
  RANK()       OVER (ORDER BY score DESC) AS rank,
  DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM tabla
```

| nombre | score | row\_num | rank | dense\_rank |
| ------ | ----- | -------- | ---- | ----------- |
| Ana    | 100   | 1        | 1    | 1           |
| Beto   | 90    | 2        | 2    | 2           |
| Carla  | 90    | 3        | 2    | 2           |
| David  | 80    | 4        | 4    | 3           |

### Ejemplo 1
En este [ejercicio](https://datalemur.com/questions/top-fans-rank), tenemos tres tablas, ``artists``,`songs`, y `global_song_rank`, lo que nos piden es mostrar es el top 5 de los artistas con más canciones en el top 10.

```sql
with prueba as (
  SELECT artist_name,
    DENSE_RANK() over(
    order by count(songs.song_id) desc) as artist_rank
  FROM artists
  inner join songs on artists.artist_id = songs.artist_id
  inner join global_song_rank on songs.song_id = global_song_rank.song_id
  where global_song_rank.rank <= 10
  group by artist_name
)

select * from prueba
where artist_rank <= 5
```

### Ejemplo 2
En este [ejercicio](https://datalemur.com/questions/histogram-users-purchases) se registra cada vez que se hizo una compra, entonces tenemos que mostrar el id del usuario junto con la fecha en que se hizo su ultima transacción y los productos que compró en esa ultima fecha. el chiste de usar `RANK()` es para separar por `user_id` y ordenar según `transaction_date` y así tendremos la ultima fecha, por qué usar *RANK* (También serviria *DENSE*)? porque puede ser que hayan dos registros con la misma fecha y se necesitan contar los registros donde `transaction_rank=1` o sea, la ultima fecha.

```sql
with latest_transactions as (
select user_id, transaction_date, product_id,
RANK() OVER (
  PARTITION BY user_id ORDER BY transaction_date DESC
) AS transaction_rank
from user_transactions  
)

select user_id, transaction_date, count(transaction_rank) as purchase_count from latest_transactions
where transaction_rank=1
group by user_id, transaction_date
order by transaction_date
```

### Ejemplo 3
Este [ejercicio](https://datalemur.com/questions/odd-even-measurements), no se entiende bien, entonces voy a tratar de explicarlo, me pide mostrar el dia (por eso tengo que pasar la fecha a dia), junto con una columna `odd_sum` y otra `even_sum`. Los valores de la suma se toman por registros, es decir, el primer registro y tercer registro del dia van en `odd_sum`, y el segundo y cuarto en `even_sum`, o sea, los pares e impares.
```sql
with ranked_measurements as (
SELECT measurement_value,
DATE_TRUNC('day', measurement_time) as measurement_day,
ROW_NUMBER() over(
partition by DATE_TRUNC('day', measurement_time) order by measurement_time asc)
FROM measurements
)

select measurement_day,
sum(case when row_number%2=0 then measurement_value else 0 end) as even_sum,
sum(case when row_number%2!=0 then measurement_value else 0 end) as odd_sum
from ranked_measurements
group by measurement_day
order by measurement_day asc
```

## LEAD & LAG
Son window functions que se utilizan para acceder a una fila anterior o posterior (dependiendo de `offset`) a la fila actual dentro de una partición o conjunto de datos ordenado.

Sintaxis (para `LEAD()` es igual)
```sql
LAG(column_name, offset) OVER ( -- Compulsory expression
  PARTITION BY partition_column -- Optional expression
  ORDER BY order_column) -- Compulsory expression
```

En este **ejemplo**, el [ejercicio](https://datalemur.com/questions/yoy-growth-rate) nos pide mostrar agrupado por ``product_id`` y ``year``, el spend del año actual, con el anterior y su % de variación.

```sql
with prueba as(
  SELECT
  DATE_PART('year', transaction_date) AS year,
  product_id,
  spend as curr_year_spend,
  lag(spend) over (partition by product_id order by DATE_PART('year', transaction_date)) as prev_year_spend
  FROM user_transactions
)

select *,  
round(((curr_year_spend-prev_year_spend)/prev_year_spend)*100,2) as yoy_rate
from prueba
```

## SELF-JOINS
Es confuso, sin embargo una definición seria algo como, un SELF JOIN es cuando una tabla se une consigo misma para poder comparar o relacionar sus propias filas.

En este **ejemplo**, el [ejercicio](https://datalemur.com/questions/sql-well-paid-employees) me pide mostrar el ID y el nombre del empleado que gana más que su manager, en esta ocación voy a mostrar un ejemplo de la tabla para que sea mas facil entender, entonces lo que hago es usar el ``inner join`` para agregar la nueva tabla (que es la misma pero la nombré managers) donde igualé el ``employee_id`` con ``manager_id``, para por decirlo así generar una tabla de solo managers.

| employee_id | name                | salary | department_id | manager_id |
|-------------|---------------------|--------|----------------|-------------|
| 1           | Emma Thompson       | 3800   | 1              | 6           |
| 2           | Daniel Rodriguez    | 2230   | 1              | 7           |
| 3           | Olivia Smith        | 7000   | 1              | 8           |
| 4           | Noah Johnson        | 6800   | 2              | 9           |
| 5           | Sophia Martinez     | 1750   | 1              | 11          |
| 6           | Liam Brown          | 13000  | 3              | NULL        |
| 7           | Ava Garcia          | 12500  | 3              | NULL        |
| 8           | William Davis       | 6800   | 2              | NULL        |
| 9           | Isabella Wilson     | 11000  | 3              | NULL        |
| 10          | James Anderson      | 4000   | 1              | 11          |
| 11          | Mia Taylor          | 10800  | 3              | NULL        |
| 12          | Benjamin Hernandez  | 9500   | 3              | 8           |
| 13          | Charlotte Miller    | 7000   | 2              | 6           |
| 14          | Logan Moore         | 8000   | 2              | 6           |
| 15          | Amelia Lee          | 4000   | 1              | 7           |


```sql
select employees.employee_id as employee_id,
employees.name as employee_name
from employee as employees
inner join employee as managers
on employees.manager_id = managers.employee_id
where employees.salary > managers.salary
```

## UNION, INTERCEPT, EXCEPT

### UNION / UNION ALL

Mientras que el JOIN une tablas de manera horizontal, el UNION lo hace de manera vertical, existe el `UNION` que me une las tablas removiendo las filas duplicadas, y el ``UNION ALL`` que no las remueve. Existen ciertas reglas obvias como sean el mismo numero y tipo de columnas, además de tener el mismo orden de las  columnas.

Por ejemplo, en este [ejercicio](https://datalemur.com/questions/prime-warehouse-storage) que me costó fue entender cómo sacar las fórmulas, ya que no eran tan evidentes al principio. Normalmente uno pensaría que estas fórmulas ya vendrían dadas, pero aquí había que deducirlas.  
La idea era calcular cuántos lotes (batches) de productos se pueden almacenar en una bodega de 500,000 pies cuadrados, dando prioridad a los productos prime. Para resolverlo, primero uso un **CTE** llamado variables donde calculo los valores clave como el área total y cantidad de ítems por tipo (**prime** y **not_prime**). Luego, en otro **CTE** (prime_area) calculo cuánto espacio ocuparían los lotes prime. Finalmente, uso un `UNION` para juntar los resultados de cuántos lotes de cada tipo se pueden almacenar, usando FLOOR para asegurar que los valores sean enteros y no se exceda el espacio.

```sql
with variables as (
select
  sum(square_footage) filter (where item_type = 'prime_eligible') as prime_sf,
  count(item_id) filter (where item_type = 'prime_eligible') as prime_count,
  sum(square_footage) filter (where item_type = 'not_prime') as not_prime_sf,
  count(item_id) filter (where item_type = 'not_prime') as not_prime_count
from inventory
), prime_area as(
select floor(500000/prime_sf)*prime_sf as pa
from variables
)

select 'prime_eligible' as item_type,
FLOOR(500000/prime_sf)*prime_count AS item_count
from variables
UNION all
select 'not_prime' as item_type,
floor((500000-(select pa from prime_area))/not_prime_sf)*not_prime_count as item_count
from variables
```

### INTERSECT
Funciona igual que un `INNER JOIN` pero en lugar de funcionar de manera horizontal (agregar columnas), funciona en vertical (agrega filas).

Un **ejemplo** sencillo seria que me muestre los ingredientes en común que tienen estas dos tablas (dos recetas).
```sql
SELECT ingredient
FROM recipe_1
INTERSECT
SELECT ingredient
FROM recipe_2;
```

Otro **ejemplo** quizás mas elaborado seria este, que me mostraria los `order_id` resultates de realizar estas dos consultas de estas dos tablas.
```sql
SELECT order_id
FROM orders
WHERE quantity >= 2
INTERSECT
SELECT order_id
FROM deliveries
WHERE delivery_status = 'Delivered';
```

### EXCEPT
`EXCEPT` muestra los datos que están en la **primera consulta** pero **no** en la **segunda**. Se puede interpretar como: “Muéstrame los resultados de esta consulta **excepto** los que también aparezcan en esta otra, o Muestrame los resultados de esta consulta que no salen en esta otra”.

Un **ejemplo** breve sería mostrar los ingredientes de la tabla `recipe_1` que **no están** en la tabla `recipe_2`:

```sql
SELECT ingredient
FROM recipe_1
EXCEPT
SELECT ingredient
FROM recipe_2;
```

Un **ejemplo** de este [ejercicio](https://datalemur.com/questions/sql-page-with-no-likes) sería mostrar los `page_id` de la tabla `pages` que **no aparecen** en la tabla `page_likes`.  
¿Por qué? Porque el ejercicio pide identificar las páginas con **cero likes**.  
En la tabla `pages` están todos los ID de páginas, mientras que en `page_likes` solo aparecen los ID de las páginas que han recibido al menos un like.  
Por lo tanto, si usamos `EXCEPT`, podemos obtener los `page_id` que existen pero **no tienen likes**, ya que no figuran en la segunda tabla.

```sql
SELECT page_id
FROM pages
EXCEPT
SELECT page_id
FROM page_likes
order by page_id;
```

## WRITE CLEAN SQL
Buenas practicas a la hora de escribir código en SQL:

### Uppercase for Keywords
❌ Evitar:
```sql
select
  id, 
  product_name
  sum(amount) as total_amount
from company.transactions;
```

✔ Mejor:
```sql
SELECT
  id, 
  product_name
  SUM(amount) AS total_amount  
FROM company.products;
```

### Lowercase or Snake Case for Names
Cuando se trata de nombrar esquemas, tablas o columnas es mejor nombrarlo en **lowercase** o **snake_case**, además de claro mantener la consistencia en caso de trabajar ya con código ya sea de un equipo o de un proyecto.

❌ Evitar:
```sql
SELECT 
  Users.City, 
  COUNT(Trades.OrderId) AS TotalOrders 
FROM Trades
INNER JOIN Users 
  ON TRADES.UserId = Users.UserId 
WHERE Trades.Status = 'Completed' 
GROUP BY Users.City 
ORDER BY TotalOrders DESC;
```
✔ Mejor:
```sql
SELECT 
  users.city, 
  COUNT(trades.order_id) AS total_orders 
FROM trades 
INNER JOIN users 
  ON trades.user_id = users.user_id 
WHERE trades.status = 'Completed' 
GROUP BY users.city 
ORDER BY total_orders DESC;
```

### Descriptive and Concise Aliases
Nombrar las cosas que se entienda

❌ Evitar
```sql
SELECT 
  plc.ticker,
  AVG(sp.close) average,
  ROUND(AVG(sp.close), 2) average_c
FROM stock_prices sp
INNER JOIN public_listed_companies plc
  ON sp.ticker = plc.ticker
WHERE EXTRACT(YEAR FROM sp.date) = 2022
GROUP BY pls.ticker;
```
✔ Mejor:
```sql
SELECT 
  companies.ticker,
  AVG(prices.close) AS avg_close,
  ROUND(AVG(prices.close), 2) AS rounded_avg_close
FROM stock_prices AS prices
INNER JOIN public_listed_companies AS companies
  ON prices.ticker = companies.ticker
WHERE EXTRACT(YEAR FROM prices.date) = 2022
GROUP BY companies.ticker;
```

### Consistent Formatting and Indentation
Mantener el formato, que se vea bien

❌ Evitar
```sql
WITH product_sales AS (
SELECT
product_id,
  SUM(sales_amount) AS total_sales
FROM sales
GROUP BY product_id
  )

SELECT
products.product_name,
sales.total_sales
FROM products
  INNER JOIN product_sales sales ON products.product_id = sales.product_id
```

✔ Mejor: 
```sql
WITH product_sales AS (
  SELECT
    product_id,
    SUM(sales_amount) AS total_sales
  FROM sales
  GROUP BY product_id)

SELECT
  products.product_name,
  sales.total_sales
FROM products
INNER JOIN product_sales AS sales
  ON products.product_id = sales.product_id;
```

### Avoid Writing SELECT *
No abusar del ``*``, mejor escribir lo que se va a usar, queda mas legible de paso.

❌ Evitar:
```sql
SELECT * 
FROM employees;
```

✔ Mejor:
```sql
SELECT 
  employee_id, 
  first_name, 
  last_name 
FROM employees;
```

### Use JOINs Explicitly for Clarity
❌ Evitar:
```sql
SELECT 
  users.name, 
  orders.order_date
FROM users, orders -- Avoid joining tables like this
WHERE users.id = orders.user_id;
```

✔ Mejor:
```sql
SELECT 
  users.name AS user_name, 
  orders.order_date
FROM users
INNER JOIN orders -- Use the JOIN keyword
  ON users.id = orders.user_id;
```

### Format Dates Consistently
❌ Evitar:
```sql
SELECT * 
FROM orders 
WHERE order_date = '01-12-2012';
```

✔ Mejor:
```sql
SELECT * 
FROM orders 
WHERE order_date = '2012-12-01';
```

### Comment Wisely
Usar los comentarios `--`, `/**/` y bien...

❌ Evitar:
```sql
-- This query retrieves data from the sales table
SELECT
  product_name,
  SUM(sales_amount) AS total_sales
FROM sales
GROUP BY product_name;
```

✔ Mejor:
```sql
-- Calculate the total sales for each product
SELECT
  product_name,
  SUM(sales_amount) AS total_sales
FROM sales
GROUP BY product_name;
```

## EXECUTION ORDER
| Cláusula     | Orden | Descripción                                                                                                                                      |
|--------------|-------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| FROM         | 1     | La consulta comienza con la cláusula FROM, donde la base de datos identifica las tablas involucradas y accede a los datos necesarios.           |
| WHERE        | 2     | La base de datos aplica las condiciones especificadas en la cláusula WHERE para filtrar los datos obtenidos de las tablas indicadas en FROM.    |
| GROUP BY     | 3     | Si hay una cláusula GROUP BY, los datos se agrupan según las columnas indicadas y se aplican funciones de agregación (como SUM(), AVG(), COUNT()). |
| HAVING       | 4     | La cláusula HAVING filtra los datos agregados basándose en las condiciones especificadas.                                                       |
| SELECT       | 5     | La cláusula SELECT define las columnas que se incluirán en el conjunto de resultados final.                                                     |
| ORDER BY     | 6     | Si se utiliza la cláusula ORDER BY, el conjunto de resultados se ordena según las columnas especificadas.                                       |
| LIMIT/OFFSET | 7     | Si hay una cláusula LIMIT u OFFSET, el conjunto de resultados se limita al número de filas indicado y opcionalmente se omiten ciertas filas.     |

## PIVOTING / UNPIVOTING
Para esto se suele usar el `CASE` junto con una función de agregación depende lo que queramos hacer, y un `GROUP BY`.

Por **ejemplo**, haremos un pivot de `engagement_rate`, para cada heroe en cada plataforma, y por supuesto agrupamos por heroe.
```sql
SELECT
  superhero_alias,
  MAX(CASE WHEN platform = 'Instagram' THEN engagement_rate END) AS instagram_engagement_rate,
  MAX(CASE WHEN platform = 'Twitter' THEN engagement_rate END) AS twitter_engagement_rate,
  MAX(CASE WHEN platform = 'TikTok' THEN engagement_rate END) AS tiktok_engagement_rate,
  MAX(CASE WHEN platform = 'YouTube' THEN engagement_rate END) AS youtube_engagement_rate
FROM marvel_avengers
WHERE superhero_alias IN ('Iron Man', 'Captain America', 'Black Widow', 'Thor')
GROUP BY superhero_alias
ORDER BY superhero_alias;
```

Luego quitamos el pivot, aunque no le veo sentido porque mejor dejamos la que estaba pero da igual, por si acaso.
```sql
SELECT
  superhero_alias,
  platform,
  CASE platform
    WHEN 'Instagram' THEN engagement_rate
    WHEN 'Twitter' THEN engagement_rate
    WHEN 'YouTube' THEN engagement_rate
    WHEN 'TikTok' THEN engagement_rate
  END AS engagement_rate
FROM marvel_avengers
WHERE superhero_alias IN ('Iron Man', 'Captain America', 'Black Widow', 'Thor')
ORDER BY superhero_alias;
```

## STRING Functions
La verdad son bastante sencillos, ni voy a poner ejemplos, cualquier cosa buscar en internet, es lo mismo de siempore que se suele ver en otros lenguajes.

- Tidying up data with ``UPPER()`` and ``LOWER()``
- Extracting substrings with ``LEFT()`` and ``RIGHT()``
- Calculating length with ``LENGTH()``
- Determining position with ``POSITION()``
- Removing spaces using ``TRIM()``, ``LTRIM()``, ``RTRIM()``, and ``BTRIM()``
- Concatenating strings with CONCAT()
- Concatenating strings with separators using CONCAT_WS()
- Slicing and dicing strings using ``SUBSTRING()``
- Extracting substring based on delimiter using ``SPLIT_PART()``