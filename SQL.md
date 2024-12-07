# SQL Cheat Sheet

## 1. Basic Structure of SQL Queries

```sql
SELECT column1, column2
FROM table_name
WHERE condition
GROUP BY column1
HAVING condition
ORDER BY column1 ASC/DESC
LIMIT n;
OFFSET x; -- ne renvoie pas les x premiers résultats

CREATE TABLE nom_de_table (
    id INT PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    age INT
);


INSERT INTO nom_de_table (first_name, last_name, age) VALUES ('Alice', 'Johnson', 30);

UPDATE nom_de_table
SET age = 31
WHERE first_name = 'Alice' AND last_name = 'Johnson';

DELETE FROM nom_de_table
WHERE first_name = 'Alice' AND last_name = 'Johnson';

ALTER TABLE nom_de_table ADD column_name datatype;

DROP TABLE nom_de_table;
```

--

## 2. Common SQL Functions

### Aggregate Functions
- `COUNT()` - Counts rows.
- `SUM()` - Calculates total.
- `AVG()` - Calculates average.
- `MIN()`/`MAX()` - Finds minimum/maximum value.

Example:
```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

---

## 3. Window Functions
Window functions operate on a set of rows related to the current row.

### Syntax
```sql
function_name() OVER (
  [PARTITION BY column]
  [ORDER BY column ASC/DESC]
)
```

### 3.1 ROW_NUMBER()
Assigns a unique sequential number to each row.
```sql
SELECT sale_id, salesperson, sale_amount,
  ROW_NUMBER() OVER(PARTITION BY salesperson ORDER BY sale_date) AS row_num
FROM sales;
```
**Result:**
| sale_id | salesperson | sale_amount | row_num |
|---------|-------------|-------------|---------|
| 1       | Alice       | 300         | 1       |
| 3       | Alice       | 700         | 2       |
| 5       | Alice       | 500         | 3       |
| 2       | Bob         | 150         | 1       |
| 4       | Bob         | 200         | 2       |

### 3.2 RANK()
Assigns ranks, leaving gaps for ties.
```sql
SELECT sale_id, salesperson, sale_amount,
  RANK() OVER(ORDER BY sale_amount DESC) AS sale_rank
FROM sales;
```
**Result:**
| sale_id | salesperson | sale_amount | sale_rank |
|---------|-------------|-------------|-----------|
| 3       | Alice       | 700         | 1         |
| 5       | Alice       | 500         | 2         |
| 1       | Alice       | 300         | 3         |
| 4       | Bob         | 200         | 4         |
| 2       | Bob         | 150         | 5         |

### 3.3 DENSE_RANK()
Assigns ranks without gaps for ties.
```sql
SELECT sale_id, salesperson, sale_amount,
  DENSE_RANK() OVER(ORDER BY sale_amount DESC) AS sale_dense_rank
FROM sales;
```
**Result:**
| sale_id | salesperson | sale_amount | sale_dense_rank |
|---------|-------------|-------------|-----------------|
| 3       | Alice       | 700         | 1               |
| 5       | Alice       | 500         | 2               |
| 1       | Alice       | 300         | 3               |
| 4       | Bob         | 200         | 4               |
| 2       | Bob         | 150         | 5               |

### 3.4 NTILE(n)
Divides rows into `n` groups.
```sql
SELECT sale_id, salesperson, sale_amount,
  NTILE(2) OVER(ORDER BY sale_amount DESC) AS sale_group
FROM sales;
```
**Result:**
| sale_id | salesperson | sale_amount | sale_group |
|---------|-------------|-------------|------------|
| 3       | Alice       | 700         | 1          |
| 5       | Alice       | 500         | 1          |
| 1       | Alice       | 300         | 2          |
| 4       | Bob         | 200         | 2          |
| 2       | Bob         | 150         | 2          |

### 3.5 LAG()/LEAD()
Access the previous (`LAG`) or next (`LEAD`) row value.
```sql
SELECT sale_id, salesperson, sale_amount,
  LAG(sale_amount, 1, 0) OVER(PARTITION BY salesperson ORDER BY sale_date) AS previous_sale
