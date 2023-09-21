# SQL-DVD-Rental
Been using PostgreSQL Sample Database DVD Rental for this demonstration.

Here is the ER-model for the database:



![image](https://www.postgresqltutorial.com/wp-content/uploads/2018/03/dvd-rental-sample-database-diagram.png)


Explanation: This snippet lists customers who have spent more than $50 in total. It displays their first name, last name, and the total amount spent, sorted in descending order based on the total amount spent.
````sql
SELECT c.first_name, c.last_name, SUM(p.amount) AS total_spent
FROM customer c
JOIN payment p ON c.customer_id = p.customer_id
GROUP BY c.first_name, c.last_name
HAVING SUM(p.amount) > 50
ORDER BY total_spent DESC;
````


Explanation: This snippet lists all films, categorizes them based on their rental rate into 'Low', 'Medium', or 'High' categories, and also provides a count of how many times each film has been rented.
````sql
SELECT f.title, 
       CASE 
           WHEN f.rental_rate < 3 THEN 'Low'
           WHEN f.rental_rate BETWEEN 3 AND 5 THEN 'Medium'
           ELSE 'High'
       END AS price_category,
       (SELECT COUNT(*) FROM rental r WHERE r.inventory_id = f.film_id) AS rental_count
FROM film f;
````


Explanation: This snippet retrieves a list of customers, along with films they've rented, which they haven't returned yet and were rented before '2006-01-01'.
````sql
SELECT c.first_name, c.last_name, f.title, r.rental_date
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE r.return_date IS NULL AND r.rental_date < '2006-01-01';
````

Explanation: This snippet provides a list of unique customers who made at least one payment between '2007-02-15' and '2007-02-20'.
````sql
SELECT DISTINCT c.first_name, c.last_name
FROM customer c
INNER JOIN payment p ON c.customer_id = p.customer_id
WHERE p.payment_date BETWEEN '2007-02-15' AND '2007-02-20';
````

Explanation: This snippet provides a list of customers along with their most recent payment amount. It first ranks payments for each customer based on the payment date in descending order and then filters out the most recent payment (i.e., rank = 1) for each customer.
````sql
WITH RankedPayments AS (
    SELECT c.first_name, c.last_name, p.amount,
           RANK() OVER (PARTITION BY c.customer_id ORDER BY p.payment_date DESC) as rnk
    FROM customer c
    JOIN payment p ON c.customer_id = p.customer_id
)
SELECT first_name, last_name, amount
FROM RankedPayments
WHERE rnk = 1;
````
