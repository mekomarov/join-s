# join-s
Задание Join`s

````
--Задание 1 Реализовать прямое соединение двух или более таблиц
/*
 * По номеру билета узнал информацию о классе обслуживания во вроемя полета, о датах,
 * аэропортах, времени вылета и прибытия, а так же стоимости билетов 
 */
select tf.ticket_no, 
	   tf.fare_conditions,
	   f.status,
	   f.departure_airport,
	   f.arrival_airport,
	   f.scheduled_departure,
	   f.scheduled_arrival,
	   tf.amount 
from bookings.ticket_flights as  tf 
join bookings.flights as f on f.flight_id = tf.flight_id  
where tf.ticket_no = '0005434704510';

--Задание 2 Реализовать левостороннее соединение двух или более таблиц
/*
 * Узнаю все запланированные вылеты боинга 777-300, и проверить 
 * нет ли по ним уже проставленных даты и времени фактического вылета или прибытия (Например таким образом узнать что в данных ошибка)
 */

select ad.model,
	   f.scheduled_departure,
	   f.scheduled_arrival,
	   f.actual_departure,
	   f.actual_arrival 
from bookings.aircrafts_data as ad
left join bookings.flights as f on f.aircraft_code = ad.aircraft_code 
where ad.aircraft_code = '773'
and f.status = 'Scheduled'
and (f.actual_departure is not null or f.actual_arrival is not null);

select ad.model,
	   f.scheduled_departure,
	   f.scheduled_arrival,
	   f.actual_departure,
	   f.actual_arrival 
from bookings.aircrafts_data as ad
left join bookings.flights as f on f.aircraft_code = ad.aircraft_code 
where ad.aircraft_code = '773'
and f.status = 'Scheduled'
and (f.actual_departure is not null or f.actual_arrival is not null);

--Задание 3 Реализовать кросс соединение двух таблиц

 select ad.aircraft_code,
 		ad.model,
 		ad."range",
 		s.seat_no,
 		s.fare_conditions 
 from bookings.aircrafts_data as ad
 cross join bookings.seats as s
 where ad.aircraft_code = '773'


--Задание 4 Реализовать полное соединение таблиц
 /*
  * Номера билетов с контактной информацией по пассажирам на которых не были выданы посадочные талоны
  */
select t.ticket_no,
	   t.passenger_name,
	   t.contact_data,
	   bp.boarding_no,
	   bp.seat_no
from bookings.tickets as t
full join bookings.boarding_passes as bp on bp.ticket_no = t.ticket_no 
where bp.seat_no is null 

--Задание 5 Написать запрос в котором будут реализованы разные типы соединения
/*
 * Вывести информацию о пассажирах которые вылетают из аэропорта VKO с классом обслуживания комфорт ближайшим запланированным рейсом  
 */

select t.ticket_no,
	   t.passenger_name,
	   t.contact_data,
	   tf.fare_conditions,
	   tf.amount,
	   f.departure_airport
from bookings.tickets as t
join bookings.ticket_flights as tf on tf.ticket_no = t.ticket_no 
left join lateral (select f.departure_airport 
				   from bookings.flights as f 
				   where f.flight_id = tf.flight_id 
				   order by f.scheduled_departure desc limit 1) as f on true
where tf.fare_conditions = 'Comfort'
and f.departure_airport = 'VKO';


--Структуры таблиц 
CREATE TABLE bookings.ticket_flights (
	ticket_no bpchar(13) NOT NULL,
	flight_id int4 NOT NULL,
	fare_conditions varchar(10) NOT NULL,
	amount numeric(10, 2) NOT NULL
);

CREATE TABLE bookings.flights (
	flight_id serial4 NOT NULL,
	flight_no bpchar(6) NOT NULL,
	scheduled_departure timestamptz NOT NULL,
	scheduled_arrival timestamptz NOT NULL,
	departure_airport bpchar(3) NOT NULL,
	arrival_airport bpchar(3) NOT NULL,
	status varchar(20) NOT NULL,
	aircraft_code bpchar(3) NOT NULL,
	actual_departure timestamptz NULL,
	actual_arrival timestamptz NULL
);

CREATE TABLE bookings.aircrafts_data (
	aircraft_code bpchar(3) NOT NULL,
	model jsonb NOT NULL,
	"range" int4 NOT NULL
);

CREATE TABLE bookings.tickets (
	ticket_no bpchar(13) NOT NULL,
	book_ref bpchar(6) NOT NULL,
	passenger_id varchar(20) NOT NULL,
	passenger_name text NOT NULL,
	contact_data jsonb NULL
);

CREATE TABLE bookings.aircrafts_data (
	aircraft_code bpchar(3) NOT NULL,
	model jsonb NOT NULL,
	"range" int4 NOT NULL
);

CREATE TABLE bookings.boarding_passes (
	ticket_no bpchar(13) NOT NULL,
	flight_id int4 NOT NULL,
	boarding_no int4 NOT NULL,
	seat_no varchar(4) NOT NULL
);




