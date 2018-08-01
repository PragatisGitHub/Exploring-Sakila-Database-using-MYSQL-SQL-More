# MYSQL-SQL-More
Exploring "sakila" database using SQL queries and the DML commands that involve various clauses, functions, sub queries & joins.

Sakila database has the following tables:
	'actor'
	'actor_info'
	'address'
	'category'
	'city'
	'country'
	'customer'
	'customer_list'
	'film'
	'film_actor'
	'film_category'
	'film_list'
	'film_text'
	'inventory'
	'language'
	'nicer_but_slower_film_list'
	'payment'
	'rental'
	'sales_by_film_category'
	'sales_by_store'
	'staff'
	'staff_list'
	'store'

### Queries are created to respond below requirements:
* Command to use the database - sakila
USE sakila
;

* 1a. Display the first and last names of all actors from the table `actor`.
SELECT first_name, last_name 
FROM actor
;

* 1b. Display the first and last name of each actor in a single column in upper case letters. Name the column `Actor Name`.
SELECT upper(concat(first_name," ",last_name)) AS Actor_Name 
FROM actor
;

* 2a. You need to find the ID number, first name, and last name of an actor, of whom you know only the first name, "Joe." What is one query would you use to obtain this information?
SELECT actor_id, first_name, last_name
FROM actor
WHERE first_name = "Joe"
;

* 2b. Find all actors whose last name contain the letters `GEN`:
SELECT concat(first_name," ",last_name) as "Actor_Name"
FROM actor
WHERE last_name LIKE "%gen%"
;

* 2c. Find all actors whose last names contain the letters `LI`. This time, order the rows by last name and first name, in that order:
SELECT last_name, first_name
FROM actor
WHERE last_name LIKE "%li%"
ORDER BY last_name,first_name
;

* 2d. Using `IN`,display the `country_id` and `country` columns of the following countries: Afghanistan, Bangladesh, and China:
SELECT country_id, country 
FROM country
WHERE country IN ("Afghanistan", "Bangladesh", "China")
;

* 3a. Add a `middle_name` column to the table `actor`. Position it between `first_name` and `last_name`. Hint: you will need to specify the data type.*/
* Verify before alter
DESC actor
;

ALTER TABLE actor
ADD COLUMN middle_name VARCHAR(45) AFTER first_name
;

* Verify after alter
DESC actor
;

* 3b. You realize that some of these actors have tremendously long last names. Change the data type of the `middle_name` column to `blobs`.
ALTER TABLE actor
MODIFY middle_name BLOB
;

* Verify after alter
DESC actor
;

* 3c. Now delete the `middle_name` column.
ALTER TABLE actor
DROP COLUMN middle_name
;

* Verify after alter
DESC actor
;

* 4a. List the last names of actors, as well as how many actors have that last name.
SELECT last_name, COUNT(last_name) AS "Last Name Count"
FROM actor 
GROUP BY last_name
;

* 4b. List last names of actors and the number of actors who have that last name, but only for names that are shared by at least two actors.
SELECT last_name, COUNT(last_name) AS "Last Name Count >=2"
FROM actor 
GROUP BY last_name
HAVING COUNT(last_name) >=2
;

* 4c. Oh, no! The actor `HARPO WILLIAMS` was accidentally entered in the `actor` table as `GROUCHO WILLIAMS`, the name of Harpo's second cousin's husband's yoga teacher. Write a query to fix the record.
* Verify before update
SELECT * 
FROM actor 
WHERE first_name = "Groucho" AND last_name = "Williams"
;

UPDATE actor
SET first_name = "HARPO"
WHERE first_name = "Groucho" AND last_name = "Williams"
;

* Verify after update
SELECT * 
FROM actor WHERE first_name = "Groucho" AND last_name = "Williams"
;
SELECT * 
FROM actor WHERE first_name = "Harpo" AND last_name = "Williams"
;
* 4d. Perhaps we were too hasty in changing `GROUCHO` to `HARPO`. It turns out that `GROUCHO` was the correct name after all! In a single query, if the first name of the actor is currently `HARPO`, change it to `GROUCHO`. Otherwise,change the first name to `MUCHO GROUCHO`, as that is exactly what the actor will be with the grievous error. BE CAREFUL NOT TO CHANGE THE FIRST NAME OF EVERY ACTOR TO `MUCHO GROUCHO`, HOWEVER! (Hint: update the record using a unique identifier.)
* Verify before update
SELECT * 
FROM actor WHERE first_name = "Groucho"
;
SELECT * 
FROM actor WHERE first_name = "Harpo"
;
SELECT * 
FROM actor WHERE last_name = "Williams"
;

