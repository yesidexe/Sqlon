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
-- SELECT...
agg_función() OVER (
  PARTITION BY ... 
  ORDER BY ...
) AS ...
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

> Alguans funciones más comunes son, aparte de las ya conocidas son:
> - `ROW_NUMBER()`
> - `RANK()` / `DENSE_RANK()`
> - `FIRST_VALUE()` / `LAST_VALUE()`
> - `LAG()` / `LEAD()`

## Ranking