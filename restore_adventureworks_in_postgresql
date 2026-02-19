# Install Adventureworks in Postgresql                                                                                                                                    

1* Go to https://github.com/lorint/AdventureWorks-for-Postgres

```bash
wget https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks-oltp-install-script.zip
```

Extract into one single Directory and paste the `install.sql` and the `update_csvs.rb` files from https://github.com/lorint/AdventureWorks-for-Postgres in same Directory.

2. Install Ruby if you haven't
```bash
sudo apt install ruby
```

3. Go to Directory where you downloaded all files and
```bash
ruby update_csvs.rb  # This will format files in place

# Create dedicated database for AdventureWorks
createdb -U admin -h localhost adventureworks

# Import
psql -U admin -d adventureworks -h localhost -f install.sql

# Connect and explore
psql -U admin -d adventureworks -h localhost
```

## Basic queries

```sql
-- List all schemas
\dn

-- List tables accross all schemas
\dt *.*

-- List ALL tables in ALL Schemas
SELECT schemaname, tablename
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY schemaname, tablename;

-- Check what tables exist in each schema
\dt sales.*
\dt person.*