FROM sales;
```
**Result:**
| sale_id | salesperson | sale_amount | previous_sale |
|---------|-------------|-------------|---------------|
| 1       | Alice       | 300         | 0             |
| 3       | Alice       | 700         | 300           |
| 5       | Alice       | 500         | 700           |
| 2       | Bob         | 150         | 0             |
| 4       | Bob         | 200         | 150           |

### 3.6 SUM() as a Window Function
Cumulative totals.
```sql
SELECT sale_id, salesperson, sale_amount,
  SUM(sale_amount) OVER(PARTITION BY salesperson ORDER BY sale_date) AS cumulative_sales
FROM sales;
```
**Result:**
| sale_id | salesperson | sale_amount | cumulative_sales |
|---------|-------------|-------------|------------------|
| 1       | Alice       | 300         | 300              |
| 3       | Alice       | 700         | 1000             |
| 5       | Alice       | 500         | 1500             |
| 2       | Bob         | 150         | 150              |
| 4       | Bob         | 200         | 350              |

---


```sql
SELECT
  RANK() OVER(ORDER BY points DESC) AS rank,
  DENSE_RANK() OVER(ORDER BY points DESC) AS dense_rank,
  ROW_NUMBER() OVER(ORDER BY points DESC) AS row_number,
  first_name,
  last_name,
  points
FROM exam_result;
```

**Result:**
| rank | dense_rank | row_number | first_name | last_name   | points |
|------|------------|------------|------------|-------------|--------|
| 1    | 1          | 1          | Marlene    | Duncan      | 90     |
| 1    | 1          | 2          | Rayhan     | Williamson  | 90     |
| 1    | 1          | 3          | Marius     | Powell      | 90     |
| 1    | 1          | 4          | Eryk       | Myers       | 90     |
| 5    | 2          | 5          | Evie-May   | Boyer       | 80     |
| 6    | 3          | 6          | Emyr       | Downes      | 70     |
| 6    | 3          | 7          | Dina       | Morin       | 70     |
| 8    | 4          | 8          | Nora       | Parkinson   | 60     |
| 9    | 5          | 9          | Joanne     | Goddard     | 50     |
| 10   | 6          | 10         | Trystan    | Oconnor     | 40     |


### Exemple moyenne glissante
|sale_date	|sales_amount|	moving_average_3_days |
|-----------|------------|------------------------|
|2024-12-01	|100	     |  100.00                |
|2024-12-02	|150	     |	125.00                |
|2024-12-03	|200	     |	150.00                |
|2024-12-04	|250	     |	200.00                |
|2024-12-05	|300	     |	250.00                |
|2024-12-06	|350	     |	300.00                |

## 4. Joins

### Inner Join
```sql
SELECT employees.name, departments.name
FROM employees
INNER JOIN departments ON employees.department_id = departments.id;
```

### Left Join
```sql
SELECT employees.name, departments.name
FROM employees
LEFT JOIN departments ON employees.department_id = departments.id;
```

---

## 5. Filtering and Sorting

### WHERE Clause
```sql
SELECT * FROM sales WHERE sale_amount > 500;
```

### ORDER BY Clause
```sql
SELECT * FROM sales ORDER BY sale_amount DESC;
```

---

## 6. Grouping and Aggregation

### GROUP BY
```sql
SELECT salesperson, SUM(sale_amount) AS total_sales
FROM sales
GROUP BY salesperson;
```

### HAVING Clause
```sql
SELECT salesperson, SUM(sale_amount) AS total_sales
FROM sales
GROUP BY salesperson
HAVING total_sales > 500;
```

---

## 7. Subqueries

### Simple Subquery
```sql
SELECT * FROM sales WHERE sale_amount > (SELECT AVG(sale_amount) FROM sales);
```

### Correlated Subquery
```sql
SELECT sale_id, sale_amount
FROM sales s1
WHERE sale_amount > (SELECT AVG(sale_amount) FROM sales s2 WHERE s1.salesperson = s2.salesperson);
```

---

## 8. Common Table Expressions (CTEs)

### Syntax
```sql
WITH cte_name AS (
  SELECT column1, column2 FROM table_name WHERE condition
)
SELECT * FROM cte_name;
```

Example:
```sql
WITH sales_summary AS (
  SELECT salesperson, SUM(sale_amount) AS total_sales
  FROM sales
  GROUP BY salesperson
)
SELECT * FROM sales_summary WHERE total_sales > 500;
``` 

## 9. CASE WHEN
```sql
SELECT
    first_name,
    last_name,
    points,
    CASE
        WHEN points >= 90 THEN 'Excellent'
        WHEN points >= 70 THEN 'Good'
        WHEN points >= 50 THEN 'Average'
        ELSE 'Poor'
    END AS performance
