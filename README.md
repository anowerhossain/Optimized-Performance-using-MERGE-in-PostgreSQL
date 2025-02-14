# Optimized-Performance-using-MERGE-in-PostgreSQL
The MERGE statement in PostgreSQL offers several key advantages over traditional UPDATE, INSERT, and DELETE operations. It simplifies complex operations and enhances performance, making it a highly efficient solution for syncing or merging data between tables.

### Create the tables 
```sql
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    last_purchase DATE
);

CREATE TABLE crm_customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    last_purchase DATE
);
```

### Insert Data 
```sql
INSERT INTO customers (customer_id, name, email, last_purchase) VALUES
(1, 'Anower', 'anower@example.com', '2023-01-15'),
(2, 'Hossain', 'hossain@example.com', '2023-03-10'),
(3, 'Akash', 'akash@example.com', '2023-05-05');

INSERT INTO crm_customers (customer_id, name, email, last_purchase) VALUES
(1, 'Anower', 'anower@example.com', '2024-02-01'), -- Updated last purchase date
(3, 'Akash', 'akash@example.com', '2023-05-05'), -- No changes
(4, 'Imrul', 'imrul@example.com', '2024-01-20'); -- New customer

```

### Requirements 

1. If a matching customer_id exists, the `last_purchase` date in `customers` is updated if it’s earlier than in `crm_customers`.
2. New customers in `crm_customers` are inserted into the `customers` table if they don't already exist. This process ensures that the customers table stays up-to-date with the CRM data.


### Without `MERGE`

1. Update the `last_purchase`

```sql
   -- First, update existing records
UPDATE customers
SET last_purchase = crm.last_purchase
FROM crm_customers crm
WHERE customers.customer_id = crm.customer_id
AND customers.last_purchase < crm.last_purchase;
```

2. Insert the new customers record from `crm_customers` if they don't already exist in `customers` table.

```sql
-- Then, insert new customers
INSERT INTO customers (customer_id, name, email, last_purchase)
SELECT crm.customer_id, crm.name, crm.email, crm.last_purchase
FROM crm_customers crm
WHERE NOT EXISTS (
    SELECT 1 FROM customers c WHERE c.customer_id = crm.customer_id
);
```
