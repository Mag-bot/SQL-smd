/*Query 1 - Query with 6 results_What family_friendly movie category is watched most?*/
WITH t1 AS (
  SELECT f.title film_title, c.name category_name
  FROM film f
  JOIN film_category fcat
  ON f.film_id = fcat.film_id
  JOIN category c
  ON c.category_id = fcat.category_id
  WHERE c.name = 'Animation' OR c.name = 'Children' OR c.name = 'Classics' OR c.name = 'Comedy' OR c.name = 'Family' OR c.name = 'Music'
  ORDER BY 1)
SELECT category_name, COUNT(category_name) AS category_rented_amt
FROM t1
GROUP BY 1
ORDER BY 1;



/*Query 2 - Who are the top ten “family” moving renting customers?*/
SELECT c.first_name || ' ' || c.last_name AS customer, SUM(c.customer_id) total_rentals
FROM rental r
JOIN customer c
ON c.customer_id = r.rental_id
JOIN inventory i
ON i.inventory_id = r.rental_id
JOIN film f
ON f.film_id = i.inventory_id
JOIN film_category fcat
ON fcat.film_id = f.film_id
JOIN category cat
ON cat.category_id = fcat.category_id
WHERE cat.name = 'Animation' OR cat.name = 'Children' OR cat.name = 'Classics'         OR cat.name = 'Comedy' OR cat.name = 'Family' OR cat.name = 'Music'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;



/*Query 3 - What time of the day of week are there the most rentals?*/
WITH t1 AS (
          SELECT DATE_PART('dow', rental_date) AS day_of_week,
  NTILE(3) OVER (ORDER BY rental_date) AS ntile
        FROM rental),

    t2 AS (
     SELECT t1.day_of_week, t1.ntile AS time_of_day, SUM(t1.day_of_week) num_rental
        FROM  t1
        GROUP BY 1,2
        ORDER BY 1,2),

  t3 AS (
    SELECT CASE WHEN t2.day_of_week = 0 THEN 'Sunday' WHEN t2.day_of_week = 1 THEN 'Monday' WHEN t2.day_of_week = 2 THEN 'Tuesday' WHEN t2.day_of_week = 3 THEN 'Wednesday' WHEN t2.day_of_week = 4 THEN 'Thursday' WHEN t2.day_of_week = 5 THEN 'Friday' ELSE 'Saturday' END AS day_of_week,
    CASE WHEN t2.time_of_day = 1 THEN 'morning' WHEN t2.time_of_day = 2 THEN 'afternoon' ELSE 'evening' END AS time_of_day, t2.num_rental AS num_rental
                FROM  t2)

SELECT t3.day_of_week, t3.time_of_day, MAX(t3.num_rental) AS num_rentals
FROM t3
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 3;



/*Query 4 - Top 10 customers & their total monthly payments.*/
WITH t2 AS (
  SELECT customer, payment_month, COUNT(payment_month) AS payments_per_month, t1.year as year, t1.month as month, SUM(t1.sum_paid) AS sum_paid
      FROM (SELECT c.first_name || ' ' || c.last_name customer,
      DATE_TRUNC('month', p.payment_date) AS payment_month,
      DATE_PART('year', p.payment_date) AS year,
      DATE_PART('month', p.payment_date) AS month,
      p.amount sum_paid
        FROM customer c
        JOIN payment p
        ON c.customer_id = p.customer_id) t1
      GROUP BY 1,2,4,5
      ORDER BY 1),

t3 AS (
  SELECT c.first_name || ' ' || c.last_name customer,
          SUM(p.amount) total_paid
  FROM customer c
  JOIN payment p
  ON p.customer_id = c.customer_id
  GROUP BY 1
  ORDER BY 2 DESC
  LIMIT 10)

SELECT t3.customer, t2.year, t2.month, t2.payments_per_month, t2.sum_paid,
        SUM(t2.sum_paid) OVER (PARTITION BY t3.customer ORDER BY t2.month, t2.sum_paid) AS accumulated_amt_paid
FROM t2
JOIN t3
ON t2.customer = t3.customer
ORDER BY 1,3,6 DESC;
