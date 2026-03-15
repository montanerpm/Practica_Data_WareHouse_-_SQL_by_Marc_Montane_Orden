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

## Enunciado 3
### 1. La información de created_at debe ser única para cada vuelo aunque tenga más de un registro.
```sql
with base as (
    select 
        unique_identifier,
        created_at
    from flights
),
info_created_at as (
    select 
        unique_identifier,
        count(*) as total_registros,
        count(created_at) as total_created_at_no_null,
        count(distinct created_at) as total_created_at_distintos
    from base
    group by 1
)
select 
    unique_identifier,
    total_registros,
    total_created_at_no_null,
    total_created_at_distintos
from info_created_at
order by total_created_at_distintos desc, total_registros desc;
```
Para la mayoria de casos, la información es unica, aunque hay algunos vuelos donde en todos los registros, el campo "created_at" es NULL. En concreto, de los 266 vuelos, 43 tienen la columna "created_at" en NULL.

### 2. La información de updated_at deber ser igual o más que la información de created_at, lo que nos indica coherencia y consistencia
```sql
with base as (
    select 
        unique_identifier,
        created_at,
        updated_at
    from flights
),
info_validacion as (
    select 
        unique_identifier,
        count(*) as total_registros,
        count(created_at) as total_created_at_no_null,
        count(updated_at) as total_updated_at_no_null,
        count(
            case 
                when updated_at >= created_at then 1
            end
        ) as total_registros_validos
    from base
    group by 1
)
select 
    unique_identifier,
    total_registros,
    total_created_at_no_null,
    total_updated_at_no_null,
    total_registros_validos
from info_validacion
order by total_registros desc;
```
Todos los registros de "update_at" son iguales o más que los de "created_at" exceptuando cuando los registros de ambos son NULL.

## Enunciado 4
```sql
with base as (
    select 
        flight_row_id,
        unique_identifier,
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
        arrival_status,
        created_at,
        updated_at
    from flights
),
final as (
    select 
        flight_row_id,
        unique_identifier,
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
        arrival_status,
        created_at,
        updated_at,
        row_number() over(
            partition by unique_identifier
            order by coalesce(updated_at, created_at) desc, flight_row_id desc
        ) as rn
    from base
)
select 
    flight_row_id,
    unique_identifier,
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
    arrival_status,
    created_at,
    updated_at
from final
where rn = 1;
```

## Enunciado 5
```sql
with last_flights_status as (
    with base as (
        select 
            flight_row_id,
            unique_identifier,
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
            arrival_status,
            created_at,
            updated_at
        from flights
    ),
    final as (
        select 
            flight_row_id,
            unique_identifier,
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
            arrival_status,
            created_at,
            updated_at,
            row_number() over(
                partition by unique_identifier
                order by flight_row_id desc
            ) as rn
        from base
    )
    select 
        flight_row_id,
        unique_identifier,
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
        arrival_status,
        created_at,
        updated_at
    from final
    where rn = 1
),
effective_flights as (
    select 
        flight_row_id,
        unique_identifier,
        local_departure,
        local_actual_departure,
        local_arrival,
        local_actual_arrival,
        created_at,
        coalesce(local_departure, created_at) as effective_local_departure,
        coalesce(local_actual_departure, local_departure, created_at) as effective_local_actual_departure,
        coalesce(local_arrival, created_at) as effective_local_arrival,
        coalesce(local_actual_arrival, local_arrival, created_at) as effective_local_actual_arrival
    from last_flights_status
)
select 
    flight_row_id,
    unique_identifier,
    local_departure,
    local_actual_departure,
    local_arrival,
    local_actual_arrival,
    created_at,
    effective_local_departure,
    effective_local_actual_departure,
    effective_local_arrival,
    effective_local_actual_arrival
from effective_flights;
```

