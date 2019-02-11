
Copyright Jana Schaich Borg/Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)

# MySQL Exercise 10: Useful Logical Operators

There are a few more logical operators we haven't covered yet that you might find useful when designing your queries.  Expressions that use logical operators return a result of "true" or "false", depending on whether the conditions you specify are met.  The "true" or "false" results are usually used to determine which, if any, subsequent parts of your query will be run.  We will discuss the IF operator, the CASE operator, and the order of operations within logical expressions in this lesson.

**Begin by loading the sql library and database, and making the Dognition database your default database:**


```python
%load_ext sql
%sql mysql://studentuser:studentpw@mysqlserver/dognitiondb
%sql USE dognitiondb
```

    0 rows affected.





    []



## 1. IF expressions

IF expressions are used to return one of two results based on whether inputs to the expressions meet the conditions you specify.  They are frequently used in SELECT statements as a compact way to rename values in a column.  The basic syntax is as follows:

```
IF([your conditions],[value outputted if conditions are met],[value outputted if conditions are NOT met])
```

So we could write:

```sql
SELECT created_at, IF(created_at<'2014-06-01','early_user','late_user') AS user_type
FROM users
```  
to output one column that provided the time stamp of when a user account was created, and a second column called user_type that used that time stamp to determine whether the user was an early or late user. User_type could then be used in a GROUP BY statement to segment summary calculations (in database systems that support the use of aliases in GROUP BY statements). 
   
For example, since we know there are duplicate user_guids in the user table, we could combine a subquery with an IF statement to retrieve a list of unique user_guids with their classification as either an early or late user (based on when their first user entry was created): 

```sql
SELECT cleaned_users.user_guid as UserID,
       IF(cleaned_users.first_account<'2014-06-01','early_user','late_user') AS user_type
FROM (SELECT user_guid, MIN(created_at) AS first_account 
      FROM users
      GROUP BY user_guid) AS cleaned_users
```
  
We could then use a GROUP BY statement to count the number of unique early or late users:

```sql 
SELECT IF(cleaned_users.first_account<'2014-06-01','early_user','late_user') AS user_type,
       COUNT(cleaned_users.first_account)
FROM (SELECT user_guid, MIN(created_at) AS first_account 
      FROM users
      GROUP BY user_guid) AS cleaned_users
GROUP BY user_type
```

**Try it yourself:**


```python
%%sql
SELECT cleaned_users.user_guid as UserID,
       IF(cleaned_users.first_account<'2014-06-01','early_user','late_user') AS user_type,
    COUNT(*)
FROM (SELECT user_guid, MIN(created_at) AS first_account 
      FROM users
      GROUP BY user_guid) AS cleaned_users
GROUP BY user_type;
```

    2 rows affected.





<table>
    <tr>
        <th>UserID</th>
        <th>user_type</th>
        <th>COUNT(*)</th>
    </tr>
    <tr>
        <td>ce134492-7144-11e5-ba71-058fbc01cf0b</td>
        <td>early_user</td>
        <td>14470</td>
    </tr>
    <tr>
        <td>ce472c1c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>late_user</td>
        <td>18723</td>
    </tr>
</table>




```python

```

**Question 1: Write a query that will output distinct user_guids and their associated country of residence from the users table, excluding any user_guids or countries that have NULL values.  You should get 16,261 rows in your result.**


