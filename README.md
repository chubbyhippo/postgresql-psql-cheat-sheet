# postgresql-psql-cheat-sheet

This cheat sheet provides commands and shortcuts for PostgreSQL operations using `psql`, along with backup and restore commands.

---

## Connecting to Database
Connect to PostgreSQL using different options:
```bash
psql -h <host> -U <username> -d <database_name>  # Connect with specified host, user, database
psql -U <username>                              # Connect using username, default to postgres
psql postgres                                   # Connect to the default postgres database
psql -p <port>                                  # Specify a custom port
```

---

## Basic psql Commands
- `\?` - Show help for psql commands.
- `\c <database>` - Connect to a specific database.
- `\l` - List all databases.
- `\dt` - List tables in the current database schema.
- `\dv` - List views in the current schema.
- `\ds` - List sequences in the current schema.
- `\d <table_name>` - Describe the schema of a table.
- `\d+ <table_name>` - Detailed schema of a table, including size.
- `\g` - Execute the previous query.
- `\x` - Toggle expanded output.
- `\timing` - Show the execution time of queries.
- `\q` - Quit the psql prompt.
- `\i <file.sql>` - Run queries from a `.sql` file.
- `\copy` - Import and export table data.

---

## Working with SQL
### CRUD Operations
#### Create Table
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT NOW()
);
```

#### Insert Data
```sql
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');
```

#### Select Data
```sql
SELECT * FROM users;                   -- Retrieve all rows from the table
SELECT * FROM users WHERE id = 1;      -- Retrieve a specific row
SELECT COUNT(*) FROM users WHERE name = 'John'; -- Count rows matching condition
```

#### Update Data
```sql
UPDATE users SET email = 'new.email@example.com' WHERE id = 1;
```

#### Delete Data
```sql
DELETE FROM users WHERE id = 5;
```

---

## Import/Export Table Data
### Export Data to CSV
```sql
COPY <table_name> TO '<path/to/file.csv>' DELIMITER ',' CSV HEADER;
```

### Import Data from CSV
```sql
COPY <table_name> FROM '<path/to/file.csv>' DELIMITER ',' CSV HEADER;
```
Alternative using `\copy` in `psql`:
```bash
\copy <table_name> FROM '<path/to/file.csv>' DELIMITER ',' CSV HEADER;
```

---

## Backup and Restore

### Backing Up Databases with `pg_dump`
#### Dump the Entire Database to SQL
```bash
pg_dump -U <username> -d <database_name> > backup.sql
```

#### Dump Only the Schema (No Data)
```bash
pg_dump -U <username> -d <database_name> --schema-only > schema_backup.sql
```

#### Dump Only Data (No Schema)
```bash
pg_dump -U <username> -d <database_name> --data-only > data_backup.sql
```

#### Dump a Specific Table
```bash
pg_dump -U <username> -d <database_name> -t <table_name> > table_backup.sql
```

#### Exclude Specific Tables
```bash
pg_dump -U <username> -d <database_name> --exclude-table=<table_name> > excluded_backup.sql
```

#### Custom Format Dump
```bash
pg_dump -F c -U <username> -d <database_name> > backup.custom
```

#### Compressed Dump
```bash
pg_dump -U <username> -d <database_name> | gzip > backup.sql.gz
```

#### Dump All Databases in the Server
```bash
pg_dumpall -U <username> > all_databases_backup.sql
```

---

### Restoring Databases with `psql` and `pg_restore`

#### Restore from a `.sql` File
```bash
psql -U <username> -d <database_name> -f backup.sql
```

#### Restore from a Custom Format Dump
```bash
pg_restore -U <username> -d <database_name> backup.custom
```

#### Restore from a Directory Backup
```bash
pg_restore -U <username> -d <database_name> -j <threads> ./backup_dir
```

#### Restore Only Schema
```bash
psql -U <username> -d <database_name> -f schema_backup.sql
```

#### Restore Only Data
```bash
psql -U <username> -d <database_name> -f data_backup.sql
```

#### Decompress and Restore
```bash
gunzip -c backup.sql.gz | psql -U <username> -d <database_name>
```

---

## Transactions
- **Start a Transaction**:
  ```sql
  BEGIN;
  ```
- **Commit Changes**:
  ```sql
  COMMIT;
  ```
- **Rollback Changes**:
  ```sql
  ROLLBACK;
  ```

---

## User and Role Management
- Create a User:
  ```sql
  CREATE USER <username> WITH PASSWORD '<password>';
  ```
- Grant Superuser Privileges:
  ```sql
  ALTER USER <username> WITH SUPERUSER;
  ```
- Grant All Privileges:
  ```sql
  GRANT ALL PRIVILEGES ON DATABASE <database_name> TO <username>;
  ```
- Revoke All Privileges:
  ```sql
  REVOKE ALL PRIVILEGES ON DATABASE <database_name> FROM <username>;
  ```
- List All Users:
  ```bash
  \du
  ```

---

## Debugging and Logs
- Enable Query Timing:
  ```bash
  \timing
  ```
- Show Query Execution Plan:
  ```sql
  EXPLAIN ANALYZE <query>;
  ```
- View Active Connections:
  ```sql
  SELECT * FROM pg_stat_activity;
  ```
- Terminate a Connection:
  ```sql
  SELECT pg_terminate_backend(<pid>);
  ```

---

## Practical Examples
### Create Indexes
```sql
CREATE INDEX idx_users_email ON users(email);
```

### Joins Example
```sql
SELECT u.name, o.order_date
FROM users u
JOIN orders o ON u.id = o.user_id;
```

### Aggregate Functions
```sql
SELECT COUNT(*), AVG(price) FROM products WHERE category = 'Electronics';
```

### List All Schemas
```bash
\dn
```

### List All Extensions
```bash
\dx
```

---

## Tips and Best Practices
- **Test Restores**: Always test your backups in a staging environment to ensure correctness.
- **Automate Backups**: Use cron jobs or task schedulers to automate backups.
- **Query Optimization**: Use `EXPLAIN ANALYZE` to monitor query performance.
- **Permissions Management**: Regularly check and update user permissions.

---
