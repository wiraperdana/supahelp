# Supahelp - Supabase Helper

A command-line utility for importing, exporting, and querying Supabase databases.

## Overview

`supahelp` is a command-line utility that provides a simple interface for:

1. **Importing SQL files** into a Supabase database
2. **Exporting (dumping)** a Supabase database to SQL files
3. **Executing SQL queries** directly from the command line or from SQL files

This tool is particularly useful for database migrations, backups, data management tasks, and ad-hoc queries.

## Prerequisites

- Node.js (v12 or higher)
- npm (Node Package Manager)
- PostgreSQL client libraries (for the `pg` module)

## Installation

### Installation from npm

The easiest way to install is directly from npm:

```bash
npm install -g supahelp
```

After installation, you can run the `supahelp` command from anywhere.

### Local Installation

If you prefer to install from source:

1. Make sure you have Node.js and npm installed
2. Clone this repository or download the source code
3. Install the package locally:

```bash
# Install dependencies
npm install

# Install globally from the local directory
npm install -g .
```

This will make the `supahelp` command available globally on your system.

### Using Without Installation

If you prefer not to install the package globally, you can run it directly:

```bash
# Make the script executable
chmod +x supahelp

# Run the script directly
./supahelp [command] [options]

# Or using node
node supahelp [command] [options]
```

## Configuration

The script uses a connection string to connect to your Supabase database. The connection string is read from the `SUPABASE_CONNECTION_STRING` environment variable.

### Setting the Connection String

You can set the environment variable before running the script:

```bash
# On Linux/Mac
export SUPABASE_CONNECTION_STRING='postgresql://postgres:password@your-supabase-host:5432/postgres'

# On Windows (Command Prompt)
set SUPABASE_CONNECTION_STRING=postgresql://postgres:password@your-supabase-host:5432/postgres

# On Windows (PowerShell)
$env:SUPABASE_CONNECTION_STRING='postgresql://postgres:password@your-supabase-host:5432/postgres'
```

If the environment variable is not set, the script will use a default local connection string:

```
postgresql://postgres:password@localhost:5432/postgres
```

### Connection String Format

The connection string follows the standard PostgreSQL format:

```
postgresql://[username]:[password]@[host]:[port]/[database]
```

For Supabase, you can find your connection details in the Supabase dashboard under Project Settings > Database.

## Usage

### Basic Commands

```bash
# Import SQL file to Supabase
supahelp import <file.sql>

# Export Supabase database to SQL file
supahelp dump [output_file.sql]

# Export only the schema without data
supahelp dump --schema-only [output_file.sql]

# Execute SQL query directly from command line
supahelp query "SELECT * FROM table_name LIMIT 10"

# Execute SQL query from a file
supahelp query query.sql

# Show help
supahelp help
```

### Importing Data

The import command allows you to execute SQL statements from a file against your Supabase database:

```bash
supahelp import siswa_data.sql
```

Features:
- Executes the SQL file in a transaction for safety
- Detects various SQL statement types (INSERT, DELETE, UPDATE, etc.)
- Provides detailed feedback on execution progress
- Allows you to continue or rollback if errors occur

### Exporting Data

The dump command exports your entire Supabase database to an SQL file:

```bash
# Export to a timestamped file
supahelp dump

# Export to a specific file
supahelp dump my_backup.sql

# Export only the schema (no data)
supahelp dump --schema-only

# Export only the schema to a specific file
supahelp dump --schema-only schema_only.sql

# Export only a specific table
supahelp dump --table siswa

# Export only a specific table to a specific file
supahelp dump --table siswa siswa_backup.sql

# Export only the schema of a specific table
supahelp dump --table siswa --schema-only
```

Features:
- Exports all tables in the public schema
- Includes table structure (CREATE TABLE statements)
- Includes primary key definitions
- Includes all data as INSERT statements (unless using --schema-only)
- Adds DROP TABLE statements for clean imports
- Option to export only the schema without data (--schema-only)
- Option to export only a specific table (--table)

### Executing Queries

The query command allows you to execute SQL queries against your Supabase database:

