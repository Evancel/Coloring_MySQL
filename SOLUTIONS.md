# Coloring_MySQL - Detailed Stages and Solutions

---

## Stage 1: Squares Never Painted
```sql
SELECT id, name 
FROM Square s 
WHERE NOT EXISTS(
    SELECT 1
    FROM Painting p 
    WHERE s.id = p.square_id
)
ORDER BY name DESC;
```

## Stage 2: Total Paint Used by Color
```sql
SELECT s.color, SUM(p.volume) AS total_paint_used
FROM Spray s
LEFT JOIN Painting p 
ON s.id = p.spray_id
GROUP BY s.color
ORDER BY total_paint_used;
```

## Stage 3: Remaining Spray Volumes
```sql
SELECT s.id, 255 - SUM(COALESCE(p.volume,0)) AS remaining_volume 
FROM Spray s
LEFT JOIN Painting p 
ON s.id = p.spray_id
GROUP BY s.id
ORDER BY s.id;
```

## Stage 4: White Squares Painted with ≥3 Sprays
```sql
SELECT square_id, COUNT(DISTINCT spray_id) AS num_sprays
FROM Painting
GROUP BY square_id
HAVING num_sprays >= 3
AND SUM(volume) >= 765
ORDER BY square_id;
```

## Stage 5: Red Sprays Used More Than Once with Blue Component
```sql
SELECT DISTINCT s.name 
FROM Spray s 
INNER JOIN Painting p 
ON s.id = p.spray_id
WHERE s.color = 'R'
AND EXISTS (
    SELECT 1 
    FROM Painting
    WHERE square_id = p.square_id
    AND spray_id IN (SELECT id FROM Spray WHERE color = 'B')
)
AND s.id IN (
    SELECT spray_id 
    FROM Painting 
    GROUP BY spray_id
    HAVING COUNT(*) >= 2
)
ORDER BY s.name;
```
Alternative:
```sql
SELECT DISTINCT Spray.name
FROM Spray
INNER JOIN Painting ON Spray.id = Painting.spray_id
WHERE color = "R"
GROUP BY square_id, name
HAVING COUNT(color = "R") > 1 
AND COUNT(color = "B") > 0;
```

## Stage 6: White Squares Painted Exclusively with Empty Sprays
```sql
WITH empty_spray_cans AS (
    SELECT spray_id 
    FROM Painting 
    GROUP BY spray_id 
    HAVING SUM(volume) >= 255
)

SELECT name
FROM Painting
JOIN Square ON square_id = id
GROUP BY square_id
HAVING COUNT(DISTINCT spray_id) >= 3
AND SUM(volume) >= 765
AND COUNT(spray_id) = SUM(
    CASE WHEN spray_id IN (SELECT spray_id FROM empty_spray_cans) THEN 1 ELSE 0 END
)
ORDER BY square_id;
```