## Enunciado 6
### 1. Qué estados de vuelo existen
```sql
with last_flights_status as (
    with base as (
        select 
            flight_row_id,
            unique_identifier,
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
            arrival_status,
            created_at,
            updated_at
        from flights
    ),
    final as (
        select 
            flight_row_id,
            unique_identifier,
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
            arrival_status,
            created_at,
            updated_at,
            row_number() over(
                partition by unique_identifier
                order by flight_row_id desc
            ) as rn
        from base
    )
    select 
        flight_row_id,
        unique_identifier,
        arrival_status
    from final
    where rn = 1
)
select distinct
    arrival_status
from last_flights_status
order by arrival_status asc;
```

Los estados que existen son: CX (Cancelled), DY (Delayed), EY (Early), NS (Not scheduled) y OT (On time)

### 2. Cuántos vuelos hay por cada estado
```sql
with last_flights_status as (
    with base as (
        select 
            flight_row_id,
            unique_identifier,
            arrival_status
        from flights
    ),
    final as (
        select 
            flight_row_id,
            unique_identifier,
            arrival_status,
            row_number() over(
                partition by unique_identifier
                order by flight_row_id desc
            ) as rn
        from base
    )
    select 
        flight_row_id,
        unique_identifier,
        arrival_status
    from final
    where rn = 1
)

select 
    arrival_status,
    count(*) as total_vuelos
from last_flights_status
group by arrival_status
order by total_vuelos desc;
```

Los numeros de vuelos por estado son: CX (Cancelled) 5 vuelos, DY (Delayed) 143 vuelos, EY (Early) 9 vuelos, NS (Not scheduled) 3 vuelos y OT (On time) 95 vuelos.

## Enunciado 7
### 1. De qué país despegan los vuelos
```sql
with last_flights_status as (
    with base as (
        select 
            flight_row_id,
            unique_identifier,
            departure_airport,
            arrival_airport
        from flights
    ),
    final as (
        select 
            flight_row_id,
            unique_identifier,
            departure_airport,
            arrival_airport,
            row_number() over(
                partition by unique_identifier
                order by flight_row_id desc
            ) as rn
        from base
    )
    select 
        flight_row_id,
        unique_identifier,
        departure_airport,
        arrival_airport
    from final
    where rn = 1
)

select 
    f.unique_identifier,
    f.departure_airport,
    a1.city as departure_city,
    a1.country as departure_country,
    f.arrival_airport,
    a2.city as arrival_city,
    a2.country as arrival_country
from last_flights_status f
left join airports a1
    on f.departure_airport = a1.airport_code
left join airports a2
    on f.arrival_airport = a2.airport_code;
```

### 2. Cuántos vuelos despegan por país
```sql
with last_flights_status as (
    with base as (
        select 
            flight_row_id,
            unique_identifier,
            departure_airport,
            arrival_airport
        from flights
    ),
    final as (
        select 
            flight_row_id,
            unique_identifier,
            departure_airport,
            arrival_airport,
            row_number() over(
                partition by unique_identifier
                order by flight_row_id desc
            ) as rn
        from base
    )
    select 
        flight_row_id,
        unique_identifier,
        departure_airport,
        arrival_airport
    from final
    where rn = 1
)

select 
    a.country,
    count(*) as total_vuelos
from last_flights_status f
left join airports a
    on f.arrival_airport = a.airport_code
group by a.country
order by total_vuelos desc;
```

En españa despegan 128 vuelos, de estados unidos 43, de Paises Bajos 31 y de reino unido 1.

## Enunciado 8
### 1. Cuál es el delay medio por país
```sql
with last_flights_status as (
    with base as (
        select 
            flight_row_id,
            unique_identifier,
            departure_airport,
            delay_mins
        from flights
    ),
    final as (
        select 
            flight_row_id,
            unique_identifier,
            departure_airport,
            delay_mins,
            row_number() over(
                partition by unique_identifier
                order by flight_row_id desc
            ) as rn
        from base
    )
    select 
        flight_row_id,
        unique_identifier,
        departure_airport,
        delay_mins
    from final
    where rn = 1
),
info_country as (
    select 
        f.unique_identifier,
        f.delay_mins,
        a.country
    from last_flights_status as f
    left join airports as a
        on f.departure_airport = a.airport_code
)
select 
    country,
    round(avg(delay_mins), 2) as delay_medio
from info_country
group by 1
order by delay_medio desc;
```