```bash
# Execute a query directly from the command line
supahelp query "SELECT * FROM siswa LIMIT 10"

# Execute a query from a file
supahelp query query.sql

# Output results in CSV format
supahelp query --csv "SELECT * FROM siswa LIMIT 10"

# Output results from a file in CSV format
supahelp query --csv query.sql
```

Features:
- Execute SQL queries directly from the command line
- Execute SQL queries from a file
- Display results in a formatted table
- Option to output results in CSV format (--csv)
- Shows execution time and row count
- Handles large result sets with column width truncation

## Examples

### Example 1: Importing Student Data

```bash
supahelp import siswa_data.sql
```

This will:
1. Connect to your Supabase database
2. Read the SQL file `siswa_data.sql`
3. Execute the SQL statements in a transaction
4. Report on the success or failure of each statement

### Example 2: Backing Up Your Database

```bash
# Full backup with data
supahelp dump school_backup.sql

# Schema-only backup
supahelp dump --schema-only school_schema.sql
```

For a full backup, this will:
1. Connect to your Supabase database
2. Retrieve the structure of all tables
3. Retrieve all data from all tables
4. Write CREATE TABLE, PRIMARY KEY, and INSERT statements to `school_backup.sql`

For a schema-only backup, this will:
1. Connect to your Supabase database
2. Retrieve the structure of all tables
3. Write CREATE TABLE and PRIMARY KEY statements to `school_schema.sql` (without any data)

### Example 3: Truncating a Table

Create a file named `truncate_table.sql` with the content:

```sql
-- SQL to empty a table
TRUNCATE TABLE pengajar_pelajaran;
```

Then run:

```bash
supahelp import truncate_table.sql
```

### Example 4: Running Queries

#### Direct Query from Command Line

```bash
# Get the count of students by gender
supahelp query "SELECT gender, COUNT(*) FROM siswa GROUP BY gender"
```

This will:
1. Connect to your Supabase database
2. Execute the SQL query
3. Display the results in a formatted table

#### Query from a File

Create a file named `student_report.sql` with the content:

```sql
-- Get detailed student report
SELECT
  s.nama AS "Student Name",
  s.gender AS "Gender",
  h.nama AS "Halaqoh",
  p.nama AS "Teacher"
FROM
  siswa s
  JOIN halaqoh h ON s.halaqoh_id = h.id
  JOIN pengajar p ON h.pengajar_id = p.id
ORDER BY
  s.nama
LIMIT 20;
```

Then run:

```bash
supahelp query student_report.sql
```

This will:
1. Connect to your Supabase database
2. Read the SQL file
3. Execute the SQL query
4. Display the results in a formatted table

#### Exporting Query Results to CSV

```bash
# Export query results to CSV format
supahelp query --csv "SELECT * FROM siswa" > students.csv
```

This will:
1. Connect to your Supabase database
2. Execute the SQL query
3. Output the results in CSV format
4. Save the output to a file named `students.csv`

## Common Tasks

### Creating a Backup

```bash
# Create a full backup with timestamp
supahelp dump

# Create a schema-only backup with timestamp
supahelp dump --schema-only

# Schedule daily full backups (using cron on Linux/Mac)
0 0 * * * /path/to/supahelp dump /path/to/backups/supabase_$(date +\%Y\%m\%d).sql

# Schedule weekly schema-only backups (using cron on Linux/Mac)
0 0 * * 0 /path/to/supahelp dump --schema-only /path/to/backups/supabase_schema_$(date +\%Y\%m\%d).sql
```

### Restoring from a Backup

```bash
supahelp import backup_file.sql
```

### Migrating Data Between Environments

```bash
# Export from production
export SUPABASE_CONNECTION_STRING='postgresql://postgres:password@production-db:5432/postgres'
supahelp dump prod_data.sql

# Import to development
export SUPABASE_CONNECTION_STRING='postgresql://postgres:password@development-db:5432/postgres'
supahelp import prod_data.sql
```

### Running Database Maintenance Tasks

