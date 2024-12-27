# Coloring_MySQL
SQL Project

## the general description
Start by learning basic SQL actions like SELECT, FROM, GROUP BY, and WHERE. Move on to understanding sum functions, logical and comparison operators, and then explore more advanced ideas like subqueries and join statements. This learning path is designed to equip you with the skills needed to efficiently fetch important information in SQL.

### 1st stage:
Identify the id and name of squares that have never been painted from the Square table in the coloring database. 
To determine if a square has not been painted, it must be absent in the Painting table. 
Ensure the results are sorted by square names in descending order. The column order is essential.

SELECT id, name 
FROM Square s 
WHERE NOT EXISTS(
    SELECT 1
    FROM Painting p 
    WHERE s.id = p.square_id
)
ORDER BY name DESC;

### 2nd stage:
Identify the color from the Spray table and SUM of total_paint_used 
from the Painting table in the coloring database. To achieve this, use a query 
that selects the color from the Spray table and calculates the sum of paint volume 
from the Painting table for each color. The column order is essential. 
Ensure that results are ordered by the total_paint_used. 
Use the GROUP BY and JOIN functions to solve the question.

SELECT s.color, sum(p.volume) AS total_paint_used
FROM Spray s
LEFT JOIN Painting p 
ON s.id = p.spray_id
GROUP BY s.color
ORDER BY total_paint_used;

### 3rd stage:
Identify the id of spray cans from the Spray table and their remaining volumes 
after all paintings from the Painting table in the coloring database. 
Use a query that selects the spray ID from the Spray table and calculates the remaining volume 
by subtracting the sum of paint volume from the Painting table. Ensure results are sorted by spray ids. 
The column order is essential. Utilize the GROUP BY and JOIN functions to accomplish the task. 
Use the COALESCE function to deal with the NULL values or in other words with the spray cans 
that were not been used for painting.

SELECT s.id, 255 - SUM(COALESCE(p.volume,0)) AS remaining_volume 
FROM Spray s
LEFT JOIN Painting p 
ON s.id = p.spray_id
GROUP BY s.id
ORDER BY s.id;

### 4th stage:
Identify the square_id of those painted in white and determine the num_sprays cans used to paint them
 from the Painting table in the coloring database. Use a query that selects the Square ID from the Painting table,
 calculates the COUNT of distinct spray ids, and filters the results for squares painted in white 
 where the SUM of paint volume is 765 and at least 3 different spray cans with different colors were used. 
 Ensure results are sorted by square ids. The column order is essential. 
 Use the GROUP BY and HAVING functions to achieve the task.
 
SELECT square_id, COUNT(DISTINCT spray_id) AS num_sprays
FROM Painting
GROUP BY square_id
HAVING num_sprays >= 3
AND SUM(volume) >= 765
ORDER BY square_id;

### 5th stage;
Identify the name of red ('R') spray cans from the Spray table that have been used more than once 
and have painted squares with a blue component in the coloring database. 
Utilize a query that joins the Spray and Painting tables, filtering for red spray cans ('R') 
and checking for the existence of blue spray cans ('B') painting the same squares. 
Ensure results are sorted by spray names. Use the DISTINCT keyword to fetch unique names 
and employ subqueries for precise filtering of spray cans.

SELECT DISTINCT s.name 
FROM Spray s 
INNER JOIN Painting p 
ON s.id = p.spray_id
WHERE s.color = 'R'
AND EXISTS (SELECT 1 FROM Painting
    WHERE square_id = p.square_id
    and spray_id IN (select id from Spray where color = 'B')
    )
AND s.id IN (SELECT spray_id FROM Painting 
            WHERE spray_id = p.spray_id
            GROUP BY spray_id
            HAVING COUNT(*) >= 2)
ORDER BY s.name

Alternative:
SELECT 
    DISTINCT Spray.name
FROM 
    Spray
INNER JOIN Painting 
    ON Spray.id = Painting.spray_id
WHERE 
    color = "R"
GROUP BY 
    square_id, name
HAVING 
    COUNT(color = "R") > 1 AND COUNT(color = "B") > 0
;

### 6rd stage:
Identify the name of white squares from the Square table exclusively painted with now-empty spray cans
 in the coloring database. Use a query that selects square names from the Square table, 
 filtering for squares where the SUM of the paint volume is 765 and painted by spray cans 
 which are empty after all paintings are done. Ensure results are sorted by square ids. 
 Employ subqueries for precise filtering of spray cans and white squares. 
 Use the GROUP BY, JOIN, and HAVING functions to achieve the task.
Hint: To accomplish the task you may use queries from Stages 4 and 5.

with empty_spray_cans AS (select spray_id from Painting group by spray_id having sum(volume) >= 255)

SELECT name
    FROM Painting
    JOIN Square
    ON square_id = id
GROUP BY square_id
HAVING COUNT(DISTINCT spray_id) >= 3
AND SUM(volume) >= 765
AND COUNT(spray_id) = SUM(CASE WHEN spray_id IN (select spray_id FROM empty_spray_cans) THEN 1 ELSE 0 END)
ORDER BY square_id

Link to Hyperskill: https://hyperskill.org/projects/429