### 2. Cuál es la distribución de estados de vuelos por país.
```sql
with last_flights_status as (
    with base as (
        select 
            flight_row_id,
            unique_identifier,
            departure_airport,
            arrival_status
        from flights
    ),
    final as (
        select 
            flight_row_id,
            unique_identifier,
            departure_airport,
            arrival_status,
            row_number() over(
                partition by unique_identifier
                order by flight_row_id desc
            ) as rn
        from base
    )
    select 
        flight_row_id,
        unique_identifier,
        departure_airport,
        arrival_status
    from final
    where rn = 1
),
info_country as (
    select 
        f.unique_identifier,
        f.arrival_status,
        a.country
    from last_flights_status as f
    left join airports as a
        on f.departure_airport = a.airport_code
)
select 
    country,
    arrival_status,
    count(*) as total_vuelos
from info_country
group by 1, 2
order by country asc, total_vuelos desc;
```

## Enunciado 9
```sql
with last_flights_status as (
    with base as (
        select 
            flight_row_id,
            unique_identifier,
            departure_airport,
            local_departure,
            delay_mins
        from flights
    ),
    final as (
        select 
            flight_row_id,
            unique_identifier,
            departure_airport,
            local_departure,
            delay_mins,
            row_number() over(
                partition by unique_identifier
                order by flight_row_id desc
            ) as rn
        from base
    )
    select 
        unique_identifier,
        departure_airport,
        local_departure,
        delay_mins
    from final
    where rn = 1
),

info_country as (
    select 
        f.unique_identifier,
        f.local_departure,
        f.delay_mins,
        a.country
    from last_flights_status f
    left join airports a
        on f.departure_airport = a.airport_code
),

info_season as (
    select 
        country,
        delay_mins,
        case 
            when extract(month from local_departure) in (12,1,2) then 'Invierno'
            when extract(month from local_departure) in (3,4,5) then 'Primavera'
            when extract(month from local_departure) in (6,7,8) then 'Verano'
            when extract(month from local_departure) in (9,10,11) then 'Otoño'
        end as season
    from info_country
)

select 
    country,
    season,
    round(avg(delay_mins),2) as avg_delay
from info_season
group by 1,2
order by country, avg_delay desc;
```

## Enunciado 10
```sql
with base as (
    select 
        flight_row_id,
        unique_identifier,
        departure_airport,
        created_at,
        updated_at,
        coalesce(updated_at, created_at) as effective_update_ts
    from flights
),
info_updates as (
    select 
        flight_row_id,
        unique_identifier,
        departure_airport,
        effective_update_ts,
        lag(effective_update_ts) over(
            partition by unique_identifier
            order by effective_update_ts asc, flight_row_id asc
        ) as previous_update_ts
    from base
),
info_diff as (
    select 
        flight_row_id,
        unique_identifier,
        departure_airport,
        effective_update_ts,
        previous_update_ts,
        effective_update_ts - previous_update_ts as update_frequency
    from info_updates
)
select 
    departure_airport,
    round(avg(extract(epoch from update_frequency) / 60), 2) as avg_update_frequency_minutes
from info_diff
where previous_update_ts is not null
group by 1
order by avg_update_frequency_minutes desc;
```

