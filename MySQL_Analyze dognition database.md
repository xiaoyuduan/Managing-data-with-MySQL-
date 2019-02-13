
Copyright Jana Schaich Borg/Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)

# MySQL Exercise 12: Queries that Test Relationships Between Test Completion and Testing Circumstances 

In this lesson, we are going to practice integrating more of the concepts we learned over the past few weeks to address whether issues in our Dognition sPAP are related to the number of tests dogs complete.  We are going to focus on a subset of the issues listed in the "Features of Testing Circumstances" branch of our sPAP.  You will need to look up new functions several times and the final queries at which we will arrive by the end of this lesson will be quite complex, but we will work up to them step-by-step.  

To begin, load the sql library and database, and make the Dognition database your default database:


```python
%load_ext sql
%sql mysql://studentuser:studentpw@mysqlserver/dognitiondb
%sql USE dognitiondb
```

    The sql extension is already loaded. To reload it, use:
      %reload_ext sql
    0 rows affected.





    []



<img src="https://duke.box.com/shared/static/p2eucjdttai08eeo7davbpfgqi3zrew0.jpg" width=600 alt="SELECT FROM WHERE" />

## 1. During which weekdays do Dognition users complete the most tests?

The first question we are going to address is whether there is a certain day of the week when users are more or less likely to complete Dognition tests.  If so, targeting promotions or reminder emails to those times of the week might increase the number of tests users complete.

At first, the query we need to address this question might seem a bit intimidating, but once you can describe what the query needs to do in words, writing the query won't seem so challenging.  

Ultimately, we want a count of the number of tests completed on each day of the week, with all of the dog_guids and user_guids the Dognition team flagged in their exclude column excluded.  To achieve this, we are going to have to use the GROUP BY clause to break up counts of the records in the completed_tests table according to days of the week.  We will also have to join the completed_tests table with the dogs and users table in order to exclude completed_tests records that are associated with dog_guids or user_guids that should be excluded.  First, though, we need a method for extracting the day of the week from a time stamp.  In MySQL Exercise 2 we used a function called "DAYNAME".  That is the most efficient function to use for this purpose, but not all database systems have this function, so let's try using a different method for the queries in this lesson.   Search these sites to find a function that will output a number from 1-7 for time stamps where 1 = Sunday, 2 = Monday, …, 7 = Saturday:

http://dev.mysql.com/doc/refman/5.7/en/func-op-summary-ref.html      
http://www.w3resource.com/mysql/mysql-functions-and-operators.php

**Question 1: Using the function you found in the websites above, write a query that will output one column with the original created_at time stamp from each row in the completed_tests table, and another column with a number that represents the day of the week associated with each of those time stamps.  Limit your output to 200 rows starting at row 50.**


```python
%%sql
SELECT created_at,DAYOFWEEK(created_at)
FROM complete_tests
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>created_at</th>
        <th>DAYOFWEEK(created_at)</th>
    </tr>
    <tr>
        <td>2013-02-05 18:26:54</td>
        <td>3</td>
    </tr>
    <tr>
        <td>2013-02-05 18:31:03</td>
        <td>3</td>
    </tr>
    <tr>
        <td>2013-02-05 18:32:04</td>
        <td>3</td>
    </tr>
    <tr>
        <td>2013-02-05 18:32:25</td>
        <td>3</td>
    </tr>
    <tr>
        <td>2013-02-05 18:32:56</td>
        <td>3</td>
    </tr>
</table>



Of course, the results of the query in Question 1 would be much easier to interpret if the output included the name of the day of the week (or a relevant abbreviation) associated with each time stamp rather than a number index.

**Question 2: Include a CASE statement in the query you wrote in Question 1 to output a third column that provides the weekday name (or an appropriate abbreviation) associated with each created_at time stamp.**


```python
%%sql
SELECT created_at,DAYOFWEEK(created_at),case
       WHEN DAYOFWEEK(created_at)=1 THEN "Su"
    WHEN DAYOFWEEK(created_at)=2 THEN "Mo"
    WHEN DAYOFWEEK(created_at)=3 THEN "Tu"
    WHEN DAYOFWEEK(created_at)=4 THEN "We"
    WHEN DAYOFWEEK(created_at)=5 THEN "Th"
    WHEN DAYOFWEEK(created_at)=6 THEN "Fr"
    WHEN DAYOFWEEK(created_at)=7 THEN "Sa"
    END AS Day_of_week
FROM complete_tests
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>created_at</th>
        <th>DAYOFWEEK(created_at)</th>
        <th>Day_of_week</th>
    </tr>
    <tr>
        <td>2013-02-05 18:26:54</td>
        <td>3</td>
        <td>Tu</td>
    </tr>
    <tr>
        <td>2013-02-05 18:31:03</td>
        <td>3</td>
        <td>Tu</td>
    </tr>
    <tr>
        <td>2013-02-05 18:32:04</td>
        <td>3</td>
        <td>Tu</td>
    </tr>
    <tr>
        <td>2013-02-05 18:32:25</td>
        <td>3</td>
        <td>Tu</td>
    </tr>
    <tr>
        <td>2013-02-05 18:32:56</td>
        <td>3</td>
        <td>Tu</td>
    </tr>
</table>



Now that we are confident we have the correct syntax for extracting weekday labels from the created_at time stamps, we can start building our larger query that examines the number of tests completed on each weekday.

