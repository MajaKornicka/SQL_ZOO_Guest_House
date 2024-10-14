# SQL ZOO - Guest House 🏠

Data and Questions Source: https://sqlzoo.net/wiki/Guest_House

Solutions written using MySQL



![image](https://github.com/user-attachments/assets/78ec586e-7a06-42f5-badd-d767f85d8273)

 <br>
 <br>
 
### Medium Problems

 <br>

6. Ruth Cadbury. Show the total amount payable by guest Ruth Cadbury for her room bookings. You should JOIN to the rate table using room_type_requested and occupants.

```sql
SELECT SUM(nights*amount) as total_amount
FROM booking
JOIN rate ON booking.room_type_Requested = rate.room_type 
AND booking.occupants = rate.occupancy
WHERE guest_id = (
                  SELECT id
                  FROM guest
                  WHERE first_name = 'Ruth' 
                  AND last_name = 'Cadbury')
```
Result

|total_amount|
|----|
|552.00|

 <br>
  <br>



7. Including Extras. Calculate the total bill for booking 5346 including extras.

 ``` sql
SELECT  (amount*nights) + (SELECT SUM(amount)
                           FROM extra
                           WHERE booking_id = 5346 ) as total_amount
FROM booking
JOIN rate ON booking.room_type_Requested = rate.room_type 
AND booking.occupants = rate.occupancy
WHERE booking.booking_id = 5346
```

Result
|total_amount|
|----|
|118.56|

 <br>
  <br>

8. Edinburgh Residents. For every guest who has the word “Edinburgh” in their address show the total number of nights booked. Be sure to include 0 for those guests who have never had a booking. Show last name, first name, address and number of nights. Order by last name then first name.

```sql
SELECT last_name, 
       first_name, 
       address, 
       CASE 
           WHEN SUM(nights) IS NULL THEN 0
           ELSE SUM(nights) 
       END as nights
FROM guest
LEFT JOIN booking ON guest.id = booking.guest_id
WHERE address LIKE '%Edinburgh%'
GROUP BY 1,2,3
ORDER BY 1,2
```


Result
|last_name|first_name|address|nights|
|--------|------------|-------|-------|
|Brock	|Deidre	|Edinburgh North and Leith	|0|
|Cherry|	Joanna	|Edinburgh South West	|0|
|Murray|	Ian|	Edinburgh South	|13|
|Sheppard|	Tommy|	Edinburgh East	|0|
|Thomson|	Michelle|	Edinburgh West	|3|


 <br>
  <br>

9. How busy are we? For each day of the week beginning 2016-11-25 show the number of bookings starting that day. Be sure to show all the days of the week in the correct order.

```sql
SELECT DATE_FORMAT(booking_date, "%Y-%m-%d") as booking_date, COUNT(arrival_time) as arrivals
FROM booking
WHERE booking_date BETWEEN '2016-11-25' AND DATE_ADD('2016-11-25', INTERVAL 6 DAY)
GROUP BY 1
ORDER BY 1
```


Results

|booking_date|arrivals|
|------------|--------|
|2016-11-25	|7|
|2016-11-26	|8|
|2016-11-27|	12|
|2016-11-28	|7|
|2016-11-29	|13|
|2016-11-30	|6|
|2016-12-01	|7|

 <br>
  <br>


10. How many guests? Show the number of guests in the hotel on the night of 2016-11-21. Include all occupants who checked in that day but not those who checked out.

```sql
WITH checked_in_guests AS (
SELECT occupants, nights,
       DATE_FORMAT(booking_date, "%Y-%m-%d") as check_in, 
       DATE_FORMAT(DATE_ADD(booking_date, INTERVAL (nights) DAY), "%Y-%m-%d") as check_out
FROM booking)

SELECT SUM(occupants) as total_guests
FROM checked_in_guests
WHERE check_in <= "2016-11-21"
AND check_out > "2016-11-21"
```


Results

|total_guests|
|------------|
|39|

 <br>
  <br>


### Hard Problems

 <br>
11. Coincidence. Have two guests with the same surname ever stayed in the hotel on the evening? Show the last name and both first names. Do not include duplicates.


```sql
WITH dates AS (
SELECT DATE_FORMAT(booking_date, "%Y-%m-%d") as check_in, 
       DATE_FORMAT(DATE_ADD(booking_date, INTERVAL (nights) DAY), "%Y-%m-%d") as check_out,
       guest_id,
       last_name,
       first_name
FROM booking
JOIN guest ON guest.id = booking.guest_id),

names AS (
SELECT d1.last_name as d1_last, d1.first_name AS d1_first, d2.last_name AS d2_last, d2.first_name as d2_first
FROM dates d1
JOIN dates d2 ON d2.check_in >= d1.check_in AND d2.check_in < d1.check_out
WHERE d1.last_name = d2.last_name
AND d1.guest_id <> d2.guest_id
)

SELECT distinct d1_last as last_name, d1_first as first_name, d2_first as first_name_2
FROM names
WHERE d1_first > d2_first
ORDER BY 1
````

Results


|last_name	|first_name	|first_name_2|
|--------|---------|----------|
|Davies|	Philip	|David T. C.|
|Evans|	Mr Nigel	|Graham|
|Howarth	|Sir Gerald	|Mr George|
|Jones|	Susan Elan	|Mr Marcus|
|Lewis	|Dr Julian	|Clive|
|McDonnell|	John	|Dr Alasdair|


 <br>
  <br>

  12. Check out per floor. The first digit of the room number indicates the floor – e.g. room 201 is on the 2nd floor. For each day of the week beginning 2016-11-14 show how many rooms are being vacated that day by floor number. Show all days in the correct order.

```sql
WITH basic AS (
SELECT DATE_FORMAT(DATE_ADD(booking_date, INTERVAL (nights) DAY), "%Y-%m-%d") as check_out, 
       room_no
FROM booking)

SELECT check_out,
       SUM(CASE WHEN room_no LIKE '1%' THEN 1
           ELSE 0
           END) AS 1st,

       SUM(CASE WHEN room_no LIKE '2%' THEN 1
           ELSE 0
           END) AS 2nd,

       SUM(CASE WHEN room_no LIKE '3%' THEN 1
           ELSE 0
           END) AS 3rd
FROM basic
WHERE check_out >= "2016-11-14" AND check_out <= DATE_ADD("2016-11-14" , INTERVAL 6 DAY)
GROUP BY check_out
ORDER BY check_out

```

Results

|check_out|	1st	|2nd	|3rd|
|--------|-----|--------|----------|
|2016-11-14	|5|	3	|4|
|2016-11-15	|6	|4	|1|
|2016-11-16	|2	|2	|4|
|2016-11-17	|5	|3	|6|
|2016-11-18	|2	|3	|2|
|2016-11-19	|5	|5	|1|
|2016-11-20 |	2	|2	|2|

 <br>
  <br>

13. Free rooms? List the rooms that are free on the day 25th Nov 2016.

```sql
WITH basic AS (
SELECT DATE_FORMAT(booking_Date, "%Y-%m-%d") as check_in, 
       DATE_FORMAT(DATE_ADD(booking_date, INTERVAL (nights) DAY), "%Y-%m-%d") as check_out,
       room_no
FROM booking)

SELECT room_no 
FROM basic

EXCEPT

SELECT room_no
FROM basic
WHERE check_in <= "2016-11-25" AND check_out > "2016-11-25"
```

Results

|room_no|
|------------|
|207|
|210|
|304|