## Enunciado 11
### 1. Crea un flag is consistent.
```sql
with last_flights_status as (
    with base as (
        select 
            flight_row_id,
            unique_identifier,
            airline_code,
            departure_airport,
            arrival_airport,
            local_departure
        from flights
    ),
    final as (
        select 
            flight_row_id,
            unique_identifier,
            airline_code,
            departure_airport,
            arrival_airport,
            local_departure,
            row_number() over(
                partition by unique_identifier
                order by flight_row_id desc
            ) as rn
        from base
    )
    select 
        flight_row_id,
        unique_identifier,
        airline_code,
        departure_airport,
        arrival_airport,
        local_departure
    from final
    where rn = 1
),
info_parts as (
    select 
        flight_row_id,
        unique_identifier,
        airline_code,
        departure_airport,
        arrival_airport,
        local_departure,
        split_part(unique_identifier, '-', 1) as uid_airline_code,
        split_part(unique_identifier, '-', 2) as uid_flight_number,
        split_part(unique_identifier, '-', 3) as uid_flight_date,
        split_part(unique_identifier, '-', 4) as uid_departure_airport,
        split_part(unique_identifier, '-', 5) as uid_arrival_airport
    from last_flights_status
)
select 
    flight_row_id,
    unique_identifier,
    airline_code,
    departure_airport,
    arrival_airport,
    local_departure,
    case
        when airline_code = uid_airline_code
         and to_char(local_departure, 'YYYYMMDD') = uid_flight_date
         and departure_airport = uid_departure_airport
         and arrival_airport = uid_arrival_airport
        then true
        else false
    end as is_consistent
from info_parts;
```

### 2. Calcula cuántos vuelos no son consistentes.
```sql
with last_flights_status as (
    with base as (
        select 
            flight_row_id,
            unique_identifier,
            airline_code,
            departure_airport,
            arrival_airport,
            local_departure
        from flights
    ),
    final as (
        select 
            flight_row_id,
            unique_identifier,
            airline_code,
            departure_airport,
            arrival_airport,
            local_departure,
            row_number() over(
                partition by unique_identifier
                order by flight_row_id desc
            ) as rn
        from base
    )
    select *
    from final
    where rn = 1
),

info_parts as (
    select 
        *,
        split_part(unique_identifier, '-', 1) as uid_airline_code,
        split_part(unique_identifier, '-', 3) as uid_flight_date,
        split_part(unique_identifier, '-', 4) as uid_departure_airport,
        split_part(unique_identifier, '-', 5) as uid_arrival_airport
    from last_flights_status
),

consistency_check as (
    select 
        *,
        case
            when airline_code = uid_airline_code
             and to_char(local_departure, 'YYYYMMDD') = uid_flight_date
             and departure_airport = uid_departure_airport
             and arrival_airport = uid_arrival_airport
            then true
            else false
        end as is_consistent
    from info_parts
)

select 
    count(*) as vuelos_no_consistentes
from consistency_check
where is_consistent = false;
```

### 3. Usando la tabla airlines, muestra el nombre de la aerolínea y cuántos vuelos no consistentes tiene.
```sql
with last_flights_status as (
    with base as (
        select 
            flight_row_id,
            unique_identifier,
            airline_code,
            departure_airport,
            arrival_airport,
            local_departure
        from flights
    ),
    final as (
        select 
            flight_row_id,
            unique_identifier,
            airline_code,
            departure_airport,
            arrival_airport,
            local_departure,
            row_number() over(
                partition by unique_identifier
                order by flight_row_id desc
            ) as rn
        from base
    )
    select 
        flight_row_id,
        unique_identifier,
        airline_code,
        departure_airport,
        arrival_airport,
        local_departure
    from final
    where rn = 1
),
info_parts as (
    select 
        flight_row_id,
        unique_identifier,
        airline_code,
        departure_airport,
        arrival_airport,
        local_departure,
        split_part(unique_identifier, '-', 1) as uid_airline_code,
        split_part(unique_identifier, '-', 3) as uid_flight_date,
        split_part(unique_identifier, '-', 4) as uid_departure_airport,
        split_part(unique_identifier, '-', 5) as uid_arrival_airport
    from last_flights_status
),
consistency_check as (
    select 
        flight_row_id,
        unique_identifier,
        airline_code,
        case
            when airline_code = uid_airline_code
             and to_char(local_departure, 'YYYYMMDD') = uid_flight_date
             and departure_airport = uid_departure_airport
             and arrival_airport = uid_arrival_airport
            then true
            else false
        end as is_consistent
    from info_parts
)
select 
    a.name as airline_name,
    count(*) as vuelos_no_consistentes
from consistency_check c
left join airlines a
    on c.airline_code = a.airline_code
where c.is_consistent = false
group by 1
order by vuelos_no_consistentes desc;
```