**Question 3: Adapt the query you wrote in Question 2 to report the total number of tests completed on each weekday.  Sort the results by the total number of tests completed in descending order.  You should get a total of 33,190 tests in the Sunday row of your output.**


```python
%%sql
SELECT DAYOFWEEK(created_at),case
       WHEN DAYOFWEEK(created_at)=1 THEN "Su"
    WHEN DAYOFWEEK(created_at)=2 THEN "Mo"
    WHEN DAYOFWEEK(created_at)=3 THEN "Tu"
    WHEN DAYOFWEEK(created_at)=4 THEN "We"
    WHEN DAYOFWEEK(created_at)=5 THEN "Th"
    WHEN DAYOFWEEK(created_at)=6 THEN "Fr"
    WHEN DAYOFWEEK(created_at)=7 THEN "Sa"
    END AS Day_of_week,
    COUNT(created_at) AS total_number
FROM complete_tests
GROUP BY 2
ORDER BY total_number DESC;
```

    7 rows affected.





<table>
    <tr>
        <th>DAYOFWEEK(created_at)</th>
        <th>Day_of_week</th>
        <th>total_number</th>
    </tr>
    <tr>
        <td>1</td>
        <td>Su</td>
        <td>33190</td>
    </tr>
    <tr>
        <td>2</td>
        <td>Mo</td>
        <td>30195</td>
    </tr>
    <tr>
        <td>3</td>
        <td>Tu</td>
        <td>27989</td>
    </tr>
    <tr>
        <td>7</td>
        <td>Sa</td>
        <td>27899</td>
    </tr>
    <tr>
        <td>4</td>
        <td>We</td>
        <td>26473</td>
    </tr>
    <tr>
        <td>5</td>
        <td>Th</td>
        <td>24420</td>
    </tr>
    <tr>
        <td>6</td>
        <td>Fr</td>
        <td>23080</td>
    </tr>
</table>



So far these results suggest that users complete the most tests on Sunday night and the fewest tests on Friday night.  We need to determine if this trend remains after flagged dog_guids and user_guids are excluded.  Let's start by removing the dog_guids that have an exclude flag.  We'll exclude user_guids with an exclude flag in later queries.

**Question 4: Rewrite the query in Question 3 to exclude the dog_guids that have a value of "1" in the exclude column (Hint: this query will require a join.)  This time you should get a total of 31,092 tests in the Sunday row of your output.**


```python
%%sql
SELECT DAYOFWEEK(c.created_at),case
       WHEN DAYOFWEEK(c.created_at)=1 THEN "Su"
    WHEN DAYOFWEEK(c.created_at)=2 THEN "Mo"
    WHEN DAYOFWEEK(c.created_at)=3 THEN "Tu"
    WHEN DAYOFWEEK(c.created_at)=4 THEN "We"
    WHEN DAYOFWEEK(c.created_at)=5 THEN "Th"
    WHEN DAYOFWEEK(c.created_at)=6 THEN "Fr"
    WHEN DAYOFWEEK(c.created_at)=7 THEN "Sa"
    END AS Day_of_week,
    COUNT(c.created_at) AS total_number
FROM complete_tests c JOIN dogs d
ON d.dog_guid=c.dog_guid
WHERE d.exclude IS NULL or d.exclude=0
GROUP BY 2
ORDER BY total_number DESC;
```

    7 rows affected.





<table>
    <tr>
        <th>DAYOFWEEK(c.created_at)</th>
        <th>Day_of_week</th>
        <th>total_number</th>
    </tr>
    <tr>
        <td>1</td>
        <td>Su</td>
        <td>31092</td>
    </tr>
    <tr>
        <td>2</td>
        <td>Mo</td>
        <td>28250</td>
    </tr>
    <tr>
        <td>7</td>
        <td>Sa</td>
        <td>26231</td>
    </tr>
    <tr>
        <td>3</td>
        <td>Tu</td>
        <td>25764</td>
    </tr>
    <tr>
        <td>4</td>
        <td>We</td>
        <td>24501</td>
    </tr>
    <tr>
        <td>5</td>
        <td>Th</td>
        <td>22347</td>
    </tr>
    <tr>
        <td>6</td>
        <td>Fr</td>
        <td>21028</td>
    </tr>
</table>



Now we need to exclude the user_guids that have a value of "1" in the exclude column as well.  One way to do this would be to join the completed_tests, dogs, and users table with a sequence of inner joins.  However, we've seen in previous lessons that there are duplicate rows in the users table.  These duplicates will get passed through the join and will affect the count calculations.  To illustrate this, compare the following two queries.

**Question 5: Write a query to retrieve all the dog_guids for users common to the dogs and users table using the traditional inner join syntax (your output will have 950,331 rows).**


```python
%%sql
SELECT dog_guid
FROM dogs d INNER JOIN users u
ON d.user_guid=u.user_guid
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>dog_guid</th>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd417cac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd3fb0f2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
</table>



**Question 6: Write a query to retrieve all the *distinct* dog_guids common to the dogs and users table using the traditional inner join syntax (your output will have 35,048 rows).**


```python
%%sql
SELECT DISTINCT dog_guid
FROM dogs d INNER JOIN users u
ON d.user_guid=u.user_guid
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>dog_guid</th>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd417cac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd3fb0f2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
</table>



