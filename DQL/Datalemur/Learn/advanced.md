# Advanced
[Link](https://datalemur.com/sql-tutorial/advanced-sql-tutorial-intro)

## CTE (Common table expressions) vs SUBQUERY
Una **CTE** (con WITH) crea una tabla temporal dentro de una consulta, lo que mejora la legibilidad y facilita mantener consultas complejas. Solo existe durante la ejecución de la consulta principal.

Una **SUBQUERY** es una consulta anidada entre paréntesis dentro de otra. Sirve para calcular o filtrar datos intermedios, permitiendo mayor precisión en el análisis.

### Ejemplos

Este [ejercicio](https://datalemur.com/questions/sql-swapped-food-delivery) es mejor si lo ven desde datalemur, porque es dificil de explicar, voy a resumirlo brevemente. 

Las órdenes se deben reordenar alternando de dos en dos: la segunda va primero, la primera va segunda, y así sucesivamente. Si la cantidad total de pedidos es impar, el último mantiene su posición; si es par, todos cambian. En otras palabras, a los impares se les suma 1 y a los pares se les resta 1, excepto el ultimo valor, en caso de que sea impar se deja como está.

Este ejercicio también se puede resolver con una **subquery**, pero usar un **CTE** lo hace más claro y eficiente, así que conviene ir acostumbrándose.
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

