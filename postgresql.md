## Prerequesties

```
1. sudo apt update
2. sudo apt install python3-venv python3-dev libpq-dev postgresql postgresql-contrib
3. sudo -u postgres psql

3.1. CREATE DATABASE panomiqDB;
3.2. CREATE USER radhe WITH PASSWORD 'password';
3.3. ALTER ROLE radhe SET client_encoding TO 'utf8';
3.4. ALTER ROLE radhe SET default_transaction_isolation TO 'read committed';
3.5. ALTER ROLE radhe SET timezone TO 'UTC';
3.6. GRANT ALL PRIVILEGES ON DATABASE panomiqDB TO radhe;
3.7. \q
```