The strategy we will use to handle duplicate rows in the users table will be to, first, write a subquery that retrieves the distinct dog_guids from an inner join between the dogs and users table with the appropriate records excluded.  Then, second, we will join the result of this subquery to the complete_tests table and group the results according to the day of the week.

**Question 7: Start by writing a query that retrieves distinct dog_guids common to the dogs and users table, excuding dog_guids and user_guids with a "1" in their respective exclude columns (your output will have 34,121 rows).**


```python
%%sql
SELECT DISTINCT dog_guid
FROM dogs d INNER JOIN users u
ON d.user_guid=u.user_guid
WHERE (u.exclude IS NULL OR u.exclude=0) AND (d.exclude IS NULL OR
d.exclude=0)
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>dog_guid</th>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd417cac-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd3fb0f2-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
    </tr>
</table>



**Question 8: Now adapt your query from Question 4 so that it inner joins on the result of the subquery you wrote in Question 7 instead of the dogs table.  This will give you a count of the number of tests completed on each day of the week, excluding all of the dog_guids and user_guids that the Dognition team flagged in the exclude columns.**  


```python
%%sql
SELECT DAYOFWEEK(c.created_at),case
       WHEN DAYOFWEEK(c.created_at)=1 THEN "Su"
    WHEN DAYOFWEEK(c.created_at)=2 THEN "Mo"
    WHEN DAYOFWEEK(c.created_at)=3 THEN "Tu"
    WHEN DAYOFWEEK(c.created_at)=4 THEN "We"
    WHEN DAYOFWEEK(c.created_at)=5 THEN "Th"
    WHEN DAYOFWEEK(c.created_at)=6 THEN "Fr"
    WHEN DAYOFWEEK(c.created_at)=7 THEN "Sa"
    END AS Day_of_week,
    COUNT(c.created_at) AS total_number
FROM complete_tests c JOIN 
(SELECT DISTINCT dog_guid
FROM dogs d INNER JOIN users u
ON d.user_guid=u.user_guid
WHERE (u.exclude IS NULL OR u.exclude=0) AND (d.exclude IS NULL OR
d.exclude=0)) AS new 

ON new.dog_guid=c.dog_guid
GROUP BY 2
ORDER BY total_number DESC;
```

    7 rows affected.





<table>
    <tr>
        <th>DAYOFWEEK(c.created_at)</th>
        <th>Day_of_week</th>
        <th>total_number</th>
    </tr>
    <tr>
        <td>1</td>
        <td>Su</td>
        <td>31036</td>
    </tr>
    <tr>
        <td>2</td>
        <td>Mo</td>
        <td>28138</td>
    </tr>
    <tr>
        <td>7</td>
        <td>Sa</td>
        <td>26149</td>
    </tr>
    <tr>
        <td>3</td>
        <td>Tu</td>
        <td>25696</td>
    </tr>
    <tr>
        <td>4</td>
        <td>We</td>
        <td>24433</td>
    </tr>
    <tr>
        <td>5</td>
        <td>Th</td>
        <td>22323</td>
    </tr>
    <tr>
        <td>6</td>
        <td>Fr</td>
        <td>21027</td>
    </tr>
</table>



These results still suggest that Sunday is the day when the most tests are completed and Friday is the day when the fewest tests are completed.  However, our first query suggested that more tests were completed on Tuesday than Saturday; our current query suggests that slightly more tests are completed on Saturday than Tuesday, now that flagged dog_guids and user_guids are excluded.

It's always a good idea to see if a data pattern replicates before you interpret it too strongly.  The ideal way to do this would be to have a completely separate and independent data set to analyze.  We don't have such a data set, but we can assess the reliability of the day of the week patterns in a different way.  We can test whether the day of the week patterns are the same in all years of our data set.

**Question 9: Adapt your query from Question 8 to provide a count of the number of tests completed on each weekday of each year in the Dognition data set.  Exclude all dog_guids and user_guids with a value of "1" in their exclude columns.  Sort the output by year in ascending order, and then by the total number of tests completed in descending order. HINT: you will need a function described in one of these references to retrieve the year of each time stamp in the created_at field:**

http://dev.mysql.com/doc/refman/5.7/en/func-op-summary-ref.html      
http://www.w3resource.com/mysql/mysql-functions-and-operators.php


```python
%%sql
SELECT DAYOFWEEK(c.created_at),YEAR(c.created_at) AS year,case
       WHEN DAYOFWEEK(c.created_at)=1 THEN "Su"
    WHEN DAYOFWEEK(c.created_at)=2 THEN "Mo"
    WHEN DAYOFWEEK(c.created_at)=3 THEN "Tu"
    WHEN DAYOFWEEK(c.created_at)=4 THEN "We"
    WHEN DAYOFWEEK(c.created_at)=5 THEN "Th"
    WHEN DAYOFWEEK(c.created_at)=6 THEN "Fr"
    WHEN DAYOFWEEK(c.created_at)=7 THEN "Sa"
    END AS Day_of_week,
    COUNT(c.created_at) AS total_number
FROM complete_tests c JOIN 
(SELECT DISTINCT dog_guid
FROM dogs d JOIN users u
ON d.user_guid=u.user_guid
WHERE (u.exclude IS NULL OR u.exclude=0) AND (d.exclude IS NULL OR
d.exclude=0)) AS new 

