
To join the tables, use a WHERE clause and add a couple of details to the FROM clause so that the database knows from what table each field in SELECT clause comes.


# INNER JOIN


```python
#Join two tables; use WHERE to tell how to join
%%sql
SELECT d.dog_guid AS DogID, d.user_guid AS UserID, AVG(r.rating) AS AvgRating, 
       COUNT(r.rating) AS NumRatings, d.breed, d.breed_group, d.breed_type
FROM dogs d, reviews r
WHERE d.dog_guid=r.dog_guid AND d.user_guid=r.user_guid
GROUP BY UserID, DogID, d.breed, d.breed_group, d.breed_type
HAVING NumRatings >= 10
ORDER BY AvgRating DESC
LIMIT 200
```

### How would you extract the user_guid, dog_guid, breed, breed_type, and breed_group for all animals who completed the "Yawn Warm-up" game


```python
%%sql
SELECT d.user_guid,d.dog_guid, d.breed, d.breed_type, d.breed_group,c.test_name
FROM dogs d, complete_tests c
WHERE d.dog_guid=c.dog_guid and c.test_name='Yawn Warm-up' ;
```


```python
#Join more than two tables
%%sql
SELECT DISTINCT d.user_guid,u.membership_type, d.dog_guid,d.breed
FROM dogs d, users u, complete_tests c
WHERE d.dog_guid=c.dog_guid and d.user_guid=u.user_guid and d.breed='golden retriever';
```


```python
#How many unique Golden Retrievers who live in North Carolina are there in the Dognition database
%%sql
SELECT DISTINCT d.breed,d.user_guid,u.state, d.dog_guid
FROM dogs d, users u
WHERE d.user_guid=u.user_guid and u.state='NC'and d.breed='golden retriever';
```


```python
#How many unique customers within each membership type provided reviews
%%sql
SELECT u.membership_type,u.user_guid, r.rating,COUNT(DISTINCT r.user_guid) 
FROM reviews r, users u
WHERE u.user_guid=r.user_guid and r.rating IS NOT NULL
GROUP BY membership_type;

```


```python
#For which 3 dog breeds do we have the greatest amount of site_activity data, (as defined by non-NULL values in script_detail_id)
%%sql
SELECT d.breed, COUNT(s.script_detail_id) 
FROM site_activity s, dogs d
WHERE d.dog_guid=s.dog_guid and s.script_detail_id IS NOT NULL
GROUP BY d.breed;
```


```python
#Join two tables using "join...on..."
%%sql
SELECT d.user_guid AS UserID, d.dog_guid AS DogID, 
       d.breed, d.breed_type, d.breed_group
FROM dogs d JOIN complete_tests c
ON d.dog_guid=c.dog_guid
WHERE test_name='Yawn Warm-up';
```

# outer JOIN


```python
#query a list of only the dog_guids that were NOT in the dogs table:
%%sql
SELECT r.dog_guid AS rDogID, d.dog_guid AS dDogID, r.user_guid AS rUserID, d.user_guid AS dUserID, AVG(r.rating) AS AvgRating, COUNT(r.rating) AS NumRatings, d.breed, d.breed_group, d.breed_type
FROM reviews r LEFT JOIN dogs d
  ON r.dog_guid=d.dog_guid AND r.user_guid=d.user_guid
WHERE d.dog_guid IS NULL
GROUP BY r.dog_guid
HAVING NumRatings >= 10
ORDER BY AvgRating DESC;
```

How would you use a left join to retrieve a list of all the unique dogs in the dogs table, and retrieve a count of how many tests each one completed? Include the dog_guids and user_guids from the dogs and complete_tests tables in your output. 


```python
%%sql
SELECT DISTINCT d.dog_guid AS ddogid,d.user_guid, c.dog_guid AS cdogid, c.user_guid, COUNT(test_name)
FROM dogs d LEFT JOIN complete_tests c
ON d.dog_guid=c.dog_guid AND d.user_guid =c.user_guid 
GROUP BY ddogid;
```

### COUNT DISTINCT does NOT count NULL values, while SELECT/GROUP BY clauses roll up NULL values into one group. If you want to infer the number of distinct entries from the results of a query using joins and GROUP BY clauses, remember to include an "IS NOT NULL" clause to ensure you are not counting NULL values.

How would you write a query that used a left join to return the number of distinct user_guids that were in the users table, but not the dogs table (your query should return a value of 2226)?


```python
%%sql
SELECT d.user_guid, COUNT(DISTINCT u.user_guid)
FROM users u LEFT JOIN dogs d
ON u.user_guid=d.user_guid
WHERE d.user_guid IS NULL;
```

Use a left join to create a list of all the unique dog_guids that are contained in the site_activities table, but not the dogs table, and how many times each one is entered. Note that there are a lot of NULL values in the dog_guid of the site_activities table, so you will want to exclude them from your list. (Hint: if you exclude null values, the results you get will have two rows with words in their site_activities dog_guid fields instead of real guids, due to mistaken entries)


```python
%%sql
SELECT s.dog_guid AS SA_dogs_not_present_in_dogs_table, COUNT(*) AS
NumEntries
FROM site_activities s LEFT JOIN dogs d
ON s.dog_guid=d.dog_guid
WHERE d.dog_guid IS NULL AND s.dog_guid IS NOT NULL
GROUP BY SA_dogs_not_present_in_dogs_table;
```
