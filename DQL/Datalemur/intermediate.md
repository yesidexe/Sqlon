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