FROM exam_result;
``` 



## 10. COALESCE
La fonction COALESCE permet de renvoyer le premier argument non NULL dans une liste de valeurs. Cela est particulièrement utile pour gérer les valeurs manquantes dans les colonnes.


### Exemple 
Voici un exemple de la table `employees` :

| first_name | last_name | phone_number |
|------------|-----------|--------------|
| Alice      | Johnson   | 1234567890   |
| Bob        | Smith     | NULL         |
| Charlie    | Brown     | 9876543210   |

```sql
SELECT
    first_name,
    last_name,
    COALESCE(phone_number, 'No phone number available') AS contact_number
FROM employees;
```

Explication :
- Si la colonne phone_number contient une valeur NULL, COALESCE renverra 'No phone number available'.
- Si phone_number n'est pas NULL, il renverra la valeur du phone_number.

| first_name | last_name | phone_number             |
|------------|-----------|--------------------------|
| Alice      | Johnson   | 1234567890               |
| Bob        | Smith     | No phone number available|
| Charlie    | Brown     | 9876543210               |

# T-SQL

```
-- Déclaration de variables
DECLARE @age INT;
SET @age = 25;

-- Condition
IF @age > 30
    PRINT 'Age is greater than 30';
ELSE
    PRINT 'Age is 30 or less';

```

La commande `MERGE` permet de combiner des opérations `INSERT`, `UPDATE` et `DELETE` en une seule instruction, ce qui est très utile pour la gestion des données dans des scénarios complexes de synchronisation.

```
MERGE INTO employees AS target -- table où on va mettre de nouvelles données
USING new_employees AS source
ON target.id = source.id
WHEN MATCHED THEN 
    UPDATE SET target.age = source.age
WHEN NOT MATCHED THEN
    INSERT (first_name, last_name, age)
    VALUES (source.first_name, source.last_name, source.age);
```


 ## Créer une procédure stockée
Les procédures stockées sont des ensembles de requêtes SQL enregistrées qui peuvent être exécutées à la demande.

```sql
CREATE PROCEDURE GetEmployeeDetails
    @employee_id INT
AS
BEGIN
    SELECT first_name, last_name, age
    FROM employees
    WHERE id = @employee_id;
END;

-- Exécuter une procédure stockée
EXEC GetEmployeeDetails @employee_id = 1;
```

## Gestion des erreurs
 TRY...CATCH Permet de capturer et gérer les erreurs.
```sql
BEGIN TRY
    -- Code susceptible de provoquer une erreur
    UPDATE employees SET age = 'invalid_value';
END TRY
BEGIN CATCH
    -- Gestion de l'erreur
    PRINT 'Erreur: ' + ERROR_MESSAGE();
END CATCH;
```


Les index permettent de rendre les recherches de données beaucoup plus rapides. T-SQL permet de créer des index pour améliorer la performance des requêtes.
```sql
CREATE INDEX idx_employee_name ON employees(first_name, last_name);
DROP INDEX idx_employee_name ON employees;
```

```sql
Exemple de transaction SQL
-- Début de la transaction
BEGIN TRANSACTION;

BEGIN TRY
    -- Mise à jour du salaire de l'employé
    UPDATE employees
    SET salary = salary + 5000
    WHERE employee_id = 1;
    
    -- Insertion dans la table des changements de salaire
    INSERT INTO salary_changes (employee_id, old_salary, new_salary, change_date)
    SELECT employee_id, salary - 5000, salary, GETDATE()
    FROM employees
    WHERE employee_id = 1;

    -- Si toutes les opérations réussissent, on valide la transaction
    COMMIT TRANSACTION;
    PRINT 'Transaction réussie et validée.';
END TRY
BEGIN CATCH
    -- En cas d'erreur, annulation de la transaction
    ROLLBACK TRANSACTION;
    PRINT 'Erreur survenue, la transaction a été annulée.';
    PRINT ERROR_MESSAGE();  -- Affiche le message d'erreur
END CATCH;
```





