
Learn how to summarize multiple subsets of your data in the same query. The method for doing this is to include a "GROUP BY" clause in the SQL query.

# GROUP BY


```python
#Query the average rating for each of the 40 tests in the Reviews table
%%sql
SELECT test_name, AVG(rating) AS AVG_Rating
FROM reviews
GROUP BY test_name
```


```python
#Group by multiple columns or derived fields.
%%sql
SELECT test_name, MONTH(created_at) AS Month, COUNT(created_at) AS Num_Completed_Tests
FROM complete_tests
GROUP BY MONTH, test_name;
```

## Question 1: Output a table that calculates the number of distinct female and male dogs in each breed group of the Dogs table, sorted by the total number of dogs in descending order:


```python
%%sql
SELECT gender, breed_group,COUNT(DISTINCT dog_guid) AS numbers
FROM dogs 
GROUP BY gender, breed_group
ORDER BY numbers DESC;
```

# The HAVING clause

Query subsets of rows using the WHERE clause, Query subsets of aggregated groups using the HAVING clause. 

## Question 3: Revise the query your wrote in Question 2 so that it (1) excludes the NULL and empty string entries in the breed_group field, and (2) excludes any groups that don't have at least 1,000 distinct Dog_Guids in them. Your result should contain 8 rows. (HINT: sometimes empty strings are registered as non-NULL values. Include the following line somewhere in your query to exclude these values as well):

breed_group!=""


```python
%%sql
SELECT gender, breed_group,COUNT(DISTINCT dog_guid) AS numbers
FROM dogs 
WHERE breed_group!="" and breed_group != 'None'and breed_group IS NOT NULL
GROUP BY 1, 2
HAVING numbers>=1000
ORDER BY 3 DESC;

```

## Question 4: Write a query that outputs the average number of tests completed and average mean inter-test-interval for every breed type, sorted by the average number of completed tests in descending order 


```python
%%sql
SELECT breed_type,
AVG(total_tests_completed) AS complete_numbers,
AVG(mean_iti_minutes) AS avg_iti
FROM dogs
GROUP BY 1
ORDER BY 2 DESC;
```

## Question 5: Write a query that outputs the average amount of time it took customers to complete each type of test where any individual reaction times over 6000 hours are excluded and only average reaction times that are greater than 0 seconds are included


```python
%%sql
SELECT test_name,
AVG(TIMESTAMPDIFF(HOUR,start_time,end_time)) AS Duration
FROM exam_answers
WHERE timestampdiff(MINUTE,start_time,end_time) < 6000
GROUP BY test_name
HAVING AVG (timestampdiff(MINUTE,start_time,end_time)) > 0 
ORDER BY Duration desc;
```

## Question 6: Write a query that outputs the total number of unique User_Guids in each combination of State and ZIP code (postal code) in the United States, sorted first by state name in ascending alphabetical order, and second by total number of unique User_Guids in descending order.


```python
%%sql
SELECT state,ZIP,COUNT(DISTINCT user_guid) AS tot_num
FROM users
WHERE country='US'
GROUP BY state,ZIP
ORDER BY 1 ASC,tot_num DESC
```

## Question 7: Write a query that outputs the total number of unique User_Guids in each combination of State and ZIP code in the United States that have at least 5 users, sorted first by state name in ascending alphabetical order, and second by total number of unique User_Guids in descending order


```python
%%sql
SELECT state, ZIP, COUNT(DISTINCT user_guid) 
FROM users
WHERE country='US'
GROUP BY state, ZIP
HAVING COUNT(DISTINCT user_guid) >5
ORDER BY 1 ASC,3 DESC;
```
