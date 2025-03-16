
```sql
SELECT *
FROM crime_scene_report
WHERE type = 'murder' and city = 'SQL City' and date = '20180115'
```
Security footage shows that there were 2 witnesses.  
The first witness lives at the last house on "Northwestern Dr".  
The second witness, named Annabel, lives somewhere on "Franklin Ave".  
Записи с камер наблюдения показывают, что было 2 свидетеля.  
Первый свидетель живет в последнем доме на "Северо-Западной улице".  
Вторая свидетельница, по имени Аннабель, живет где-то на "Франклин авеню".  

```sql
SELECT *
FROM person
WHERE address_street_name = 'Northwestern Dr' 
	  AND address_number IN (
	  	SELECT MAX(address_number)
		  FROM person
		  WHERE address_street_name = 'Northwestern Dr' 
	  )

SELECT *
FROM person
WHERE name LIKE '%Annabel%' AND address_street_name = 'Franklin Ave'        
```

14887	Morty Schapiro	118009	4919	Northwestern Dr	111564949  
16371	Annabel Miller	490173	103	Franklin Ave	318771143  

```sql
SELECT *
FROM interview
WHERE person_id IN (16371, 14887)
```
14887	I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag.  
The membership number on the bag started with "48Z". Only gold members have those bags.  
The man got into a car with a plate that included "H42W".  

16371	I saw the murder happen, and I recognized the killer from my gym when I was working out  
last week on January the 9th.  

```sql
SELECT p.id, p.name
FROM person p
JOIN 
(SELECT id, age, gender, plate_number
FROM drivers_license
WHERE plate_number LIKE '%H42W%' and gender = 'male') as t2
ON p.license_id = t2.id
```
51739 Tushar Chandra  
67318 Jeremy Bowers  

```sql
SELECT check_in_date, id, person_id
FROM get_fit_now_check_in getcheck 
JOIN get_fit_now_member getnow ON getcheck.membership_id = getnow.id
WHERE check_in_date = '20180109' and membership_status = 'gold' 
	    and id LIKE '48Z%'
```
48Z7A 28819  
48Z55 67318  

Is it Jeremy?  

```sql
INSERT INTO solution VALUES (1, 'Jeremy Bowers');
SELECT value FROM solution;
```
Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge,  
try querying the interview transcript of the murderer to find the real villain behind this crime.  

```sql
SELECT *
FROM interview
WHERE person_id = 67318
```

I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67").  
She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.  

```sql
SELECT p.id, name
FROM person p 
JOIN drivers_license dl ON p.license_id = dl.id
JOIN facebook_event_checkin fec on p.id = person_id
WHERE hair_color = 'red' and height > 65 and height < 67
	  and car_make = 'Tesla' and car_model = 'Model S'
	  and event_name LIKE 'SQL Symphony%' and date LIKE '201712%'
```
Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time.  
Time to break out the champagne!  