ON new.dog_guid=c.dog_guid
GROUP BY 2,3
ORDER BY year ASC,total_number DESC;
```

    21 rows affected.





<table>
    <tr>
        <th>DAYOFWEEK(c.created_at)</th>
        <th>year</th>
        <th>Day_of_week</th>
        <th>total_number</th>
    </tr>
    <tr>
        <td>1</td>
        <td>2013</td>
        <td>Su</td>
        <td>8203</td>
    </tr>
    <tr>
        <td>7</td>
        <td>2013</td>
        <td>Sa</td>
        <td>6854</td>
    </tr>
    <tr>
        <td>2</td>
        <td>2013</td>
        <td>Mo</td>
        <td>5740</td>
    </tr>
    <tr>
        <td>4</td>
        <td>2013</td>
        <td>We</td>
        <td>5665</td>
    </tr>
    <tr>
        <td>3</td>
        <td>2013</td>
        <td>Tu</td>
        <td>5393</td>
    </tr>
    <tr>
        <td>6</td>
        <td>2013</td>
        <td>Fr</td>
        <td>4997</td>
    </tr>
    <tr>
        <td>5</td>
        <td>2013</td>
        <td>Th</td>
        <td>4961</td>
    </tr>
    <tr>
        <td>2</td>
        <td>2014</td>
        <td>Mo</td>
        <td>9309</td>
    </tr>
    <tr>
        <td>1</td>
        <td>2014</td>
        <td>Su</td>
        <td>9210</td>
    </tr>
    <tr>
        <td>3</td>
        <td>2014</td>
        <td>Tu</td>
        <td>9177</td>
    </tr>
    <tr>
        <td>4</td>
        <td>2014</td>
        <td>We</td>
        <td>8857</td>
    </tr>
    <tr>
        <td>7</td>
        <td>2014</td>
        <td>Sa</td>
        <td>8257</td>
    </tr>
    <tr>
        <td>5</td>
        <td>2014</td>
        <td>Th</td>
        <td>7286</td>
    </tr>
    <tr>
        <td>6</td>
        <td>2014</td>
        <td>Fr</td>
        <td>6475</td>
    </tr>
    <tr>
        <td>1</td>
        <td>2015</td>
        <td>Su</td>
        <td>13623</td>
    </tr>
    <tr>
        <td>2</td>
        <td>2015</td>
        <td>Mo</td>
        <td>13089</td>
    </tr>
    <tr>
        <td>3</td>
        <td>2015</td>
        <td>Tu</td>
        <td>11126</td>
    </tr>
    <tr>
        <td>7</td>
        <td>2015</td>
        <td>Sa</td>
        <td>11038</td>
    </tr>
    <tr>
        <td>5</td>
        <td>2015</td>
        <td>Th</td>
        <td>10076</td>
    </tr>
    <tr>
        <td>4</td>
        <td>2015</td>
        <td>We</td>
        <td>9911</td>
    </tr>
    <tr>
        <td>6</td>
        <td>2015</td>
        <td>Fr</td>
        <td>9555</td>
    </tr>
</table>



These results suggest that although the precise order of the weekdays with the most to fewest completed tests changes slightly from year to year, Sundays always have a lot of completed tests, and Fridays always have the fewest or close to the fewest completed tests.  So far, it seems like it might be a good idea for Dognition to target reminder or encouragement messages to customers on Sundays.  However, there is one more issue our analysis does not address.  All of the time stamps in the created_at column are in Coordinated Universal Time (abbreviated UTC).  This is a time convention that is constant around the globe.  Nonetheless, as the picture below illustrates, countries and states have different time zones.  The same UTC time can correspond with local times in different countries that are as much as 24 hours apart:

<img src="https://duke.box.com/shared/static/0p8ee9az908soq1m0o4jst94vqlh2oh7.jpg" width=600 alt="TIME_ZONE_MAP" />


Therefore, the weekdays we have extracted so far may not accurately reflect the weekdays in the local times of different countries.  The only way to correct the time stamps for time zone differences is to obtain a table with the time zones of every city, state, or country.  Such a table was not available to us in this course, but we can run some analyses that approximate a time zone correction for United States customers.

**Question 10: First, adapt your query from Question 9 so that you only examine customers located in the United States, with Hawaii and Alaska residents excluded.  HINTS: In this data set, the abbreviation for the United States is "US", the abbreviation for Hawaii is "HI" and the abbreviation for Alaska is "AK".  You should have 5,860 tests completed on Sunday of 2013.**


```python
%%sql
SELECT DAYOFWEEK(c.created_at),YEAR(c.created_at) AS year,case
       WHEN DAYOFWEEK(c.created_at)=1 THEN "Su"
    WHEN DAYOFWEEK(c.created_at)=2 THEN "Mo"
    WHEN DAYOFWEEK(c.created_at)=3 THEN "Tu"
    WHEN DAYOFWEEK(c.created_at)=4 THEN "We"
    WHEN DAYOFWEEK(c.created_at)=5 THEN "Th"
    WHEN DAYOFWEEK(c.created_at)=6 THEN "Fr"
    WHEN DAYOFWEEK(c.created_at)=7 THEN "Sa"
    END AS Day_of_week,
    COUNT(c.created_at) AS total_number
FROM complete_tests c JOIN 
(SELECT DISTINCT dog_guid
FROM dogs d JOIN users u
ON d.user_guid=u.user_guid
WHERE u.country="US" and u.state != "HI" and u.state != "AK" and (u.exclude IS NULL OR u.exclude=0) AND (d.exclude IS NULL OR
d.exclude=0)) AS new 

