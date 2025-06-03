# Basic
[Link](https://datalemur.com/sql-tutorial/intro-to-sql)
> Voy a omitir: ``SELECT``, ``WHERE``, ``AND``, ``OR``, ``NOT``, ``BETWEEN``,``IN``, y `ORDER BY`. Esto por el simple hecho de que son conceptos que ya tengo claros.

## Like Y Not like
Los operadores lógicos LIKE y NOT LIKE permiten filtrar filas basándose en si una cadena coincide con un determinado patrón.

Sintaxis básica
```sql
SELECT ...
FROM ...
WHERE column LIKE ... 
  AND/OR column NOT LIKE ...;
```

Resumen de opciones que puedes usar con `LIKE`

| Example in Query                    | Definition                                             |
|------------------------------------|--------------------------------------------------------|
| `WHERE first_name LIKE 'a%'`       | Finds any values that start with "a"                  |
| `WHERE first_name LIKE '%a'`       | Finds any values that end with "a"                    |
| `WHERE first_name LIKE '%ae%'`     | Finds any values that have "ae" in the middle         |
| `WHERE first_name LIKE '_b%'`      | Finds any values with "b" in the second position      |
| `WHERE first_name LIKE 'a%o'`      | Finds any values that start with "a" and end with "o" |
| `WHERE first_name LIKE 'a___'`     | Finds any value that starts with "a" and has 4 letters (a + 3 more) |

