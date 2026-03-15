# Práctica Data WareHouse &amp; SQL by Marc Montané Orden
## Enunciado 1
### 1. Cuántos registros hay en total
```sql
select count(*) as total_registros
from flights;
```

En total hay 1209 registros
### 2. Cuántos vuelos distintos hay
```sql
select count(distinct unique_identifier) as vuelos_distintos
from flights;
```

Hay 266 vuelos distintos
### 3. Cuántos vuelos tienen más de un registro
```sql
with vuelos_repetidos as (
    select unique_identifier
    from flights
    group by unique_identifier
    having count(*) > 1
)

select count(*) as vuelos_con_varios_registros
from vuelos_repetidos;
```

Hay 250 vuelos con más de un registro