ON new.dog_guid=c.dog_guid
GROUP BY 2,3
ORDER BY year ASC,total_number DESC;
```

    21 rows affected.





<table>
    <tr>
        <th>DAYOFWEEK(c.created_at)</th>
        <th>year</th>
        <th>Day_of_week</th>
        <th>total_number</th>
    </tr>
    <tr>
        <td>1</td>
        <td>2013</td>
        <td>Su</td>
        <td>5860</td>
    </tr>
    <tr>
        <td>7</td>
        <td>2013</td>
        <td>Sa</td>
        <td>4674</td>
    </tr>
    <tr>
        <td>2</td>
        <td>2013</td>
        <td>Mo</td>
        <td>3695</td>
    </tr>
    <tr>
        <td>4</td>
        <td>2013</td>
        <td>We</td>
        <td>3496</td>
    </tr>
    <tr>
        <td>3</td>
        <td>2013</td>
        <td>Tu</td>
        <td>3449</td>
    </tr>
    <tr>
        <td>6</td>
        <td>2013</td>
        <td>Fr</td>
        <td>3163</td>
    </tr>
    <tr>
        <td>5</td>
        <td>2013</td>
        <td>Th</td>
        <td>3090</td>
    </tr>
    <tr>
        <td>2</td>
        <td>2014</td>
        <td>Mo</td>
        <td>7278</td>
    </tr>
    <tr>
        <td>3</td>
        <td>2014</td>
        <td>Tu</td>
        <td>6800</td>
    </tr>
    <tr>
        <td>1</td>
        <td>2014</td>
        <td>Su</td>
        <td>6632</td>
    </tr>
    <tr>
        <td>4</td>
        <td>2014</td>
        <td>We</td>
        <td>6331</td>
    </tr>
    <tr>
        <td>7</td>
        <td>2014</td>
        <td>Sa</td>
        <td>6006</td>
    </tr>
    <tr>
        <td>5</td>
        <td>2014</td>
        <td>Th</td>
        <td>5271</td>
    </tr>
    <tr>
        <td>6</td>
        <td>2014</td>
        <td>Fr</td>
        <td>4831</td>
    </tr>
    <tr>
        <td>2</td>
        <td>2015</td>
        <td>Mo</td>
        <td>8784</td>
    </tr>
    <tr>
        <td>1</td>
        <td>2015</td>
        <td>Su</td>
        <td>8570</td>
    </tr>
    <tr>
        <td>3</td>
        <td>2015</td>
        <td>Tu</td>
        <td>7218</td>
    </tr>
    <tr>
        <td>7</td>
        <td>2015</td>
        <td>Sa</td>
        <td>7116</td>
    </tr>
    <tr>
        <td>5</td>
        <td>2015</td>
        <td>Th</td>
        <td>6343</td>
    </tr>
    <tr>
        <td>4</td>
        <td>2015</td>
        <td>We</td>
        <td>6164</td>
    </tr>
    <tr>
        <td>6</td>
        <td>2015</td>
        <td>Fr</td>
        <td>5689</td>
    </tr>
</table>



The next step is to adjust the created_at times for differences in time zone. Most United States states (excluding Hawaii and Alaska) have a time zone of UTC time -5 hours (in the eastern-most regions) to -8 hours (in the western-most regions).  To get a general idea for how much our weekday analysis is likely to change based on time zone, we will subtract 6 hours from every time stamp in the complete_tests table.  Although this means our time stamps can be inaccurate by 1 or 2 hours, people are not likely to be playing Dognition games at midnight, so 1-2 hours should not affect the weekdays extracted from each time stamp too much. 

The functions used to subtract time differ across database systems, so you should double-check which function you need to use every time you are working with a new database.  We will use the date_sub function:

http://www.w3schools.com/sql/func_date_sub.asp

**Question 11: Write a query that extracts the original created_at time stamps for rows in the complete_tests table in one column, and the created_at time stamps with 6 hours subtracted in another column.  Limit your output to 100 rows.**


```python
%%sql
SELECT created_at, date_sub(created_at,interval 6 hour)
FROM complete_tests
LIMIT 10;
```

    10 rows affected.





<table>
    <tr>
        <th>created_at</th>
        <th>date_sub(created_at,interval 6 hour)</th>
    </tr>
    <tr>
        <td>2013-02-05 18:26:54</td>
        <td>2013-02-05 12:26:54</td>
    </tr>
    <tr>
        <td>2013-02-05 18:31:03</td>
        <td>2013-02-05 12:31:03</td>
    </tr>
    <tr>
        <td>2013-02-05 18:32:04</td>
        <td>2013-02-05 12:32:04</td>
    </tr>
    <tr>
        <td>2013-02-05 18:32:25</td>
        <td>2013-02-05 12:32:25</td>
    </tr>
    <tr>
        <td>2013-02-05 18:32:56</td>
        <td>2013-02-05 12:32:56</td>
    </tr>
    <tr>
        <td>2013-02-05 18:33:15</td>
        <td>2013-02-05 12:33:15</td>
    </tr>
    <tr>
        <td>2013-02-05 18:33:33</td>
        <td>2013-02-05 12:33:33</td>
    </tr>
    <tr>
        <td>2013-02-05 18:33:59</td>
        <td>2013-02-05 12:33:59</td>
    </tr>
    <tr>
        <td>2013-02-05 18:34:25</td>
        <td>2013-02-05 12:34:25</td>
    </tr>
    <tr>
        <td>2013-02-05 18:34:39</td>
        <td>2013-02-05 12:34:39</td>
    </tr>
</table>



**Question 12: Use your query from Question 11 to adapt your query from Question 10 in order to provide a count of the number of tests completed on each day of the week, with approximate time zones taken into account, in each year in the Dognition data set. Exclude all dog_guids and user_guids with a value of "1" in their exclude columns. Sort the output by year in ascending order, and then by the total number of tests completed in descending order. HINT: Don't forget to adjust for the time zone in your DAYOFWEEK statement and your CASE statement.** 


```python
%%sql
SELECT DAYOFWEEK(date_sub(created_at,interval 6 hour)),YEAR(c.created_at) AS year,case
       WHEN DAYOFWEEK(date_sub(created_at,interval 6 hour))=1 THEN "Su"
    WHEN DAYOFWEEK(date_sub(created_at,interval 6 hour))=2 THEN "Mo"
    WHEN DAYOFWEEK(date_sub(created_at,interval 6 hour))=3 THEN "Tu"
    WHEN DAYOFWEEK(date_sub(created_at,interval 6 hour))=4 THEN "We"
    WHEN DAYOFWEEK(date_sub(created_at,interval 6 hour))=5 THEN "Th"
    WHEN DAYOFWEEK(date_sub(created_at,interval 6 hour))=6 THEN "Fr"
    WHEN DAYOFWEEK(date_sub(created_at,interval 6 hour))=7 THEN "Sa"
    END AS Day_of_week,
    COUNT(c.created_at) AS total_number

