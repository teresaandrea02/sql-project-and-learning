# SQL Muder Mystery Solution (by teresaandrea02)

**This study case it's from https://mystery.knightlab.com by The Northwestern University Knight Lab**

There's a description or prologue for this case.

*A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​murder​ that occurred sometime on ​Jan.15, 2018​ and that it took place in ​SQL City​. Start by retrieving the corresponding crime scene report from the police department’s database.*

There's an ERD. So it's very useful for this case. 
<img width="815" height="399" alt="image" src="https://github.com/user-attachments/assets/0a9d0c02-7615-4331-aa81-66994ce8ef9a" />

First, I must know what exactly this case going on, and I must know what their format for date. So I just pull 3 cases in table `crime-scene-report`'s table. 
```
select * from crime_scene_report limit 3
```
|  date  |  type	|  description	city  |
|---|---|---|
|20180115|robbery|	A Man Dressed as Spider-Man Is on a Robbery Spree	NYC|
|20180115|murder|Life? Dont talk to me about life.	Albany|
|20180115|murder|Mama, I killed a man, put a gun against his head...	Reno|

From this table, I can identify the date format. I see the first 4 digits are always the same, so it must be the year number. Continue with four more numbers. The last two numbers are 15. There can't be a fifteenth month. So, the last two numbers are the date. The middle two numbers are the month. So the date format is `YYYY/MM/DD`.

Now, I know the date format. I can start to find out what this case's going on. Based on the prolog says, we know the key point: Jan 15, 2018 (2018/01/15); SQL City; murder. I can run the query
```
select * from crime_scene_report
where date = '20180115' and city = 'SQL City' and type='murder'
```
**Always remember that SQL is case sensitive. So take attention to the uppercase and lowercase** If you want all lower, using lower() syntax.
|date|type|description|city|
|---|---|---|---|
|20180115|murder|Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".|SQL City|

So, I have a witness. That's a good track. From the ERD, data that have address is on `person` table. But I want to confirm whether this data contains a last name or not. Judging from the crime description, there is a witness named 'Annabel'. So I query the syntax
```
select * from person limit 3
```
|id|name|license_id|address_number|address_street_name|ssn|
|---|---|---|---|---|---|
|10000|Christoper Peteuil|993845|624|Bankhall Ave|747714076|
|10007|Kourtney Calderwood|861794|2791|Gustavus Blvd|477972044|
|10010|Muoi Cary|385336|741|Northwestern Dr|828638512|

And they have a last name. So I got 2 key pieces of information about this witness: 
1. The first witness lives in the corner-end of Northwestern Dr Street. So, I can assume the last house is the last house number.
2. The second witness, Annabel *something*, lives in Franklin Ave Street somewhere.

See the ERD back again. In the `interview` table, there's only `person_id` and `transcript`. I think I find a solution if I look more closely at the ERD. The data type for `person_id` is integer, meaning it only contains numbers. Did you find what I mean? Judging from the previous output I pulled from the `person` table, the ID is also a number. So, the `person_id` in the `interview` table and the `id` in the `person` table may be the same.
So I try to query this syntax
```
SELECT person.id, person.name, MAX(person.address_number) AS address_number, person.address_street_name, interview.transcript 
FROM person
JOIN interview ON person.id = interview.person_id
WHERE address_street_name = 'Northwestern Dr'

UNION ALL

SELECT person.id, person.name, person.address_number, person.address_street_name, interview.transcript
FROM person
JOIN interview ON person.id = interview.person_id
WHERE address_street_name = 'Franklin Ave' AND person.name LIKE 'Annabel%';
```