UPDATE actor
SET first_name = "GROUCHO"
WHERE first_name = "Harpo" AND actor_id IN (72,137,172)
;

* Verify after update
SELECT * 
FROM actor WHERE first_name = "Groucho"
;
SELECT * 
FROM actor WHERE first_name = "Harpo"
;
SELECT * 
FROM actor WHERE last_name = "Williams"
;

UPDATE actor
SET first_name = "Groucho Mucho"
WHERE first_name != "Groucho" AND actor_id IN (72,137,172)
;
*Verify after update
SELECT * 
FROM actor WHERE first_name = "Groucho"
;
SELECT * 
FROM actor WHERE first_name = "Harpo"
;
SELECT * 
FROM actor WHERE last_name = "Williams"
;
* The above excercise (4d) was discussed to be left

* 5a. You cannot locate the schema of the `address` table. Which query would you use to re-create it?
SHOW CREATE TABLE address
;
* Above query gives the query that was used to create the schema of the address table. We can use the same query to re-create the schema.
CREATE TABLE `address` (
   `address_id` smallint(5) unsigned NOT NULL AUTO_INCREMENT,
   `address` varchar(50) NOT NULL,
   `address2` varchar(50) DEFAULT NULL,
   `district` varchar(20) NOT NULL,
   `city_id` smallint(5) unsigned NOT NULL,
   `postal_code` varchar(10) DEFAULT NULL,
   `phone` varchar(20) NOT NULL,
   `location` geometry NOT NULL,
   `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
   PRIMARY KEY (`address_id`),
   KEY `idx_fk_city_id` (`city_id`),
   SPATIAL KEY `idx_location` (`location`),
   CONSTRAINT `fk_address_city` FOREIGN KEY (`city_id`) REFERENCES `city` (`city_id`) ON UPDATE CASCADE
 ) ENGINE=InnoDB AUTO_INCREMENT=606 DEFAULT CHARSET=utf8

* 6a. Use `JOIN` to display the first and last names, as well as the address, of each staff member. Use the tables `staff` and `address`.
select first_name, last_name, address
from staff 
join address 
on staff.address_id = address.address_id
;

* 6b. Use `JOIN` to display the total amount rung up by each staff member in August of 2005. Use tables `staff` and `payment`.
SELECT staff.staff_id, first_name, last_name,
SUM(amount) as "Total Amount Rung Up In August 2005"
FROM staff 
JOIN payment 
ON staff.staff_id = payment.staff_id
WHERE payment_date >= "2005-08-01" and payment_date < "2005-09-01"
GROUP BY (payment.staff_id)
;

* 6c. List each film and the number of actors who are listed for that film. Use tables `film_actor` and `film`. Use inner join.
SELECT film.film_id, title, COUNT(actor_id) AS "Number of Actors          in the movie"
FROM film 
JOIN film_actor 
ON film.film_id = film_actor.film_id
GROUP BY (film.film_id)
;

* 6d. How many copies of the film `Hunchback Impossible` exist in the inventory system?
SELECT film.film_id, title, COUNT(inventory.film_id) AS "'Hunchback Impossible' copies in inventory"
FROM inventory 
JOIN film 
ON inventory.film_id = film.film_id
WHERE title = "Hunchback Impossible"
;
   
* 6e. Using the tables `payment` and `customer`and the `JOIN` command, list the total paid
by each customer. List the customers alphabetically by last name: */
SELECT last_name, first_name, SUM(amount) AS "Total paid by customer"
FROM customer 
JOIN payment 
ON customer.customer_id = payment.customer_id
GROUP BY payment.customer_id
ORDER BY last_name
; 

* 7a.Use subqueries to display the titles of movies starting with the letters `K` and `Q` 
whose language is English.*/
-- i) Using subquery
SELECT title
FROM film
WHERE (title LIKE "Q%" OR title LIKE "K%")
AND language_id = (SELECT language_id FROM language WHERE name = "English")
; 

-- i) Using join
SELECT title, name AS "Language"
FROM film
JOIN language
ON film.language_id = language.language_id
WHERE (title LIKE "Q%" OR title LIKE "K%")
;

* 7b. Use subqueries to display all actors who appear in the film `Alone Trip`.*/
SELECT concat(first_name," ",last_name) AS Actor_name
FROM actor WHERE actor_id IN (SELECT actor_id 
FROM film_actor
WHERE film_id = (SELECT film_id 
FROM film
WHERE title = "Alone Trip" ))
;

* 7c. You want to run an email marketing campaign in Canada, for which you will need the names and email addresses of all Canadian customers. Use joins to retrieve this information.
SELECT CONCAT(first_name," ",last_name) AS "Canadian Customer", email AS "Email Address"
FROM customer 
WHERE address_id IN (SELECT address_id FROM address
WHERE city_id IN (SELECT city_id FROM city
WHERE country_id = (SELECT country_id FROM country
WHERE country = "Canada")))
;

* 7d. Sales have been lagging among young families, and you wish to target all family movies for a promotion. Identify all movies categorized as family films.
-- i) Using subquery
SELECT title FROM film
WHERE film_id IN (SELECT film_id FROM film_category
WHERE category_id = (SELECT category_id FROM category 
WHERE name = "Family"))
;
-- ii) Using joins
SELECT title, name as "Genre"
FROM film
JOIN film_category
ON film.film_id = film_category.film_id
JOIN category
ON film_category.category_id = category.category_id
WHERE name = "FAMILY"
;

* 7e. Display the most frequently rented movies in descending order.
SELECT title, COUNT(rental_id) AS "Rental Frequency"
FROM rental 
JOIN inventory ON inventory.inventory_id = rental.inventory_id
JOIN film ON inventory.film_id = film.film_id
GROUP BY inventory.film_id
ORDER BY COUNT(rental_id) DESC
;

* 7f. Write a query to display how much business, in dollars, each store brought in.
SELECT store.store_id,SUM(amount) AS "Business in $$"
FROM payment 
JOIN customer
ON payment.customer_id = customer.customer_id
JOIN store
ON customer.store_id = store.store_id
GROUP BY customer.store_id
ORDER BY store.store_id,customer.customer_id
;

* 7g. Write a query to display for each store its store ID, city, and country.
SELECT store_id, city, country
FROM store
JOIN address
ON store.address_id = address.address_id
JOIN city
ON address.city_id = city.city_id
JOIN country
ON city.country_id = country.country_id
;
   
* 7h. List the top five genres in gross revenue in descending order. (**Hint**: you may need to use the following tables: category, film_category, inventory, payment, and rental.)
SELECT category.name AS "Genre", SUM(amount) AS "Gross Revenue"
FROM payment
JOIN rental
ON payment.rental_id = rental.rental_id
JOIN inventory
ON rental.inventory_id = inventory.inventory_id
JOIN film_category
ON inventory.film_id = film_category.film_id
JOIN category
ON film_category.category_id = category.category_id
GROUP BY category.name
ORDER BY SUM(amount) DESC
LIMIT 5
;
 
* 8a. In your new role as an executive, you would like to have an easy way of viewing the Top five genres by gross  revenue. Use the solution from the problem above to create a view. If you haven't solved # 7h, you can substitute another query to create a view.
CREATE VIEW Top_5_Genres_by_Gross 
AS SELECT category.name AS "Genre", SUM(amount) AS "Gross Revenue"
FROM payment
JOIN rental
ON payment.rental_id = rental.rental_id
JOIN inventory
ON rental.inventory_id = inventory.inventory_id
JOIN film_category
ON inventory.film_id = film_category.film_id
JOIN category
ON film_category.category_id = category.category_id
GROUP BY category.name
ORDER BY SUM(amount) DESC
LIMIT 5
;

* 8b. How would you display the view that you created in 8a?
SELECT * FROM Top_5_Genres_by_Gross
;

* 8c. You find that you no longer need the view `top_five_genres`. Write a query to delete it.
DROP VIEW IF EXISTS Top_5_Genres_by_Gross
;

### Appendix: List of Tables in the Sakila DB

* A schema is also available as `sakila_schema.svg`. Open it with a browser to view.

```sql
	'actor'
	'actor_info'
	'address'
	'category'
	'city'
	'country'
	'customer'
	'customer_list'
	'film'
	'film_actor'
	'film_category'
	'film_list'
	'film_text'
	'inventory'
	'language'
	'nicer_but_slower_film_list'
	'payment'
	'rental'
	'sales_by_film_category'
	'sales_by_store'
	'staff'
	'staff_list'
	'store'
```