
# Summarizing the data

These are the five most common aggregate functions used to summarize information stored in tables: AVG(),COUNT()(works on any types of variables),MAX(),MIN(),SUM()

## The COUNT function


```python
#Count the number of breed in Table DOGS
%%sql
SELECT COUNT(breed)
FROM dogs
```


```python
#count all the unique values in a column
SELECT COUNT(DISTINCT breed)
FROM dogs
```

### Question 1: Try combining this query with a WHERE clause to find how many individual dogs completed tests after March 1, 2014 (the answer should be 13,289): 


```python
%%sql
SELECT COUNT(DISTINCT Dog_Guid)
FROM complete_tests
WHERE created_at>'2014-03-01'
```

When a column is included in a count function, null values are ignored in the count. When an asterisk is included in a count function, nulls are included in the count.

### Question 4: How many distinct dogs have an exclude flag in the dogs table (value will be "1")? (the answer should be 853)


```python
%%sql
SELECT COUNT(DISTINCT dog_guid)
FROM dogs
WHERE exclude=1
```

## The SUM function 

Conveniently, we can combine the SUM function with ISNULL to count exactly how many NULL values there are. 


```python
%%sql
SELECT SUM(ISNULL(exclude))
FROM dogs
```

## The AVG, MIN, and MAX Functions

### Question 5: What is the average, minimum, and maximum ratings given to "Memory versus Pointing" game? (Your answer should be 3.5584, 0, and 9, respectively)**


```python
#This is wrong because min,max and "-" cannot be used here
SELECT test_name, 
AVG(rating) AS ave-rating, 
MIN(rating) AS min-rating, 
MAX(rating) AS max-rating 
FROM reviews
WHERE test_name ="Memory versus Pointing";

#Correct answer
%%sql
SELECT test_name, 
AVG(rating) AS AVG_rating, 
MIN(rating) AS MIN_rating, 
MAX(rating) AS MAX_rating 
FROM reviews
WHERE test_name ="Memory versus Pointing";
```

### Question 6: How would you query how much time it took to complete each test provided in the exam_answers table, in minutes? Title the column that represents this data "Duration." Note that the exam_answers table has over 2 million rows, so if you don't limit your output, it will take longer than usual to run this query. (HINT: use the TIMESTAMPDIFF function described at: http://www.w3resource.com/mysql/date-and-time-functions/date-and-time-functions.php. )

%%sql
SELECT test_name, TIMESTAMPDIFF(MINUTE, start_time,end_time) AS duration
FROM exam_answers
LIMIT 100;

### Question 7: Include a column for Dog_Guid, start_time, and end_time in your query, and examine the output. Do you notice anything strange?


```python
%%sql
SELECT test_name, dog_guid, TIMESTAMPDIFF(MINUTE, start_time,end_time) AS duration
FROM exam_answers
ORDER BY duration ASC 
LIMIT 300;
```

### Question 8: What is the average amount of time it took customers to complete all of the tests in the exam_answers table, if you do not exclude any data (the answer will be approximately 587 minutes)?


```python
%%sql
SELECT AVG(TIMESTAMPDIFF(MINUTE, start_time,end_time)) as duration
FROM exam_answers
LIMIT 200;
```

### Question 9: What is the average amount of time it took customers to complete the "Treat Warm-Up" test, according to the exam_answers table (about 165 minutes, if no data is excluded)?


```python
%%sql
SELECT AVG(TIMESTAMPDIFF(minute,start_time,end_time))
FROM exam_answers
WHERE test_name="Treat Warm-Up";
```

### Question 10: How many possible test names are there in the exam_answers table?


```python
%%sql
SELECT COUNT(DISTINCT test_name)
FROM exam_answers;
```

### Question 11: What is the minimum and maximum value in the Duration column of your query that included the data from the entire table?


```python
%%sql
SELECT MIN(TIMESTAMPDIFF(MINUTE, start_time,end_time)) as MIN_duration,
MAX(TIMESTAMPDIFF(MINUTE, start_time,end_time)) as MAX_duration
FROM exam_answers;
```

### Question 12: How many of these negative Duration entries are there? (the answer should be 620)


```python
%%sql
SELECT COUNT(TIMESTAMPDIFF(MINUTE, start_time,end_time)) AS duration
FROM exam_answers
WHERE TIMESTAMPDIFF(MINUTE, start_time,end_time)<0;
```

### Question 13: How would you query all the columns of all the rows that have negative durations so that you could examine whether they share any features that might give you clues about what caused the entry mistake?


```python
%%sql
SELECT *
FROM exam_answers
WHERE TIMESTAMPDIFF(MINUTE, start_time,end_time)<0;

```

### Question 14: What is the average amount of time it took customers to complete all of the tests in the exam_answers table when the negative durations are excluded from your calculation (you should get 11233 minutes)?


```python
%%sql
SELECT AVG(TIMESTAMPDIFF(MINUTE, start_time, end_time)) AS duration
FROM exam_answers
WHERE TIMESTAMPDIFF(MINUTE, start_time,end_time)>0;

```
