
Extract all the data from exam_answers that had test durations that were greater than the average duration for the "Yawn Warm-Up" game 


```python
%%sql
SELECT *
FROM exam_answers 
WHERE TIMESTAMPDIFF(minute,start_time,end_time) >
     (SELECT AVG(TIMESTAMPDIFF(minute,start_time,end_time)) AS AvgDuration
      FROM exam_answers
      WHERE test_name='Yawn Warm-Up' AND TIMESTAMPDIFF(minute,start_time,end_time)>0); 
```

Use an IN operator to determine how many entries in the exam_answers tables are from the "Puzzles", "Numerosity", or "Bark Game" tests.


```python
%%sql
SELECT *
FROM exam_answers 
WHERE subcategory_name IN ('puzzles','Numerosity','Bark Game');
```

Use a NOT IN operator to determine how many unique dogs in the dog table are NOT in the "Working", "Sporting", or "Herding" breeding groups. 


```python
%%sql
SELECT COUNT(*)
FROM dogs
WHERE breed_group NOT IN ('working','sporting','herding');
```

## EXISTS/NOT EXISTS statements return a value of TRUE or FALSE


```python
#determine the number of unique users in the users table who were NOT in the dogs table using a NOT EXISTS clause
%%sql
SELECT DISTINCT u.user_guid AS uUserID
FROM users u
WHERE NOT EXISTS (SELECT *
              FROM dogs d 
              WHERE u.user_guid =d.user_guid);
```

To complete the join on ONLY distinct UserIDs from the users table:


```python
#create a temporary table: derived table, name it
%%sql
SELECT DistinctUUsersID.user_guid AS uUserID, d.user_guid AS dUserID, count(*) AS numrows
FROM (SELECT DISTINCT u.user_guid 
      FROM users u) AS DistinctUUsersID 
LEFT JOIN dogs d
   ON DistinctUUsersID.user_guid=d.user_guid
GROUP BY DistinctUUsersID.user_guid
ORDER BY numrows DESC
```

 Write a query using an IN clause and equijoin syntax that outputs the dog_guid, breed group, state of the owner, and zip of the owner for each distinct dog in the Working, Sporting, and Herding breed groups. 


```python
%%sql
SELECT DISTINCT d.dog_guid, d.breed_group, u.state, u.zip
FROM dogs d left join users u
ON   d.user_guid=u.user_guid
WHERE d.breed_group IN ('working','sporting','herding');
```

Use a NOT EXISTS clause to examine all the users in the dogs table that are not in the users table


```python
%%sql
SELECT d.user_guid AS dUserID, d.dog_guid AS dDogID
FROM dogs d
WHERE NOT EXISTS (SELECT DISTINCT u.user_guid
FROM users u
WHERE d.user_guid =u.user_guid);
```

LEFT JOIN distinct user(s) from the user table whose user_guid='ce7b75bc-7144-11e5-ba71-058fbc01cf0b'. 


```python
%%sql
SELECT DUID.user_guid, d.user_guid AS dUserID, count(*) AS numrows
FROM (SELECT DISTINCT u.user_guid
FROM users u
WHERE u.user_guid='ce7b75bc-7144-11e5-ba71-058fbc01cf0b') AS DUID
LEFT JOIN dogs d
ON DUID.user_guid=d.user_guid
GROUP BY DUID.user_guid
ORDER BY numrows DESC;
#output: 1819
```

Now let's prepare and test the inner query for the right half of the join. Give the dogs table an alias, and write a query that would select the distinct user_guids from the dogs table (we will use this query as a inner subquery in subsequent questions, so you will need an alias to differentiate the user_guid column of the dogs table from the user_guid column of the users table).


```python
%%sql
SELECT DUID.user_guid, dogID.user_guid AS dUserID, count(*) AS numrows
FROM (SELECT DISTINCT u.user_guid
FROM users u
WHERE u.user_guid='ce7b75bc-7144-11e5-ba71-058fbc01cf0b') AS DUID
LEFT JOIN (SELECT DISTINCT d.user_guid
FROM dogs d
WHERE d.user_guid='ce7b75bc-7144-11e5-ba71-058fbc01cf0b') AS dogID
ON DUID.user_guid=dogID.user_guid
GROUP BY DUID.user_guid
ORDER BY numrows DESC;
#output:1
```

retrieve a full list of all the DogIDs a user in the users table owns, with its accompagnying breed information whenever possible. HOWEVER, BEFORE YOU RUN THE QUERY MAKE SURE TO LIMIT YOUR OUTPUT TO 100 ROWS WITHIN THE SUBQUERY TO THE LEFT OF YOUR JOIN. 


```python
%%sql
SELECT DistinctUUsersID.user_guid AS uUserID, DistictDUsersID.user_guid AS
dUserID,
DistictDUsersID.dog_guid AS DogID, DistictDUsersID.breed AS breed
FROM (SELECT DISTINCT u.user_guid
FROM users u
LIMIT 100) AS DistinctUUsersID
LEFT JOIN (SELECT DISTINCT d.user_guid, d.dog_guid, d.breed
FROM dogs d) AS DistictDUsersID
ON DistinctUUsersID.user_guid=DistictDUsersID.user_guid
GROUP BY DistinctUUsersID.user_guid;
```

Add dog breed and dog weight to the columns that will be included in the final output of your query. In addition, use a HAVING clause to include only UserIDs who would have more than 10 rows in the output of the left join (your output should contain 5 rows).


```python
%%sql
SELECT DistictUUsersID.user_guid AS userid, d.breed, d.weight, count(*) AS numrows
FROM (SELECT DISTINCT u.user_guid
FROM users u) AS DistictUUsersID
LEFT JOIN dogs d
ON DistictUUsersID.user_guid=d.user_guid
GROUP BY DistictUUsersID.user_guid
HAVING numrows>10
ORDER BY numrows DESC;
```


```python
userid	breed	weight	numrows
ce7b75bc-7144-11e5-ba71-058fbc01cf0b	Shih Tzu	190	1819
ce225842-7144-11e5-ba71-058fbc01cf0b	Shih Tzu	190	26
ce2258a6-7144-11e5-ba71-058fbc01cf0b	Shih Tzu	190	20
ce135e14-7144-11e5-ba71-058fbc01cf0b	Shih Tzu	190	13
ce29675e-7144-11e5-ba71-058fbc01cf0b	Labrador Retriever- Mix	60	11

```

You can see that almost all of the UserIDs that are causing problems are Shih Tzus that weigh 190 pounds. As we learned in earlier lessons, Dognition used this combination of breed and weight to code for testing accounts. These UserIDs do not represent real data. These types of testing entries would likely be cleaned out of databases used in large established companies, but could certainly still be present in either new databases that are still being prepared and configured, or in small companies which have not had time or resources to perfect their data storage.
