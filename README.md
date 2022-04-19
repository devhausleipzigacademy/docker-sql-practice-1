# Docker & SQL Practice #1

## Prerequisites
- Install the VSCode extension 'SQLTools'
- Have the Docker driver/daemon running

## Vocabulary
- image
- container
- spin up
- table
- drop table
- row/entry
- primary key, foreign key
- inner/outer joins

## Docker
- `docker-compose up`
- `docker-compose up -d`
- `docker-compose stop`

## SQL
- `CREATE TABLE`
- `INT`
- `CHAR`
- `PRIMARY KEY`
- `DROP TABLE`
- `SERIAL`
- `VARCHAR`
- `NOT NULL`
- `UUID`
- `ALTER TABLE`
- `ADD`
- `SELECT`
- `AS`
- `ORDER BY`
- `ASC`
- `DESC`
- `LIMIT`
- `WHERE`
- `DISTINCT`
- `LIKE`
- `AND`
- `OR`
- `BETWEEN`
- `IS NULL`
- `DELETE`
- `UPDATE`
- `SET`
- `JOIN`
- `LEFT JOIN`
- `HAVING`

## Why UUIDs instead of auto-incrementing integers for primary keys?

### Positives
- Security: Externals cannot guess IDs and cannot infer number of records from new inserts.
- Flexibility: You can better interoperate with distributed system-based tools/services that work with flat collections of objects that must have unique IDs.

### Negatives
- Space: UUIDs take up more space than integers. However, modern systems have a lot of RAM and there are sophisticated caching strategies available.
- Performance?: In some SQL implementations, UUIDs can reduce the performance of indices (not applicable to PostgreSQL).

Default to using UUIDs unless you encounter performance issues and understand how to manage the security implications of using auto-incrementing integers.

