
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
SELECT id, age, gender, plate_number
FROM drivers_license
WHERE plate_number LIKE '%H42W%' and gender = 'male'           
```
423327	30	male	0H42W2  
664760	21	male	4H42WR  

```sql
SELECT check_in_date, id, person_id
FROM get_fit_now_check_in getcheck 
JOIN get_fit_now_member getnow ON getcheck.membership_id = getnow.id
WHERE check_in_date = '20180109' and membership_status = 'gold' 
	    and id LIKE '48Z%'
```
20180109	48Z7A	28819
20180109	48Z55	67318

## Кто то врёт, ошибается заблуждается. Хм.  

```sql
```

```sql
```

```sql
```

```sql
```

```sql
```

```sql
```
