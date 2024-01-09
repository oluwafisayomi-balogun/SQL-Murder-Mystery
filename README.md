# SQL-Murder-Mystery
A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​murder​ that occurred sometime on ​Jan.15, 2018,​ and that it took place in ​SQL City. 


**Retrieving the corresponding crime scene report from the police department’s database.**
`SELECT * 
	FROM crime_scene_report
	WHERE city LIKE '%SQL City%' 
	AND type = 'murder'
	AND date = '20180115'
;
`
-- Security footage shows that there were 2 witnesses. 
-- The first witness lives at the last house on "Northwestern Dr". 
-- The second witness, named Annabel, lives somewhere on "Franklin Ave".

*******************************************************************************************************************

-- First Witness
SELECT * 
	FROM person
	WHERE address_street_name LIKE '%Northwestern Dr%' 
	ORDER BY address_number DESC
	LIMIT 1
;
-- RESULT
-- 1st Witness is 'Morty Schapiro' with id - '14887', license_id - '118009', ssn - 111564949, 
-- address_street_name - 'Northwestern Dr' and address_number - '4919'
	
	
-- Second Witness
SELECT * 
	FROM person
	WHERE address_street_name LIKE '%Franklin Ave%'
	AND name LIKE '%Annabel%'
;
-- RESULT
-- 2nd Witness is 'Annabel Miller' with id - '16371', license_id - '490173', ssn - 318771143, 
-- address_street_name - 'Franklin Ave' and address_number - '103

***********************************************************************************************************

SELECT i.* 
	FROM interview i
	INNER JOIN person p
	ON i.person_id = p.id
	WHERE p.id = '14887' or p.id = '16371'
;
-- 1st Witness Transcript
-- I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. 
-- The membership number on the bag started with "48Z". Only gold members have those bags. 
-- The man got into a car with a plate that included "H42W".

-- 2nd Witness Transcript
-- I saw the murder happen, and I recognized the killer from my gym 
-- when I was working out last week on January the 9th.

**************************************************************************************************************

CREATE VIEW [Suspects] AS
SELECT g1.*, g2.*
	FROM get_fit_now_check_in g1
	JOIN get_fit_now_member g2
	ON g1.membership_id = g2.id 
	WHERE g2.membership_status = 'gold'
	AND g1.membership_id LIKE '48Z%'
	AND check_in_date = '20180109'
;

-- We currently have 2 suspects
-- Suspect 1:
-- membership_id = 48Z7A; check_in_date = 20180109; check_in_time = 1600; check_out_time =	1730	
-- id = 48Z7A; person_id = 28819; name = Joe Germuska; membership_start_date = 20160305; membership_status = gold

-- Suspect 2:
-- membership_id = 48Z55; check_in_date = 20180109; check_in_time = 1530;	check_out_time = 1700;
-- id = 48Z55;	 person_id = 67318; name = Jeremy Bowers;  membership_start_date = 20160101; membership_status = gold	

**************************************************************************************************************	

CREATE VIEW [Suspects2] AS
SELECT p.id, p.name, d.plate_number 
	FROM drivers_license d
	JOIN person p
	ON p.license_id = d.id
	WHERE d.plate_number LIKE '%H42W%'
;

***************************************************************************************************************

-- Find row(s) that appear(s) in both views	
SELECT *
	FROM Suspects s1
	JOIN Suspects2 s2
	ON s1.person_id = s2.id
;

*************************************************************************************************************

SELECT * 
	FROM interview
	WHERE person_id = 67318
;
-- RESULT
-- I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). 
-- She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.

***************************************************************************************************************

SELECT p.*, d.*
	FROM person p
	JOIN drivers_license d
	ON p.license_id = d.id
	WHERE d.gender = 'female'
	AND d.height BETWEEN 65 and 67
	AND d.hair_color = 'red'
	AND d.car_make = 'Tesla'
	AND p.id IN
		(SELECT f.person_id
			FROM facebook_event_checkin f
			WHERE f.event_name = 'SQL Symphony Concert'
			AND f.date LIKE '201712%'
			GROUP BY f.person_id
			HAVING count(person_id) = 3
		)
;
-- RESULT
-- id - 99716; name - Miranda Priestly; license_id - 202298

******************************************************************************************************************
SELECT * 
	FROM interview 
	WHERE person_id = 99716
;
