# SQL Murder Mystery Solution

Author: teresaandrea02

Case Source: https://mystery.knightlab.com by The Northwestern University Knight Lab

___

This report documents the technical methodology used to solve the "SQL Murder Mystery." The objective was to leverage SQL querying logic to identify a suspect by connecting fragmented data points across multiple relational tables.

Brief:

*A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​murder​ that occurred sometime on ​Jan.15, 2018​ and that it took place in ​SQL City​. Start by retrieving the corresponding crime scene report from the police department’s database.*

To understand the data relationships, I utilized the following Entity Relationship Diagram (ERD) as a primary reference for table joins
<img width="815" height="399" alt="image" src="https://github.com/user-attachments/assets/0a9d0c02-7615-4331-aa81-66994ce8ef9a" />
(Knight Lab)

# 1. Initial Discovery
The investigation began by analyzing the `crime_scene_report` table to understand the data format and extract specific details regarding the incident. 
```
select * from crime_scene_report limit 3
```
|  date  |  type	|  description	city  |
|---|---|---|
|20180115|robbery|	A Man Dressed as Spider-Man Is on a Robbery Spree	NYC|
|20180115|murder|Life? Dont talk to me about life.	Albany|
|20180115|murder|Mama, I killed a man, put a gun against his head...	Reno|

From this table, I can identify the date format. I see the first 4 digits are always the same, so it must be the year number. Continue with four more numbers. The last two numbers are 15. There can't be a fifteenth month. So, the last two numbers are the date. The middle two numbers are the month. So the date format is `YYYY/MM/DD`.

Using this, I filtered for the specific murder case in SQL City:
```
select * from crime_scene_report
where date = '20180115' and city = 'SQL City' and type='murder'
```
**Always remember that SQL is case sensitive. So take attention to the uppercase and lowercase** If you want all lower, using lower() syntax.
|date|type|description|city|
|---|---|---|---|
|20180115|murder|Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".|SQL City|

# 2. Witness Interview

So, I have a witness. That's a good track. From the ERD, data that have address is on `person` table. But I want to confirm whether this data contains a last name or not. Judging from the crime description, there is a witness named 'Annabel'.
```
select * from person limit 3
```
|id|name|license_id|address_number|address_street_name|ssn|
|---|---|---|---|---|---|
|10000|Christoper Peteuil|993845|624|Bankhall Ave|747714076|
|10007|Kourtney Calderwood|861794|2791|Gustavus Blvd|477972044|
|10010|Muoi Cary|385336|741|Northwestern Dr|828638512|

And they have a last name. So I got 2 key pieces of information about this witness: 
1. The first witness lives in the corner-end of Northwestern Dr Street. So, I can assume the last house is the biggest house number.
2. The second witness, Annabel *something*, lives in Franklin Ave Street somewhere.

Upon further inspection of the Entity Relationship Diagram (ERD), a clear relational link was identified between the `person` and `interview` tables. The `interview` table utilizes a `person_id` column (Integer) which serves as a Foreign Key referencing the id column (Primary Key) in the `person` table.

By aligning these unique identifiers, I was able to perform a JOIN operation to correlate personal details with their respective interview transcripts. This logical connection is essential for consolidating witness testimonies into a single, actionable dataset. Consequently, I executed the following syntax to retrieve the necessary evidence:
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
|id|name|address_number|address_street_name|transcript|
|---|---|---|---|---|
|14887|Morty Schapiro|4919|Northwestern Dr|I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".|
|16371|Annabel Miller|103|Franklin Ave|I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.|

Critical Evidence Extracted:
1. Physical Description: The suspect is male.
2. Gym Membership: A Gold member of "Get Fit Now Gym" with an ID starting with "48Z".
3. Vehicle Details: The car license plate contains the sequence "H42W".
4. Alibi/Timeline: The suspect was spotted at the gym on January 9, 2018.

# 3. Identifying the Suspect

By synthesizing the evidence, I constructed a multi-table join to isolate the individual who matched all criteria.
```
select checkin.membership_id,person.id,checkin.check_in_date,member.name,license.gender,person.license_id,license.plate_number
from get_fit_now_check_in as checkin
join get_fit_now_member as member on checkin.membership_id=member.id
join person on member.person_id=person.id
join drivers_license as license on person.license_id=license.id
where checkin.check_in_date='20180109' and member.id like '48Z%'
and member.membership_status='gold' and license.plate_number like '%H42W%'
```
|membership_id|id|check_in_date|name|gender|license_id|plate_number|
|---|---|---|---|---|---|---|
|48Z55|67318|20180109|Jeremy Bowers|male|423327|0H42W2|

# Conclusion
After verifying the suspect in the solution schema:
```
INSERT INTO solution VALUES (1, 'Jeremy Bowers');
        
        SELECT value FROM solution;
```
Output:

*Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime. If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries. Use this same INSERT statement with your new suspect to check your answer.*

***

Status: Case Closed. Next objective: Identifying the mastermind behind the crime...
