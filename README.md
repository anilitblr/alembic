# Alembic

- [Alembic](#alembic)
  - [Setup a local (development) Postgres database](#setup-a-local-development-postgres-database)
      - [Create the file repository configuration](#create-the-file-repository-configuration)
      - [Import the repository signing key](#import-the-repository-signing-key)
      - [Update the package lists](#update-the-package-lists)
      - [Install PostgresSQL.](#install-postgressql)
      - [Create .pgpass file for passwordless connection](#create-pgpass-file-for-passwordless-connection)
      - [Change .pgpass file permission](#change-pgpass-file-permission)
      - [Start postgresql service](#start-postgresql-service)
      - [Create user](#create-user)
      - [Create database](#create-database)
      - [Drop database](#drop-database)
      - [Grant all privileges on database example](#grant-all-privileges-on-database-example)
      - [Execute SQL Commands from psql](#execute-sql-commands-from-psql)
  - [Install and setup alembic](#install-and-setup-alembic)
      - [Install virtual environment and activate](#install-virtual-environment-and-activate)
      - [Install alembic](#install-alembic)
      - [initialize alembic](#initialize-alembic)
      - [Database connection\_string](#database-connection_string)
      - [Get the database credentials from the .env](#get-the-database-credentials-from-the-env)
      - [Create a Migration Scripts](#create-a-migration-scripts)
      - [Create users table](#create-users-table)
      - [Running Migration](#running-migration)
      - [Delete all tables in the database](#delete-all-tables-in-the-database)
      - [Datatypes](#datatypes)
- [References](#references)

## Setup a local (development) Postgres database

#### Create the file repository configuration

```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```

#### Import the repository signing key

```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```

#### Update the package lists

```
sudo apt-get update;
```

#### Install PostgresSQL.

```
sudo apt-get -y install postgresql-15 postgresql-client-15;
```

#### Create .pgpass file for passwordless connection

```
echo "
#hostname:port:database:username:password
localhost:5432:example:ex:pass.word" | tee ~/.pgpass
```

#### Change .pgpass file permission

```
chmod 600 ~/.pgpass
```

#### Start postgresql service

```bash
sudo systemctl enable --now postgresql.service;
sudo systemctl start postgresql.service;
sudo systemctl restart postgresql.service;
sudo systemctl stop postgresql.service;
```

#### Create user

```sql
sudo -u postgres psql -c "CREATE USER ex WITH PASSWORD 'pass.word';"
```

#### Create database

```sql
sudo -u postgres psql -c "CREATE DATABASE example WITH OWNER=ex;"
```

#### Drop database

```sql
sudo -u postgres psql -c "DROP database example;"
```

#### Grant all privileges on database example

```sql
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE example to ex;"
```

#### Execute SQL Commands from psql

```bash
psql -h localhost -p 5432 -U ex -d example -w -c "\dt;"
psql -h localhost -p 5432 -U ex -d example -w -c "\dS device;"
psql -h localhost -p 5432 -U ex -d example -w -c "SELECT * FROM users;"
psql -h localhost -p 5432 -U ex -d example -w -c "<Other SQL queries>;"
```

## Install and setup alembic

#### Install virtual environment and activate

```bash
python3.10 -m venv venv;
source venv/bin/activate;
```

#### Install alembic

```bash
python3.10 -m pip install --upgrade pip
pip install alembic;
```

#### initialize alembic

```bash
alembic init alembic
```

#### Database connection_string

```
# To be updated in alembic.ini file.
sqlalchemy.url = postgresql://ex:pass.word@localhost/example
```

#### Get the database credentials from the .env

```python
# To be updated in env.py file.

import os
import sys
from dotenv import load_dotenv


BASE_DIR_PTAH = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
ENV = os.getenv('ENV')

"""
export ENV=development
export ENV=test
export ENV=production"""

if ENV == 'development' or ENV == 'test' or ENV == 'production':
    DOTENV_FILE_PATH = os.path.join(BASE_DIR_PTAH + '/app/' + '.env.{}'.format(ENV))
    load_dotenv(DOTENV_FILE_PATH)
    print(f"Loading .env.{ENV}")
else:
    print("ENV is not set. Please set it to 'development', 'test', or 'production'.")
    sys.exit()

DATABASE_CONNECTION_STRING = (
    "postgresql://"
    + os.getenv("DB_USERNAME")
    + ":"
    + os.getenv("DB_PASSWORD")
    + "@"
    + os.getenv("DB_HOSTNAME")
    # + ":"
    # + os.getenv("DB_PORT")
    + "/"
    + os.getenv("DATABASE")
    )

config.set_main_option("sqlalchemy.url", DATABASE_CONNECTION_STRING)
```

#### Create a Migration Scripts

```bash
alembic revision -m "create table users";
```

#### Create users table

```python
def upgrade():
    op.create_table(
        'users',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('name', sa.String(50), nullable=False),
        sa.Column('description', sa.Unicode(200)),
    )

def downgrade():
    op.drop_table('users')
```

#### Running Migration

```bash
alembic upgrade head;
```

#### Delete all tables in the database

```bash
alembic downgrade base;
```

#### Datatypes

```python
sa.UUID
sa.VARCHAR
sa.Integer
sa.Numeric
sa.Text
sa.Boolean
sa.DateTime(timezone=True)
sa.DateTime(timezone=True), server_default=func.now()
sa.UniqueConstraint('one')
sa.UniqueConstraint('one', 'two')
nullable=False
nullable=True
sa.Unicode
server_default=sa.schema.DefaultClause("0")
server_default=sa.schema.DefaultClause("false")
op.create_index(op.f('users_id_index'), 'users', ['id'])
op.create_unique_constraint(op.f('users_id_index'), 'users', ["user1_id", "user2_id", "user3_id"])
```

# References

ref: https://alembic.sqlalchemy.org/en/latest/tutorial.html