# DBT Cheat Sheet

## Introduction
**DBT (Data Build Tool)** est un outil de transformation des données dans un pipeline ELT. Il simplifie l'organisation, la transformation, le test, et la documentation des données dans des entrepôts modernes.

---

## Installation

1. **Prérequis** :
   - Python 3.8+
   - Entrepôt compatible (Postgres, Snowflake, BigQuery, etc.)

2. **Installation via pip** :
   ```bash
   pip install dbt-core
   pip install dbt-{adapter}  # Exemple : dbt-postgres, dbt-snowflake
   ```

3. **Vérification** :
   ```bash
   dbt --version
   ```

---

## Initialisation d'un Projet

1. **Créer un projet** :
   ```bash
   dbt init nom_du_projet
   ```

2. **Structure du projet** :
   ```
   nom_du_projet/
   ├── dbt_project.yml         # Configuration principale
   ├── models/                 # Modèles SQL
   ├── seeds/                  # Données statiques (CSV)
   ├── macros/                 # Fonctions SQL personnalisées
   ├── tests/                  # Tests de données
   ├── target/                 # Résultats d'exécution
   └── logs/                   # Journaux d'exécution
   ```

3. **Configurer le profil (`profiles.yml`)** :
   Exemple pour Postgres :
   ```yaml
   my_profile:
     target: dev
     outputs:
       dev:
         type: postgres
         host: localhost
         user: my_user
         password: my_password
         port: 5432
         dbname: my_database
         schema: public
   ```

---

## Concepts Clés

### 1. **Modèles (`models/`):**
Les modèles sont des fichiers SQL contenant des transformations. DBT gère les dépendances avec `ref()`.

Exemple :
```sql
SELECT 
    customer_id,
    COUNT(order_id) AS total_orders
FROM {{ ref('orders') }} #{{ ref('orders') }} fait référence à un autre modèle DBT (par ex. un fichier models/orders.sql).
GROUP BY customer_id
```
Commande :
```bash
dbt run
```

Pour exécuter un modèle spécifique :
```bash
dbt run --select nom_du_modele
```

### 2. **Seeds (`seeds/`):**
Ce sont des données statiques en fichier CSV dans `seeds/` et peuvent être chargés dans l'entrepôt.

Commande :
```bash
dbt seed
```

### 3. **Tests :**
#### Tests Natifs :
- **`not_null`** : Vérifie qu'une colonne n'a pas de valeurs nulles.
- **`unique`** : Vérifie l'unicité des valeurs d'une colonne.
- **`accepted_values`** : Vérifie que les valeurs appartiennent à une liste donnée.
- **`relationships`** : Vérifie les relations (clé étrangère).

Exemple dans `schema.yml` :
```yaml
version: 2
models:
  - name: sales_summary #modèle
    columns:
      - name: customer_id #nom de la colonne à effectuer des tests
        tests:
          - not_null
          - unique
```

```yaml
sources:
  - name: raw_data
    tables:
      - name: orders
        freshness:
          warn_after: {count: 1, period: day}
          error_after: {count: 2, period: day}
```

Commande :
```bash
dbt test
```

Exécuter les tests pour un modèle spécifique :
```bash
dbt test --models sales_summary
```

#### Tests Personnalisés :
1. Exemple 1
Définir un test dans `macros/` :
```sql
{% macro test_positive_values(model, column_name) %}
SELECT *
FROM {{ model }}
WHERE {{ column_name }} < 0
{% endmacro %}
```
Utilisation dans `schema.yml` :
```yaml
- name: total_revenue
  tests:
    - test_positive_values
```

2. Exemple 2
```yaml
{% set columns_to_test = ['customer_id', 'order_id', 'product_id'] %}
version: 2
models:
  - name: sales_summary
    columns:
    {% for column in columns_to_test %}
      - name: {{ column }}
        tests:
          - not_null
    {% endfor %}
```

3. Exemple 3
```sql
{% macro test_value_range(model, column_name, min_value, max_value) %}
SELECT *
FROM {{ model }}
WHERE {{ column_name }} < {{ min_value }} OR {{ column_name }} > {{ max_value }}
{% endmacro %}
```

```yaml
- name: order_amount
  tests:
    - test_value_range:
        min_value: 0
        max_value: 10000
```


### 4. **Macros (`macros/`):**
Les macros sont des fonctions SQL réutilisables écrites en Jinja.

Exemple :
```sql
{% macro calculate_growth(current, previous) %}
  ({{ current }} - {{ previous }}) / {{ previous }}
{% endmacro %}
```
Utilisation :
```sql
SELECT 
    {{ calculate_growth('current_month_revenue', 'previous_month_revenue') }} AS growth
```

### 5. **Documentation :**
Générer une documentation interactive basée sur les modèles :
```bash
dbt docs generate
dbt docs serve
```

---

## Tests Disponibles

### Tests Natifs :
- **`not_null`** : Aucune valeur nulle.
- **`unique`** : Valeurs uniques.
- **`accepted_values`** : Valeurs dans une liste donnée.
- **`relationships`** : Clé étrangère valide.
- **`freshness`** : Actualité des données dans une source.

### Tests Personnalisés :
Créer des tests spécifiques avec des macros SQL. Exemples :
- Vérifier les plages de valeurs.
- Comparer deux colonnes.
- Tester un pourcentage maximal de valeurs nulles.

---

## Commandes Essentielles

1. **Initialiser un projet** :
   ```bash
   dbt init nom_du_projet
   ```

2. **Exécuter les modèles** :
   ```bash
   dbt run
   ```

   Exécuter un modèle spécifique :
   ```bash
   dbt run --select nom_du_modele
   ```

   Exécuter les modèles dans un dossier spécifique :
   ```bash
   dbt run --select dossier /*
   ```

   Modèles en cascade (descendants ou dépendants d’un modèle) :
   ```bash
   dbt run --select dossier /*
   ```

3. **Tester les données** :
   ```bash
   dbt test
   ```

4. **Générer la documentation** :
   ```bash
   dbt docs generate
   dbt docs serve
   ```

5. **Tester la fraîcheur des sources** :
   ```bash
   dbt source freshness
   ```

6. **Nettoyer les fichiers générés** :
   ```bash
   dbt clean
   ```

---

## Bonnes Pratiques

1. **Documenter les modèles et colonnes** dans `schema.yml`.
2. **Appliquer des tests aux données critiques** (ex. clés primaires).
3. **Automatiser les tests** avec des macros et des scripts CI/CD.
4. **Organiser les modèles** en dossiers : `staging`, `intermediate`, `final`.
5. **Réutiliser les macros** pour éviter la duplication du code.

---

## Liens Utiles
- Documentation officielle : [DBT Documentation](https://docs.getdbt.com/)
