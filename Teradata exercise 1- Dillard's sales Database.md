
# EXERCISE 1

Question 1
(a) Use COUNT and DISTINCT to determine how many distinct skus there are in pairs of the skuinfo, skstinfo, and trnsact tables. Which skus are common to pairs of tables, or unique to specific tables?


```python
SELECT COUNT(DISTINCT a.sku)
FROM skuinfo a JOIN skstinfo b
ON a.sku = b.sku;
    
SELECT COUNT(DISTINCT a.sku)
FROM skuinfo a JOIN trnsact b
ON a.sku = b.sku;
    
SELECT COUNT(DISTINCT a.sku)
FROM skstinfo a JOIN trnsact b
ON a.sku = b.sku;
```

(b) Use COUNT to determine how many instances there are of each sku associated with each store in the skstinfo table and the trnsact table?


```python
SELECT sku, store, COUNT(sku)
FROM skstinfo
GROUP BY sku, store;
```

We find out that there are multiple instances of every sku/store combination in the trnsact table, but only one instance of every sku/store combination in the skstinfo table. Therefore we could join the trnsact and skstinfo tables, but need to join them on both of the following conditions: trnsact.sku= skstinfo.sku AND trnsact.store= skstinfo.store.

# EXERCISE 2

Which stores are common to all four tables, or unique to specific tables?


```python
SELECT a.store, b.store, c.store, d.store
FROM strinfo a 
  LEFT JOIN skstinfo b 
    ON a.store = b.store
  LEFT JOIN trnsact c 
    ON a.store = c.store
  LEFT JOIN store_msa d
    ON c.store = d.store
```

# EXERCISE 3

It turns out there are many skus in the trnsact table that are not in the skstinfo table. As a consequence, we will not be able to complete many desirable analyses of Dillard’s profit, as opposed to revenue, because we do not have the cost information for all the skus in the transact table (recall that profit = revenue - cost).

Examine some of the rows in the trnsact table that are not in the skstinfo table; can you find any common features that could explain why the cost information is missing?


```python
SELECT distinct a.sku, a.store
FROM trnsact a 
  LEFT JOIN skstinfo b
    ON a.sku=b.sku AND a.store = b.store
WHERE b.sku IS NULL 
GROUP BY a.sku, a.store;
```

That leaves exactly 17,816,793 sku-store combinations found in the transactions table that are not listed in the master skstinfo table. 

# EXERCISE 4

Although we can’t complete all the analyses we’d like to on Dillard’s profit, we can look at general trends. What is Dillard’s average profit per day?

PROFIT = trnsact.amt - (trnsact.quantity * skstinfo.cost)

Further, since we want to know the average profit, we can find the number of days by diving the profit by count(distinct saledate).

Overall, we can build the rest of the query around it like so:


```python
SELECT SUM(a.amt - a.quantity*b.cost)/COUNT(DISTINCT a.saledate)
FROM trnsact a
  LEFT JOIN SKSTINFO b
    ON a.sku = b.sku AND a.store = b.store
WHERE a.stype = 'P'; 
```

# EXERCISE 5

On what day was the total value (in $) of returned goods the greatest?


```python
SELECT saledate, sum(amt)  
FROM trnsact
WHERE stype = 'R'
GROUP BY saledate 
ORDER BY sum(amt) DESC;
```

How many departments have more than 100 brands associated with them, and what are their descriptions?


```python
SELECT DISTINCT a.dept, b.deptdesc, count(distinct a.brand) 
FROM skuinfo a
  LEFT JOIN deptinfo b
    ON  a.dept=b.dept 
GROUP BY a.dept, b.deptdesc
HAVING count(distinct a.brand) > 100;
```

Write a query that retrieves the department descriptions of each of the skus in the skstinfo table.


```python
SELECT a.sku, c.deptdesc
FROM skstinfo a 
  LEFT JOIN skuinfo b 
    ON a.sku = b.sku 
  LEFT JOIN deptinfo c
    ON b.dept = c.dept
SAMPLE 100; 
```

In what state and city is the store that had the greatest total revenue during the time period monitored in our dataset?


```python
SELECT b.state, b.city, SUM(a.amt) -- no need to include sum(a.amt), but this is good for checking.
FROM strinfo b
  LEFT JOIN trnsact a
    ON a.store = b.store
WHERE stype = 'P'
GROUP BY b.state, b.city
ORDER BY SUM(a.amt) DESC;
```