## Resources
- [VSCode SQLTools Docs](https://vscode-sqltools.mteixeira.dev/)
- [Docker Compose Docs](https://docs.docker.com/compose/)
- [Bitnami PostgreSQL Images](https://hub.docker.com/r/bitnami/postgresql/)
- [PostgreSQL Docs](https://www.postgresql.org/docs/14/index.html)
- [PostgreSQL Auto-incrementing Integer Key](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-serial/)
- [SQL UUIDS & Background Info](https://arctype.com/blog/postgres-uuid/)
- [PostgreSQL Alter Table](https://www.postgresql.org/docs/14/sql-altertable.html)
- [PostgreSQL Operators & Functions](https://www.postgresql.org/docs/14/functions.html)
- [PostgreSQL Having](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-having/)

## Reference

```sql
CREATE TABLE test (
    identifier INT PRIMARY KEY,
    label CHAR(255)
);

DROP TABLE test;
CREATE TABLE test (
    identifier SERIAL PRIMARY KEY,
    label VARCHAR(255) NOT NULL
);

DROP TABLE test;
CREATE TABLE restaurant (
    identifier UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    cuisine VARCHAR(255) NOT NULL
);

ALTER TABLE restaurant
ADD founded VARCHAR(255);

INSERT INTO restaurant (name, founded, cuisine)
VALUES
    ('Lord of the Fries', '1983-10-14', 'American'),
    ('Planet of the Grapes', '1985-04-03', 'American'),
    ('Lord of the Wings', '1979-05-24', 'Italian'),
    ('I Know Carrote', null, 'French')

SELECT name, cuisine
FROM restaurant;

SELECT name AS restaurantName, cuisine AS foodStyle
FROM restaurant;

SELECT *
FROM restaurant;

SELECT *
FROM restaurant
ORDER BY founded;

SELECT *
FROM restaurant
ORDER BY founded ASC;

SELECT *
FROM restaurant
ORDER BY founded DESC;

SELECT *
FROM restaurant
ORDER BY founded DESC
LIMIT 2;

SELECT *
FROM restaurant
WHERE cuisine = 'American';

SELECT cuisine
FROM restaurant;

SELECT DISTINCT cuisine
FROM restaurant;

SELECT *
FROM restaurant
WHERE founded < '1982';

SELECT *
FROM restaurant
WHERE name LIKE '%Lord%';

SELECT *
FROM restaurant
WHERE name LIKE '%Lord%' AND cuisine = 'American';

SELECT *
FROM restaurant
WHERE name LIKE '%Lord%' OR founded > '1984'

SELECT *
FROM restaurant
WHERE founded BETWEEN '1974' AND '1981';

SELECT *
FROM restaurant
WHERE founded IS NULL;

DELETE
FROM restaurant
WHERE name = 'I Know Carrote';

SELECT *
FROM restaurant
ORDER BY founded;

UPDATE restaurant
SET founded = '1983-11-14'
WHERE name = 'Lord of the Fries';

SELECT *
FROM restaurant
ORDER BY founded;

CREATE TABLE menu_item (
    identifier UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    price NUMERIC,
    restaurantID UUID REFERENCES restaurant(identifier)
);

INSERT INTO restaurant (name, founded, cuisine)
    VALUES ('World Peas', '1971-06-13', 'French')
    RETURNING identifier;

DELETE FROM restaurant
WHERE name = 'World Peas';

WITH returned AS (
    INSERT INTO restaurant (name, founded, cuisine)
    VALUES ('World Peas', '1971-06-13', 'French')
    RETURNING identifier
)
INSERT INTO menu_item (name, price, restaurantID)
VALUES ('Chicken Enfilade', 8.99, (SELECT identifier FROM returned));

SELECT * FROM menu_item;

INSERT INTO menu_item (name, price, restaurantID)
VALUES ('Martini Escargo', 5.99, (SELECT identifier FROM restaurant WHERE name = 'World Peas') );

INSERT INTO menu_item (name, price, restaurantID)
VALUES ('Green Eggs & Ham', 10.99, (SELECT identifier FROM restaurant WHERE name = 'World Peas') );

INSERT INTO menu_item (name, price, restaurantID)
VALUES ('Fry me a River', 4.99, (SELECT identifier FROM restaurant WHERE name = 'Lord of the Fries') );

INSERT INTO menu_item (name, price, restaurantID)
VALUES ('Conquerwing Nation', 7.99, (SELECT identifier FROM restaurant WHERE name = 'Lord of the Wings') );

SELECT *
FROM menu_item;

SELECT *
FROM restaurant
JOIN menu_item ON restaurant.identifier = menu_item.restaurantID;

SELECT * FROM restaurant
INNER JOIN menu_item ON restaurant.identifier = menu_item.restaurantID;

SELECT * FROM restaurant
LEFT JOIN menu_item ON restaurant.identifier = menu_item.restaurantID;

SELECT (identifier, name, price) from menu_item
WHERE restaurantID = (SELECT identifier FROM restaurant WHERE name = 'World Peas')

SELECT AVG(price) from menu_item;

SELECT restaurantID, COUNT(restaurantID) AS numberOfMenuItems
FROM menu_item
GROUP BY restaurantID;

SELECT restaurant.name as restaurantName, COUNT(menu_item.identifier) AS numberOfMenuItems
FROM restaurant
LEFT JOIN menu_item ON restaurant.identifier = menu_item.restaurantID
GROUP BY restaurant.identifier;

SELECT restaurant.name as restaurantName, COUNT(menu_item.identifier) AS numberOfMenuItems
FROM restaurant
LEFT JOIN menu_item ON restaurant.identifier = menu_item.restaurantID
GROUP BY restaurant.identifier
HAVING COUNT(menu_item.identifier) > 2;

SELECT restaurant.name as restaurantName, COUNT(menu_item.identifier) AS numberOfMenuItems, AVG(menu_item.price) as averagePrice
FROM restaurant
LEFT JOIN menu_item ON restaurant.identifier = menu_item.restaurantID
GROUP BY restaurant.identifier;

SELECT restaurant.name as restaurantName, COUNT(menu_item.identifier) AS numberOfMenuItems, AVG(menu_item.price) as averagePrice
FROM restaurant
LEFT JOIN menu_item ON restaurant.identifier = menu_item.restaurantID
GROUP BY restaurant.identifier
HAVING AVG(menu_item.price) < 8;
```