```python
%%sql
SELECT Distinct user_guid,country
FROM users
WHERE user_guid IS NOT NULL and country IS NOT NULL
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>user_guid</th>
        <th>country</th>
    </tr>
    <tr>
        <td>ce134492-7144-11e5-ba71-058fbc01cf0b</td>
        <td>CA</td>
    </tr>
    <tr>
        <td>ce134a78-7144-11e5-ba71-058fbc01cf0b</td>
        <td>US</td>
    </tr>
    <tr>
        <td>ce134be0-7144-11e5-ba71-058fbc01cf0b</td>
        <td>US</td>
    </tr>
    <tr>
        <td>ce134d16-7144-11e5-ba71-058fbc01cf0b</td>
        <td>US</td>
    </tr>
    <tr>
        <td>ce134e42-7144-11e5-ba71-058fbc01cf0b</td>
        <td>US</td>
    </tr>
</table>



**Question 2: Use an IF expression and the query you wrote in Question 1 as a subquery to determine the number of unique user_guids who reside in the United States (abbreviated "US") and outside of the US.**


```python
%%sql
SELECT usercoun.user_guid, COUNT(*), IF(usercoun.country='US','US', 'Out of US') 
FROM (SELECT Distinct user_guid,country
FROM users
WHERE user_guid IS NOT NULL and country IS NOT NULL) AS usercoun
GROUP BY 3;
```

    2 rows affected.





<table>
    <tr>
        <th>user_guid</th>
        <th>COUNT(*)</th>
        <th>IF(usercoun.country=&#x27;US&#x27;,&#x27;US&#x27;, &#x27;Out of US&#x27;)</th>
    </tr>
    <tr>
        <td>ce137a7a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>6905</td>
        <td>Out of US</td>
    </tr>
    <tr>
        <td>ce134e42-7144-11e5-ba71-058fbc01cf0b</td>
        <td>9356</td>
        <td>US</td>
    </tr>
</table>



Single IF expressions can only result in one of two specified outputs, but multiple IF expressions can be nested to result in more than two possible outputs.  <mark>When you nest IF expressions, it is important to encase each IF expression--as well as the entire IF expression put together--in parentheses.</mark>
    
For example, if you examine the entries contained in the non-US countries category, you will see that many users are associated with a country called "N/A." "N/A" is an abbreviation for "Not Applicable"; it is not a real country name.  We should separate these entries from the "Outside of the US" category we made earlier.  We could use a nested query to say whenever "country" does not equal "US", use the results of a second IF expression to determine whether the outputed value should be "Not Applicable" or "Outside US."  The IF expression would look like this:

```sql
IF(cleaned_users.country='US','In US', IF(cleaned_users.country='N/A','Not Applicable','Outside US'))
```

Since the second IF expression is in the position within the IF expression where you specify "value outputted if conditions are not met," its two possible outputs will only be considered if cleaned_users.country='US' is evaluated as false.

The full query to output the number of unique users in each of the three groups would be:

```sql 
SELECT IF(cleaned_users.country='US','In US', 
          IF(cleaned_users.country='N/A','Not Applicable','Outside US')) AS US_user, 
      count(cleaned_users.user_guid)   
FROM (SELECT DISTINCT user_guid, country 
      FROM users
      WHERE country IS NOT NULL) AS cleaned_users
GROUP BY US_user
```

**Try it yourself. You should get 5,642 unique user_guids in the "Not Applicable" category, and 1,263 users in the "Outside US" category.**


```python
%%sql
SELECT IF(cleaned_users.country='US','In US', 
          IF(cleaned_users.country='N/A','Not Applicable','Outside US')) AS US_user, 
      count(cleaned_users.user_guid)   
FROM (SELECT DISTINCT user_guid, country 
      FROM users
      WHERE country IS NOT NULL) AS cleaned_users
GROUP BY US_user;
```

    3 rows affected.





<table>
    <tr>
        <th>US_user</th>
        <th>count(cleaned_users.user_guid)</th>
    </tr>
    <tr>
        <td>In US</td>
        <td>9356</td>
    </tr>
    <tr>
        <td>Not Applicable</td>
        <td>5642</td>
    </tr>
    <tr>
        <td>Outside US</td>
        <td>1263</td>
    </tr>
</table>



<mark>The IF function is not supported by all database platforms, and some spell the function as IIF rather than IF, so be sure to double-check how the function works in the platform you are using.</mark>

If nested IF expressions seem confusing or hard to read, don't worry, there is a better function available for situations when you want to use conditional logic to output more than two groups.  That function is called CASE.

   
## 2. CASE expressions

The main purpose of CASE expressions is to return a singular value based on one or more conditional tests.  You can think of CASE expressions as an efficient way to write a set of IF and ELSEIF statements.  There are two viable syntaxes for CASE expressions.  If you need to manipulate values in a current column of your data, you would use this syntax:

<img src="https://duke.box.com/shared/static/bvyvscvvg9d1rjnov340gqyu85mhch9i.jpg" width=600 alt="CASE_Expression" />

Using this syntax, our nested IF statement from above could be written as:

```sql
SELECT CASE WHEN cleaned_users.country="US" THEN "In US"
            WHEN cleaned_users.country="N/A" THEN "Not Applicable"
            ELSE "Outside US"
            END AS US_user, 
      count(cleaned_users.user_guid)   
