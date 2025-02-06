# PostgreSQL (psql) Cheat Sheet

## Connecting to Database
Connect to PostgreSQL using different options:
```bash
psql -h <host> -U <username> -d <database_name>  # Connect with specified host, user, database
psql -U <username>                              # Connect using username, default to postgres
psql postgres                                   # Connect to the default postgres database
psql -p <port>                                  # Specify a custom port
```

---

## Login Without Typing a Password
To log in to PostgreSQL or run commands without typing the password every time, use one of the following methods:

### Option 1: Use the `.pgpass` File (Recommended)
1. Locate your **home directory**:
   - **Linux/Mac**:
     ```bash
     cd ~
     ```
   - **Windows**:
     Navigate to `C:\Users\<your-username>`.

2. Create a `.pgpass` file (or `pgpass.conf` on Windows):
   - **Linux/Mac**:
     ```bash
     touch ~/.pgpass
     chmod 600 ~/.pgpass  # Restrict file permissions
     ```
   - **Windows**:
     Create a file named `pgpass.conf` in the folder `C:\Users\<your-username>`.

3. Add your PostgreSQL credentials in the following format (one entry per line):
   ```
   hostname:port:database:username:password
   ```
   Example:
   ```
   localhost:5432:mydatabase:myuser:mypassword
   ```

4. PostgreSQL tools like `psql` and `pg_dump` will now read this file for authentication and save you from typing the password.

---

### Option 2: Export Password as an Environment Variable (Temporary)
You can temporarily export the PostgreSQL password as an environment variable, which avoids manually entering it for the current session only.

- **Linux/Mac**:
  ```bash
  export PGPASSWORD="your_password"
  ```
- **Windows (Command Prompt)**:
  ```cmd
  set PGPASSWORD=your_password
  ```
- **Windows (PowerShell)**:
  ```powershell
  $env:PGPASSWORD="your_password"
  ```

When using this method, you can run commands like the following without being prompted for a password:
```bash
psql -U <username> -h <hostname> -d <database>
pg_dump -U <username> -h <hostname> -d <database> > backup.sql
```

> **Note:** This method works only for the current session and isn't persistent. Use the `.pgpass` file for a permanent solution.

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
- `\z [object_name]` - Show access privileges for a table/column/schema.
- `\g` - Execute the previous query.
- `\x` - Toggle expanded output.
- `\timing` - Show the execution time of queries.
- `\q` - Quit the psql prompt.
- `\i <file.sql>` - Run queries from a `.sql` file.
- `\copy` - Import and export table data.

---

## Running `.sql` Files
To execute `.sql` files that contain SQL commands, use the following:

### Using `psql` Command-Line
```bash
psql -U <username> -d <database_name> -f <path_to_sql_file.sql>
```

### Inside `psql`
If you're already inside the `psql` interactive terminal:
```sql
\i <path_to_sql_file.sql>
```

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

#### Custom Format Dump
```bash
pg_dump -F c -U <username> -d <database_name> > backup.custom
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
- List All Users:
  ```sql
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

---

## Tips and Best Practices
- **Test Restores**: Always test your backups in a staging environment to ensure correctness.
- **Automate Backups**: Use cron jobs or task schedulers to automate backups.
- **Query Optimization**: Use `EXPLAIN ANALYZE` to monitor query performance.
- **Security Best Practices**: Use `.pgpass` for managing passwords securely.

---