```bash
# Check for orphaned records
supahelp query "
  SELECT s.id, s.nama
  FROM siswa s
  LEFT JOIN halaqoh h ON s.halaqoh_id = h.id
  WHERE h.id IS NULL
"

# Find duplicate records
supahelp query "
  SELECT nama, COUNT(*)
  FROM siswa
  GROUP BY nama
  HAVING COUNT(*) > 1
"

# Get table statistics
supahelp query "
  SELECT
    table_name,
    pg_size_pretty(pg_total_relation_size(table_name::text)) as size,
    (SELECT COUNT(*) FROM \"\${table_name}\") as row_count
  FROM information_schema.tables
  WHERE table_schema = 'public'
  ORDER BY pg_total_relation_size(table_name::text) DESC
"
```

### Generating Reports

```bash
# Create a student report and save as CSV
supahelp query --csv "
  SELECT
    s.nama AS student_name,
    s.gender,
    h.nama AS halaqoh_name,
    p.nama AS teacher_name
  FROM
    siswa s
    JOIN halaqoh h ON s.halaqoh_id = h.id
    JOIN pengajar p ON h.pengajar_id = p.id
  ORDER BY
    s.nama
" > student_report.csv

# Save complex queries in SQL files for reuse
echo "
  SELECT
    DATE(waktu_setoran) AS date,
    jenis,
    COUNT(*) AS submission_count
  FROM
    setoran
  WHERE
    waktu_setoran >= CURRENT_DATE - INTERVAL '30 days'
  GROUP BY
    DATE(waktu_setoran), jenis
  ORDER BY
    date DESC, jenis
" > submission_report.sql

# Run the saved query
supahelp query submission_report.sql
```

## Troubleshooting

### Connection Issues

If you encounter connection issues, the script will now provide detailed error messages with instructions on how to set up your connection string correctly.

Example error message:
```
Database connection error. Please check your connection settings.
To set the connection string, use the SUPABASE_CONNECTION_STRING environment variable:
export SUPABASE_CONNECTION_STRING='postgresql://postgres:password@your-supabase-host:5432/postgres'
Make sure your database is running and accessible.
```

Common connection issues to check:

1. Verify your connection string is correct (format: `postgresql://username:password@host:port/database`)
2. Check that your IP is allowed in Supabase's network settings
3. Ensure your database password is correct
4. Check if your Supabase project is active
5. Verify that the database port is open and accessible from your network

### Import Errors

If you encounter errors during import:

1. Check the SQL syntax in your file
2. Look for conflicting primary keys or constraint violations
3. Consider using the `--if-exists` flag in your DROP statements
4. For large imports, consider splitting the file into smaller chunks

## Advanced Usage

### Selective Table Export

You can export only a specific table using the `--table` option:

```bash
# Export only the 'siswa' table
supahelp dump --table siswa

# Export only the 'pengajar' table schema
supahelp dump --table pengajar --schema-only
```

This is useful when you only need to backup or migrate a single table instead of the entire database.

If you need to export multiple specific tables but not all tables, you can run the command multiple times with different table names, or create a shell script to automate this:

```bash
#!/bin/bash
# export_selected_tables.sh
tables=("siswa" "pengajar" "halaqoh" "setoran")
timestamp=$(date +"%Y-%m-%d_%H-%M-%S")

for table in "${tables[@]}"; do
  echo "Exporting table: $table"
  supahelp dump --table "$table" "${table}_${timestamp}.sql"
done

echo "Export completed!"
```

### Advanced Queries

You can use the query functionality for more complex database operations:

```bash
# Create a temporary table and query it
supahelp query "
  CREATE TEMPORARY TABLE temp_stats AS
  SELECT
    h.nama AS halaqoh_name,
    COUNT(s.id) AS student_count,
    SUM(CASE WHEN s.gender = 'laki-laki' THEN 1 ELSE 0 END) AS male_count,
    SUM(CASE WHEN s.gender = 'perempuan' THEN 1 ELSE 0 END) AS female_count
  FROM
    halaqoh h
    LEFT JOIN siswa s ON h.id = s.halaqoh_id
  GROUP BY
    h.nama;

  SELECT * FROM temp_stats ORDER BY student_count DESC;
"

# Use SQL functions and window functions
supahelp query "
  SELECT
    s.nama,
    COUNT(st.id) AS setoran_count,
    MIN(st.waktu_setoran) AS first_setoran,
    MAX(st.waktu_setoran) AS last_setoran,
    RANK() OVER (ORDER BY COUNT(st.id) DESC) AS rank
  FROM
    siswa s
    LEFT JOIN setoran st ON s.id = st.siswa_id
  GROUP BY
    s.id, s.nama
  ORDER BY
    setoran_count DESC
  LIMIT 10;
"
```

### Creating SQL Script Files

You can create SQL script files for complex operations:

```sql
-- analysis.sql
-- First, create a temporary table with setoran statistics
CREATE TEMPORARY TABLE setoran_stats AS
SELECT
  s.siswa_id,
  s.jenis,
  COUNT(*) AS count,
  MIN(s.waktu_setoran) AS first_setoran,
  MAX(s.waktu_setoran) AS last_setoran,
  AVG(s.halaman_akhir - s.halaman_awal + 1) AS avg_pages
FROM
  setoran s
GROUP BY
  s.siswa_id, s.jenis;

-- Then, join with student information
SELECT
  si.nama AS student_name,
  h.nama AS halaqoh_name,
  p.nama AS teacher_name,
  COALESCE(sz.count, 0) AS ziyadah_count,
  COALESCE(sr.count, 0) AS rabth_count,
  COALESCE(si2.count, 0) AS ikhtibar_count,
  COALESCE(sz.count, 0) + COALESCE(sr.count, 0) + COALESCE(si2.count, 0) AS total_count
FROM
  siswa si
  JOIN halaqoh h ON si.halaqoh_id = h.id
  JOIN pengajar p ON h.pengajar_id = p.id
  LEFT JOIN setoran_stats sz ON si.id = sz.siswa_id AND sz.jenis = 'ziyadah'
  LEFT JOIN setoran_stats sr ON si.id = sr.siswa_id AND sr.jenis = 'rabth'
  LEFT JOIN setoran_stats si2 ON si.id = si2.siswa_id AND si2.jenis = 'ikhtibar'
ORDER BY
  total_count DESC
LIMIT 20;
```

Then run it with:

```bash
supahelp query analysis.sql
```

### Handling Large Datasets

For very large databases:

1. Consider using the `COPY` command instead of INSERT statements
2. Implement pagination when retrieving data
3. Use streams for writing large SQL files
4. Consider compressing the output file
5. For large query results, use the CSV output option and process with other tools

## Security Considerations

- The script uses environment variables for database credentials, which is more secure than hardcoding them.
- Be careful not to expose your environment variables in logs or shared terminal sessions.
- Avoid storing backup files with sensitive data in unsecured locations.
- Consider encrypting backup files that contain sensitive information.
- Be careful when sharing SQL files as they may contain sensitive data.
- For production environments, consider using a secrets management solution instead of environment variables.

## Publishing to npm

If you want to publish this package to npm, follow these steps:

1. Make sure you have an npm account and are logged in:
   ```bash
   npm login
   ```

2. Update the package.json file with your information:
   - Update the "author" field with your name and email
   - Consider updating the "license" field if needed
   - Add a "repository" field pointing to your GitHub repository

3. Publish the package:
   ```bash
   npm publish
   ```

4. To update the package later:
   - Update the version in package.json (follow semantic versioning)
   - Run `npm publish` again

## Contributing

Feel free to modify this script to suit your specific needs. Some potential improvements:

- Add support for schema migrations
- Implement data transformations during import/export
- Add support for database comparison and synchronization
- Implement progress bars for large operations
- Add support for saving query results to files directly
- Implement query pagination for very large result sets
- Add support for parameterized queries
- Implement query history and favorites
- Add support for exporting multiple specific tables in a single command

## License

This script is provided as-is with no warranty. Use at your own risk.