FROM complete_tests c JOIN 
(SELECT DISTINCT dog_guid
FROM dogs d JOIN users u
ON d.user_guid=u.user_guid
WHERE u.country="US" and u.state != "HI" and u.state != "AK" and (u.exclude IS NULL OR u.exclude=0) AND (d.exclude IS NULL OR
d.exclude=0)) AS new 

ON new.dog_guid=c.dog_guid
GROUP BY 2,3
ORDER BY year ASC,total_number DESC;
```

    21 rows affected.





<table>
    <tr>
        <th>DAYOFWEEK(date_sub(created_at,interval 6 hour))</th>
        <th>year</th>
        <th>Day_of_week</th>
        <th>total_number</th>
    </tr>
    <tr>
        <td>1</td>
        <td>2013</td>
        <td>Su</td>
        <td>6061</td>
    </tr>
    <tr>
        <td>7</td>
        <td>2013</td>
        <td>Sa</td>
        <td>4754</td>
    </tr>
    <tr>
        <td>2</td>
        <td>2013</td>
        <td>Mo</td>
        <td>3798</td>
    </tr>
    <tr>
        <td>4</td>
        <td>2013</td>
        <td>We</td>
        <td>3410</td>
    </tr>
    <tr>
        <td>3</td>
        <td>2013</td>
        <td>Tu</td>
        <td>3276</td>
    </tr>
    <tr>
        <td>5</td>
        <td>2013</td>
        <td>Th</td>
        <td>3079</td>
    </tr>
    <tr>
        <td>6</td>
        <td>2013</td>
        <td>Fr</td>
        <td>3049</td>
    </tr>
    <tr>
        <td>2</td>
        <td>2014</td>
        <td>Mo</td>
        <td>7908</td>
    </tr>
    <tr>
        <td>1</td>
        <td>2014</td>
        <td>Su</td>
        <td>7736</td>
    </tr>
    <tr>
        <td>3</td>
        <td>2014</td>
        <td>Tu</td>
        <td>6513</td>
    </tr>
    <tr>
        <td>7</td>
        <td>2014</td>
        <td>Sa</td>
        <td>6081</td>
    </tr>
    <tr>
        <td>4</td>
        <td>2014</td>
        <td>We</td>
        <td>5772</td>
    </tr>
    <tr>
        <td>5</td>
        <td>2014</td>
        <td>Th</td>
        <td>4800</td>
    </tr>
    <tr>
        <td>6</td>
        <td>2014</td>
        <td>Fr</td>
        <td>4339</td>
    </tr>
    <tr>
        <td>1</td>
        <td>2015</td>
        <td>Su</td>
        <td>10406</td>
    </tr>
    <tr>
        <td>2</td>
        <td>2015</td>
        <td>Mo</td>
        <td>8229</td>
    </tr>
    <tr>
        <td>7</td>
        <td>2015</td>
        <td>Sa</td>
        <td>7154</td>
    </tr>
    <tr>
        <td>3</td>
        <td>2015</td>
        <td>Tu</td>
        <td>6673</td>
    </tr>
    <tr>
        <td>4</td>
        <td>2015</td>
        <td>We</td>
        <td>6266</td>
    </tr>
    <tr>
        <td>5</td>
        <td>2015</td>
        <td>Th</td>
        <td>5881</td>
    </tr>
    <tr>
        <td>6</td>
        <td>2015</td>
        <td>Fr</td>
        <td>5275</td>
    </tr>
</table>



You can try re-running the query with time-zone corrections of 5, 7, or 8 hours, and the results remain essentially the same.  All of these analyses suggest that customers are most likely to complete tests around Sunday and Monday, and least likely to complete tests around the end of the work week, on Thursday and Friday. This is certainly valuable information for Dognition to take advantage of.

If you were presenting this information to the Dognition team, you might want to present the information in the form of a graph that you make in another program.  The graph would be easier to read if the output was ordered according to the days of the week shown in standard calendars, with Monday being the first day and Sunday being the last day.  MySQL provides an easy way to do this using the FIELD function in the ORDER BY statement:

https://www.virendrachandak.com/techtalk/mysql-ordering-results-by-specific-field-values/

**Question 13: Adapt your query from Question 12 so that the results are sorted by year in ascending order, and then by the day of the week in the following order: Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday.**


```python
%%sql
SELECT DAYOFWEEK(DATE_SUB(created_at, interval 6 hour)) AS dayasnum,
YEAR(c.created_at) AS year, COUNT(c.created_at) AS numtests,
(CASE
WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=1 THEN "Su"
WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=2 THEN "Mo"
WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=3 THEN "Tu"
WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=4 THEN "We"
WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=5 THEN "Th"
WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=6 THEN "Fr"
WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=7 THEN "Sa"
END) AS daylabel