FROM (SELECT DISTINCT user_guid, country 
      FROM users
      WHERE country IS NOT NULL) AS cleaned_users
GROUP BY US_user
```

**Go ahead and try it:**


```python
%%sql
SELECT CASE WHEN cleaned_users.country="US" THEN "In US"
            WHEN cleaned_users.country="N/A" THEN "Not Applicable"
            ELSE "Outside US"
            END AS US_user, 
      count(cleaned_users.user_guid)   
FROM (SELECT DISTINCT user_guid, country 
      FROM users
      WHERE country IS NOT NULL) AS cleaned_users
GROUP BY US_user;
```

    3 rows affected.





<table>
    <tr>
        <th>US_user</th>
        <th>count(cleaned_users.user_guid)</th>
    </tr>
    <tr>
        <td>In US</td>
        <td>9356</td>
    </tr>
    <tr>
        <td>Not Applicable</td>
        <td>5642</td>
    </tr>
    <tr>
        <td>Outside US</td>
        <td>1263</td>
    </tr>
</table>



Since our query does not require manipulation of any of the values in the country column, though, we could also take advantage of this syntax, which is slightly more compact:

<img src="https://duke.box.com/shared/static/z9fezozm55wj5pz6slxscouxrcpq7bpz.jpg" width=600 alt="CASE_Value" />

Our query written in this syntax would look like this:

```sql
SELECT CASE cleaned_users.country
            WHEN "US" THEN "In US"
            WHEN "N/A" THEN "Not Applicable"
            ELSE "Outside US"
            END AS US_user, 
      count(cleaned_users.user_guid)   
FROM (SELECT DISTINCT user_guid, country 
      FROM users
      WHERE country IS NOT NULL) AS cleaned_users
GROUP BY US_user
```

**Try this query as well:**



```python
%%sql
SELECT CASE cleaned_users.country
            WHEN "US" THEN "In US"
            WHEN "N/A" THEN "Not Applicable"
            ELSE "Outside US"
            END AS US_user, 
      count(cleaned_users.user_guid)   
FROM (SELECT DISTINCT user_guid, country 
      FROM users
      WHERE country IS NOT NULL) AS cleaned_users
GROUP BY US_user;
```

    3 rows affected.





<table>
    <tr>
        <th>US_user</th>
        <th>count(cleaned_users.user_guid)</th>
    </tr>
    <tr>
        <td>In US</td>
        <td>9356</td>
    </tr>
    <tr>
        <td>Not Applicable</td>
        <td>5642</td>
    </tr>
    <tr>
        <td>Outside US</td>
        <td>1263</td>
    </tr>
</table>



There are a couple of things to know about CASE expressions:
   
+ Make sure to include the word END at the end of the expression
+ CASE expressions do not require parentheses
+ ELSE expressions are optional
+ If an ELSE expression is omitted, NULL values will be outputted for all rows that do not meet any of the conditions stated explicitly in the expression
+ CASE expressions can be used anywhere in a SQL statement, including in GROUP BY, HAVING, and ORDER BY clauses or the SELECT column list.

You will find that CASE statements are useful in many contexts. For example, they can be used to rename or revise values in a column.

**Question 3: Write a query using a CASE statement that outputs 3 columns: dog_guid, dog_fixed, and a third column that reads "neutered" every time there is a 1 in the "dog_fixed" column of dogs, "not neutered" every time there is a value of 0 in the "dog_fixed" column of dogs, and "NULL" every time there is a value of anything else in the "dog_fixed" column.  Limit your results for troubleshooting purposes.**



```python
%%sql
SELECT dog_guid, dog_fixed, CASE dog_fixed
            when "1" then "neutered"
            when "0" then "not neutered"
            ELSE "null"
        END
FROM dogs
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>dog_guid</th>
        <th>dog_fixed</th>
        <th>CASE dog_fixed<br>            when &quot;1&quot; then &quot;neutered&quot;<br>            when &quot;0&quot; then &quot;not neutered&quot;<br>            ELSE &quot;null&quot;<br>        END</th>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>neutered</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>neutered</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0</td>
        <td>not neutered</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0</td>
        <td>not neutered</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>0</td>
        <td>not neutered</td>
    </tr>
</table>



You can also use CASE statements to standardize or combine several values into one.  

