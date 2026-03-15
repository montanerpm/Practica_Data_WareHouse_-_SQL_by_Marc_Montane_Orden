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

Hay 266 vuelos distintos.
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

Hay 250 vuelos con más de un registro.

## Enunciado 2
### 1. Qué información cambia de un registro a otro
```sql
with vuelos_repetidos as (
    select 
        unique_identifier,
        count(*) as total_registros
    from flights
    group by 1
    having count(*) > 1
)
select 
    unique_identifier,
    total_registros
from vuelos_repetidos
order by total_registros desc
limit 5;
```

Para ver los vuelos con más registros y coger los unique_identifier para analizar.

```sql
with base as (
    select 
    	flight_row_id,
        unique_identifier,
        created_at,
        updated_at,
        local_departure,
        local_actual_departure,
        local_arrival,
        local_actual_arrival,
        gmt_departure,
        gmt_actual_departure,
        gmt_arrival,
        gmt_actual_arrival,
        departure_airport,
        arrival_airport,
        airline_code,
        delay_mins,
        arrival_status
    from flights
    where unique_identifier in (
        'AV-47-20211101-MAD-BOG',
        'KL-1704-20240111-MAD-AMS',
        'EI-8337-20230630-DUB-LHR'
    )
)
select 
	flight_row_id,
    unique_identifier,
    created_at,
    updated_at,
    local_departure,
    local_actual_departure,
    local_arrival,
    local_actual_arrival,
    gmt_departure,
    gmt_actual_departure,
    gmt_arrival,
    gmt_actual_arrival,
    departure_airport,
    arrival_airport,
    airline_code,
    delay_mins,
    arrival_status
from base
order by unique_identifier, flight_row_id asc;
```

Después de analizar los datos, lo que se va actualizando de un registro a otro, son las columnas que contienen el nombre "actual", ya que registran la hora real a la que el avión realiza la acción. Además, también se actualizan "delay_mins" y "arrival_status".