FROM complete_tests c JOIN
(SELECT DISTINCT dog_guid
FROM dogs d JOIN users u
ON d.user_guid=u.user_guid
WHERE ((u.exclude IS NULL OR u.exclude=0)
AND u.country="US"
AND (u.state!="HI" AND u.state!="AK")
AND (d.exclude IS NULL OR d.exclude=0))) AS dogs_cleaned

ON c.dog_guid=dogs_cleaned.dog_guid
GROUP BY year,daylabel
ORDER BY year ASC, FIELD(daylabel,'Mo','Tu','We','Th','Fr','Sa','Su');
```

    21 rows affected.





<table>
    <tr>
        <th>dayasnum</th>
        <th>year</th>
        <th>numtests</th>
        <th>daylabel</th>
    </tr>
    <tr>
        <td>2</td>
        <td>2013</td>
        <td>3798</td>
        <td>Mo</td>
    </tr>
    <tr>
        <td>3</td>
        <td>2013</td>
        <td>3276</td>
        <td>Tu</td>
    </tr>
    <tr>
        <td>4</td>
        <td>2013</td>
        <td>3410</td>
        <td>We</td>
    </tr>
    <tr>
        <td>5</td>
        <td>2013</td>
        <td>3079</td>
        <td>Th</td>
    </tr>
    <tr>
        <td>6</td>
        <td>2013</td>
        <td>3049</td>
        <td>Fr</td>
    </tr>
    <tr>
        <td>7</td>
        <td>2013</td>
        <td>4754</td>
        <td>Sa</td>
    </tr>
    <tr>
        <td>1</td>
        <td>2013</td>
        <td>6061</td>
        <td>Su</td>
    </tr>
    <tr>
        <td>2</td>
        <td>2014</td>
        <td>7908</td>
        <td>Mo</td>
    </tr>
    <tr>
        <td>3</td>
        <td>2014</td>
        <td>6513</td>
        <td>Tu</td>
    </tr>
    <tr>
        <td>4</td>
        <td>2014</td>
        <td>5772</td>
        <td>We</td>
    </tr>
    <tr>
        <td>5</td>
        <td>2014</td>
        <td>4800</td>
        <td>Th</td>
    </tr>
    <tr>
        <td>6</td>
        <td>2014</td>
        <td>4339</td>
        <td>Fr</td>
    </tr>
    <tr>
        <td>7</td>
        <td>2014</td>
        <td>6081</td>
        <td>Sa</td>
    </tr>
    <tr>
        <td>1</td>
        <td>2014</td>
        <td>7736</td>
        <td>Su</td>
    </tr>
    <tr>
        <td>2</td>
        <td>2015</td>
        <td>8229</td>
        <td>Mo</td>
    </tr>
    <tr>
        <td>3</td>
        <td>2015</td>
        <td>6673</td>
        <td>Tu</td>
    </tr>
    <tr>
        <td>4</td>
        <td>2015</td>
        <td>6266</td>
        <td>We</td>
    </tr>
    <tr>
        <td>5</td>
        <td>2015</td>
        <td>5881</td>
        <td>Th</td>
    </tr>
    <tr>
        <td>6</td>
        <td>2015</td>
        <td>5275</td>
        <td>Fr</td>
    </tr>
    <tr>
        <td>7</td>
        <td>2015</td>
        <td>7154</td>
        <td>Sa</td>
    </tr>
    <tr>
        <td>1</td>
        <td>2015</td>
        <td>10406</td>
        <td>Su</td>
    </tr>
</table>



Unfortunately other database platforms do not have the ORDER BY FIELD functionality.  To achieve the same result in other platforms, you would have to use a CASE statement or a more advanced solution:

http://stackoverflow.com/questions/1309624/simulating-mysqls-order-by-field-in-postgresql

The link provided above is to a discussion on stackoverflow.com.  Stackoverflow is a great website that, in their words, "is a community of 4.7 million programmers, just like you, helping each other."  You can ask questions about SQL queries and get help from other experts, or search through questions posted previously to see if somebody else has already asked a question that is relevant to the problem you are trying to solve.  It's a great resource to use whenever you run into trouble with your queries.

## 2. Which states and countries have the most Dognition users?

You ended up with a pretty long and complex query in the questions above that you tested step-by-step.  Many people save these types of queries so that they can be adapted for similar queries in the future without having to redesign and retest the entire query.  
    
In the next two questions, we will practice repurposing previously-designed queries for new questions.  Both questions can be answered through relatively minor modifications of the queries you wrote above.

**Question 14: Which 5 states within the United States have the most Dognition customers, once all dog_guids and user_guids with a value of "1" in their exclude columns are removed?  Try using the following general strategy: count how many unique user_guids are associated with dogs in the complete_tests table, break up the counts according to state, sort the results by counts of unique user_guids in descending order, and then limit your output to 5 rows. California ("CA") and New York ("NY") should be at the top of your list.**


```python
%%sql
SELECT new.state,COUNT(DISTINCT new.user_guid) AS total_number
FROM complete_tests c JOIN 
(SELECT DISTINCT dog_guid,u.user_guid,u.state
FROM dogs d JOIN users u
ON d.user_guid=u.user_guid
WHERE u.country="US" and (u.exclude IS NULL OR u.exclude=0) AND (d.exclude IS NULL OR
d.exclude=0)) AS new 