**Question 4: We learned that NULL values should be treated the same as "0" values in the exclude columns of the dogs and users tables.  Write a query using a CASE statement that outputs 3 columns: dog_guid, exclude, and a third column that reads "exclude" every time there is a 1 in the "exclude" column of dogs and "keep" every time there is any other value in the exclude column. Limit your results for troubleshooting purposes.**




```python
%%sql
SELECT dog_guid,exclude, CASE exclude
           WHEN "1" THEN "exclude"
           ELSE "keep"
        END
FROM dogs
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>dog_guid</th>
        <th>exclude</th>
        <th>CASE exclude<br>           WHEN &quot;1&quot; THEN &quot;exclude&quot;<br>           ELSE &quot;keep&quot;<br>        END</th>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>keep</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>keep</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>keep</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>keep</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>exclude</td>
    </tr>
</table>



**Question 5: Re-write your query from Question 4 using an IF statement instead of a CASE statement.**


```python
%%sql
SELECT dog_guid,exclude, IF(exclude='1','exclude','keep')
FROM dogs
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>dog_guid</th>
        <th>exclude</th>
        <th>IF(exclude=&#x27;1&#x27;,&#x27;exclude&#x27;,&#x27;keep&#x27;)</th>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>keep</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>keep</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>keep</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>None</td>
        <td>keep</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>1</td>
        <td>exclude</td>
    </tr>
</table>



Case expressions are also useful for breaking values in a column up into multiple groups that meet specific criteria or that have specific ranges of values.

**Question 6: Write a query that uses a CASE expression to output 3 columns: dog_guid, weight, and a third column that reads...     
"very small" when a dog's weight is 1-10 pounds     
"small" when a dog's weight is greater than 10 pounds to 30 pounds     
"medium" when a dog's weight is greater than 30 pounds to 50 pounds     
"large" when a dog's weight is greater than 50 pounds to 85 pounds     
"very large" when a dog's weight is greater than 85 pounds      
Limit your results for troubleshooting purposes.**

**Remember that when you use AND to define values between two boundaries, you need to include the variable name in all clauses that define the conditions of the values you want to extract.  In other words, you could use this combined clause in your query: 
“WHEN weight>10 AND weight<=30 THEN "small"
…but this combined clause would cause an error:
“WHEN weight>10 AND <=30 THEN "small"**


```python
%%sql
SELECT dog_guid,weight, CASE 
           WHEN weight>1 AND weight<=10 THEN "very small"
           WHEN weight>10 AND weight<=30 THEN "small"
        WHEN weight>30 AND weight<=50 THEN "medium"
        WHEN weight>50 AND weight<=85 THEN "large"
        WHEN weight>85 THEN "very large"
        
        END
FROM dogs
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>dog_guid</th>
        <th>weight</th>
        <th>CASE <br>           WHEN weight&gt;1 AND weight&lt;=10 THEN &quot;very small&quot;<br>           WHEN weight&gt;10 AND weight&lt;=30 THEN &quot;small&quot;<br>        WHEN weight&gt;30 AND weight&lt;=50 THEN &quot;medium&quot;<br>        WHEN weight&gt;50 AND weight&lt;=85 THEN &quot;large&quot;<br>        WHEN weight&gt;85 THEN &quot;very </th>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>50</td>
        <td>medium</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>20</td>
        <td>small</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>70</td>
        <td>large</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>70</td>
        <td>large</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>190</td>
        <td>very large</td>
    </tr>
</table>



## 3. Pay attention to the order of operations within logical expressions

As you started to see with the query you wrote in Question 6, CASE expressions often end up needing multiple AND and OR operators to accurately describe the logical conditions you want to impose on the groups in your queries.  You must pay attention to the order in which these operators are included in your logical expressions, because unless parentheses are included, the NOT operator is always evaluated before an AND operator, and an AND operator is always evaluated before the OR operator.  

<img src="https://duke.box.com/shared/static/uhrmp6vkxznn9zjgezfmxhnbfjaa8c90.jpg" width=200 alt="CASE_Value" />

When parentheses are included, the expressions within the parenthese are evaluated first.  That means this expression:

```sql
CASE WHEN "condition 1" OR "condition 2" AND "condition 3"...
```
will lead to different results than this expression:
   
```sql
CASE WHEN "condition 3" AND "condition 1" OR "condition 2"...
```
   
or this expression:
```sql
CASE WHEN ("condition 1" OR "condition 2") AND "condition 3"...
```
   
In the first case you will get rows that meet condition 2 and 3, or condition 1.  In the second case you will get rows that meet condition 1 and 3, or condition 2.  In the third case, you will get rows that meet condition 1 or 2, and condition 3.

Let's see a concrete example of how the order in which logical operators are evaluated affects query results.  

**Question 7: How many distinct dog_guids are found in group 1 using this query?**
    
```sql
SELECT COUNT(DISTINCT dog_guid), 
CASE WHEN breed_group='Sporting' OR breed_group='Herding' AND exclude!='1' THEN "group 1"
     ELSE "everything else"
     END AS groups
FROM dogs
GROUP BY groups
```


```python
%%sql
SELECT COUNT(DISTINCT dog_guid), 
CASE WHEN breed_group='Sporting' OR breed_group='Herding' AND exclude='1' THEN "group 1"
     ELSE "everything else"
     END AS groups
FROM dogs
GROUP BY groups
```

    2 rows affected.





<table>
    <tr>
        <th>COUNT(DISTINCT dog_guid)</th>
        <th>groups</th>
    </tr>
    <tr>
        <td>30100</td>
        <td>everything else</td>
    </tr>
    <tr>
        <td>4950</td>
        <td>group 1</td>
    </tr>
</table>



**Question 8: How many distinct dog_guids are found in group 1 using this query?**
    
```sql
SELECT COUNT(DISTINCT dog_guid), 
CASE WHEN exclude!='1' AND breed_group='Sporting' OR breed_group='Herding' THEN "group 1"
     ELSE "everything else"
     END AS group_name
FROM dogs
GROUP BY group_name
```



```python
%%sql
SELECT COUNT(DISTINCT dog_guid), 
CASE WHEN exclude='1' AND breed_group='Sporting' OR breed_group='Herding' THEN "group 1"
     ELSE "everything else"
     END AS group_name
FROM dogs
GROUP BY group_name

```

    2 rows affected.





<table>
    <tr>
        <th>COUNT(DISTINCT dog_guid)</th>
        <th>group_name</th>
    </tr>
    <tr>
        <td>31490</td>
        <td>everything else</td>
    </tr>
    <tr>
        <td>3560</td>
        <td>group 1</td>
    </tr>
</table>



**Question 9: How many distinct dog_guids are found in group 1 using this query?**

```sql
SELECT COUNT(DISTINCT dog_guid), 
CASE WHEN exclude!='1' AND (breed_group='Sporting' OR breed_group='Herding') THEN "group 1"
     ELSE "everything else"
     END AS group_name
FROM dogs
GROUP BY group_name
```


```python
%%sql
SELECT COUNT(DISTINCT dog_guid), 
CASE WHEN exclude='1' AND (breed_group='Sporting' OR breed_group='Herding') THEN "group 1"
     ELSE "everything else"
     END AS group_name
FROM dogs
GROUP BY group_name
```

    2 rows affected.





<table>
    <tr>
        <th>COUNT(DISTINCT dog_guid)</th>
        <th>group_name</th>
    </tr>
    <tr>
        <td>34826</td>
        <td>everything else</td>
    </tr>
    <tr>
        <td>224</td>
        <td>group 1</td>
    </tr>
</table>



<mark> **So make sure you always pay attention to the order in which your logical operators are listed in your expressions, and whenever possible, include parentheses to ensure that the expressions are evaluated in the way you intend!**</mark>

## Let's practice some more IF and CASE statements

<img src="https://duke.box.com/shared/static/p2eucjdttai08eeo7davbpfgqi3zrew0.jpg" width=600 alt="SELECT FROM WHERE" />

**Question 10: For each dog_guid, output its dog_guid, breed_type, number of completed tests, and use an IF statement to include an extra column that reads "Pure_Breed" whenever breed_type equals 'Pure Breed" and "Not_Pure_Breed" whenever breed_type equals anything else. LIMIT your output to 50 rows for troubleshooting.  HINT: you will need to use a join to complete this query.**


```python
%%sql
SELECT d.dog_guid,d.breed_type,count(c.created_at),
if(d.breed_type='pure breed','pure_breed','not_pure_breed')
FROM dogs d LEFT JOIN complete_tests c
ON d.dog_guid = c.dog_guid
GROUP BY 1
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>dog_guid</th>
        <th>breed_type</th>
        <th>count(c.created_at)</th>
        <th>if(d.breed_type=&#x27;pure breed&#x27;,&#x27;pure_breed&#x27;,&#x27;not_pure_breed&#x27;)</th>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pure Breed</td>
        <td>21</td>
        <td>pure_breed</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pure Breed</td>
        <td>20</td>
        <td>pure_breed</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pure Breed</td>
        <td>2</td>
        <td>pure_breed</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pure Breed</td>
        <td>11</td>
        <td>pure_breed</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pure Breed</td>
        <td>31</td>
        <td>pure_breed</td>
    </tr>
</table>




```python
%%sql
SELECT d.dog_guid AS dogID, d.breed_type AS breed_type, count(c.created_at) AS
numtests,
IF(d.breed_type='Pure Breed','pure_breed', 'not_pure_breed') AS pure_breed
FROM dogs d, complete_tests c
WHERE d.dog_guid=c.dog_guid
GROUP BY dogID, breed_type, pure_breed
LIMIT 5;
```

    5 rows affected.





<table>
    <tr>
        <th>dogID</th>
        <th>breed_type</th>
        <th>numtests</th>
        <th>pure_breed</th>
    </tr>
    <tr>
        <td>fd27b272-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pure Breed</td>
        <td>21</td>
        <td>pure_breed</td>
    </tr>
    <tr>
        <td>fd27b5ba-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pure Breed</td>
        <td>20</td>
        <td>pure_breed</td>
    </tr>
    <tr>
        <td>fd27b6b4-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pure Breed</td>
        <td>2</td>
        <td>pure_breed</td>
    </tr>
    <tr>
        <td>fd27b79a-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pure Breed</td>
        <td>11</td>
        <td>pure_breed</td>
    </tr>
    <tr>
        <td>fd27b86c-7144-11e5-ba71-058fbc01cf0b</td>
        <td>Pure Breed</td>
        <td>31</td>
        <td>pure_breed</td>
    </tr>
</table>



**Question 11: Write a query that uses a CASE statement to report the number of unique user_guids associated with customers who live in the United States and who are in the following groups of states:**

**Group 1: New York (abbreviated "NY") or New Jersey (abbreviated "NJ")    
Group 2: North Carolina (abbreviated "NC") or South Carolina (abbreviated "SC")    
Group 3: California (abbreviated "CA")    
Group 4: All other states with non-null values**

**You should find 898 unique user_guids in Group1.**




```python
%%sql
SELECT COUNT(DISTINCT user_guid),
CASE
WHEN (state="NY" OR state="NJ") THEN "Group 1-NY/NJ"
WHEN (state="NC" OR state="SC") THEN "Group 2-NC/SC"
WHEN state="CA" THEN "Group 3-CA"
ELSE "Group 4-Other"
END 
FROM users
WHERE country="US" AND state IS NOT NULL
GROUP BY 2;
```

    4 rows affected.





<table>
    <tr>
        <th>COUNT(DISTINCT user_guid)</th>
        <th>CASE<br>WHEN (state=&quot;NY&quot; OR state=&quot;NJ&quot;) THEN &quot;Group 1-NY/NJ&quot;<br>WHEN (state=&quot;NC&quot; OR state=&quot;SC&quot;) THEN &quot;Group 2-NC/SC&quot;<br>WHEN state=&quot;CA&quot; THEN &quot;Group 3-CA&quot;<br>ELSE &quot;Group 4-Other&quot;<br>END</th>
    </tr>
    <tr>
        <td>898</td>
        <td>Group 1-NY/NJ</td>
    </tr>
    <tr>
        <td>653</td>
        <td>Group 2-NC/SC</td>
    </tr>
    <tr>
        <td>1417</td>
        <td>Group 3-CA</td>
    </tr>
    <tr>
        <td>6388</td>
        <td>Group 4-Other</td>
    </tr>
</table>



**Question 12: Write a query that allows you to determine how many unique dog_guids are associated with dogs who are DNA tested and have either stargazer or socialite personality dimensions.  Your answer should be 70.**


```python
%%sql
SELECT COUNT(DISTINCT dog_guid)
FROM dogs
WHERE dna_tested=1 AND (dimension='stargazer' OR dimension='socialite');
```

    1 rows affected.





<table>
    <tr>
        <th>COUNT(DISTINCT dog_guid)</th>
    </tr>
    <tr>
        <td>70</td>
    </tr>
</table>



**Feel free to practice any other queries you like here!**


```python

```
