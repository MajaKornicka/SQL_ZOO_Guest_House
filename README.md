# SQL ZOO - Guest House 🏠

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