ON new.dog_guid=c.dog_guid
GROUP BY 1
ORDER BY total_number DESC
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>state</th>
        <th>total_number</th>
    </tr>
    <tr>
        <td>CA</td>
        <td>1363</td>
    </tr>
    <tr>
        <td>NY</td>
        <td>628</td>
    </tr>
    <tr>
        <td>TX</td>
        <td>536</td>
    </tr>
    <tr>
        <td>FL</td>
        <td>502</td>
    </tr>
    <tr>
        <td>NC</td>
        <td>467</td>
    </tr>
</table>




```python
#sample answer
%%sql
SELECT dogs_cleaned.state AS state, COUNT(DISTINCT dogs_cleaned.user_guid) AS
numusers
FROM complete_tests c JOIN
(SELECT DISTINCT dog_guid, u.user_guid, u.state
FROM dogs d JOIN users u
ON d.user_guid=u.user_guid
WHERE ((u.exclude IS NULL OR u.exclude=0)
AND u.country="US"
AND (d.exclude IS NULL OR d.exclude=0))) AS dogs_cleaned
ON c.dog_guid=dogs_cleaned.dog_guid
GROUP BY state
ORDER BY numusers DESC
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>state</th>
        <th>numusers</th>
    </tr>
    <tr>
        <td>CA</td>
        <td>1363</td>
    </tr>
    <tr>
        <td>NY</td>
        <td>628</td>
    </tr>
    <tr>
        <td>TX</td>
        <td>536</td>
    </tr>
    <tr>
        <td>FL</td>
        <td>502</td>
    </tr>
    <tr>
        <td>NC</td>
        <td>467</td>
    </tr>
</table>



The number of unique Dognition users in California is more than two times greater than any other state.  This information could be very helpful to Dognition.  Useful follow-up questions would be: were special promotions run in California that weren't run in other states?  Did Dognition use advertising channels that are particularly effective in California?  If not, what traits differentiate California users from other users?  Can these traits be taken advantage of in future marketing efforts or product developments?

Let's try one more analysis that examines testing circumstances from a different angle.

**Question 15: Which 10 countries have the most Dognition customers, once all dog_guids and user_guids with a value of "1" in their exclude columns are removed? HINT: don't forget to remove the u.country="US" statement from your WHERE clause.**


```python
%%sql
SELECT dogs_cleaned.country AS country, COUNT(DISTINCT dogs_cleaned.user_guid) AS
numusers
FROM complete_tests c JOIN
(SELECT DISTINCT dog_guid, u.user_guid, u.country
FROM dogs d JOIN users u
ON d.user_guid=u.user_guid
WHERE ((u.exclude IS NULL OR u.exclude=0)
AND (d.exclude IS NULL OR d.exclude=0))) AS dogs_cleaned
ON c.dog_guid=dogs_cleaned.dog_guid
GROUP BY country
ORDER BY numusers DESC
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>country</th>
        <th>numusers</th>
    </tr>
    <tr>
        <td>US</td>
        <td>8936</td>
    </tr>
    <tr>
        <td>N/A</td>
        <td>5466</td>
    </tr>
    <tr>
        <td>CA</td>
        <td>484</td>
    </tr>
    <tr>
        <td>AU</td>
        <td>142</td>
    </tr>
    <tr>
        <td>GB</td>
        <td>123</td>
    </tr>
</table>



The United States, Canada, Australia, and Great Britain are the countries with the most Dognition users.  N/A refers to "not applicable" which essentially means we have no usable country data from those rows.  After Great Britain, the number of Dognition users drops quite a lot.  This analysis suggests that Dognition is most likely to be used by English-speaking countries.  One question Dognition might want to consider is whether there are any countries whose participation would dramatically increase if a translated website were available.

## 3. Congratulations!

You have now written many complex queries on your own that address real analysis questions about a real business problem.  You know how to look up new functions, you know how to troubleshoot your queries by isolating each piece of the query until you are sure the syntax is correct, and you know where to look for help if you get stuck.  You are ready to start using SQL in your own business ventures.  Keep learning, keep trying new things, and keep asking questions.  Congratulations for taking your career to the next level!

There is another video to watch, and of course, more exercises to work through using the Dillard's data set.  
    
**In the meantime, enjoy practicing any other queries you want to try here:**


```python

```
