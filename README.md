# 🍿SQL-DVD-Rental

Utilizing the PostgreSQL and pgAdmin in this demonstration, showcasing various SQL queries on the DVD Rental database.

*Note: In the results I have applied a LIMIT of 5 for readability.* 

# 📚Table of Contents
- [Database ER-Model](#database-er-model)
- [Questions, Solutions and Results](#questions-solutions-and-results)
    - [1. Who are our top spenders?](#1-who-are-our-top-spenders)
    - [2. How are films priced and how often are they rented?](#2-how-are-films-priced-and-how-often-are-they-rented)
    - [3. Are there overdue films from before 2006?](#3-are-there-overdue-films-from-before-2006)
    - [4. Which customers made transactions with us in mid-February 2007?](#4-which-customers-made-transactions-with-us-in-mid-february-2007)
    - [5. Can we determine the latest transaction amount for each customer?](#5-can-we-determine-the-latest-transaction-amount-for-each-customer)

## Database ER-Model
The data I have used in this demonstration can be found [here](https://www.postgresqltutorial.com/postgresql-getting-started/postgresql-sample-database/)

All though the data set contains many tables and relationships I have chosen to focus on only a few for this demonstration.
![Link to ER-Model Image](https://www.postgresqltutorial.com/wp-content/uploads/2018/03/dvd-rental-sample-database-diagram.png)

## Questions, Solutions and Results

### 1. Who are our top spenders?

**Question:** Can we identify customers who have spent more than $50, providing details of their name and total amount spent?

```sql
SELECT c.first_name, c.last_name, SUM(p.amount) AS total_spent
FROM customer c
JOIN payment p ON c.customer_id = p.customer_id
GROUP BY c.first_name, c.last_name
HAVING SUM(p.amount) > 50
ORDER BY total_spent DESC;
````

**Steps:**
1. Merge customer and payment records using customer_id.
2. Group the results by each customer.
3. Filter to show only those who have spent over $50.
4. Arrange the data by spending in a descending order.

**Result:**
| first_name | last_name | total_spent |
|------------|-----------|-------------|
| Eleanor    | Hunt      | 211.55      |
| Karl       | Seal      | 208.58      |
| Marion     | Snyder    | 194.61      |
| Rhonda     | Kennedy   | 191.62      |
| Clara      | Shaw      | 189.60      |

___

### 2. How are films priced and how often are they rented?

**Description:** How do we classify films based on their rental rates, and how frequently are films in each category rented?

```sql
SELECT f.title, 
       CASE 
           WHEN f.rental_rate < 3 THEN 'Low'
           WHEN f.rental_rate BETWEEN 3 AND 5 THEN 'Medium'
           ELSE 'High'
       END AS price_category,
       (SELECT COUNT(*) FROM rental r WHERE r.inventory_id = f.film_id) AS rental_count
FROM film f;
````

**Steps:**
1. Apply categorization to films based on their rental rate.
2. Use a sub-query to determine rental frequency for each title.
3. Display the title and its assigned category.

**Result**:
| title             | price_category | rental_count |
|-------------------|----------------|--------------|
| Chamber Italian   | Medium         | 5            |
| Grosse Wonderful  | Medium         | 4            |
| Airport Pollock   | Medium         | 2            |
| Bright Encounters | Medium         | 4            |
| Academy Dinosaur  | Low            | 3            |

___

### 3. Are there overdue films from before 2006?
**Question:** Which customers have films, rented before '2006-01-01', that are still not returned?

```sql
SELECT c.first_name, c.last_name, f.title, r.rental_date
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE r.return_date IS NULL AND r.rental_date < '2006-01-01';
````

**Steps:**
1. Combine records from customer, rental, inventory, and film databases.
2. Filter the results to show only unreturned films and those rented before '2006-01-01'.

**Result:**
| first_name | last_name | title             | rental_date           |
|------------|-----------|--------------------|-----------------------|
| Dwayne     | Olvera    | Academy Dinosaur  | 2005-08-21 00:30:32   |

___

### 4. Which customers made transactions with us in mid-February 2007?
**Question:** Can we pinpoint unique customers who made payments between '2007-02-15' and '2007-02-20'?

```sql
SELECT DISTINCT c.first_name, c.last_name
FROM customer c
INNER JOIN payment p ON c.customer_id = p.customer_id
WHERE p.payment_date BETWEEN '2007-02-15' AND '2007-02-20';
````

**Steps:**
1. Merge data from customer and payment records.
2. Screen the results for payments made within the specified timeframe.

**Result:**
| first_name | last_name |
|------------|-----------|
| Aaron      | Selby     |
| Adrian     | Clary     |
| Agnes      | Bishop    |
| Alan       | Kahn      |
| Albert     | Crouse    |

___

### 5. Can we determine the latest transaction amount for each customer?
**Question:** How can we generate a list of customers with the value of their most recent transaction?

```sql
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

**Steps:**
1. Formulate a Common Table Expression (CTE) termed "RankedPayments".
2. Use the RANK() function to organize payment records for each client based on the date.
3. Focus on the most up-to-date transaction for every individual customer.


**Result:**
| first_name  | last_name | amount |
|-------------|-----------|--------|
| Mary        | Smith     | 2.99   |
| Patricia    | Johnson   | 6.99   |
| Linda       | Williams  | 3.99   |
| Barbara     | Jones     | 5.99   |
| Elizabeth   | Brown     | 0.99   |





