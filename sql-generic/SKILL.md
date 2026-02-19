---
name: sql-generic
description: Writes and reviews SQL for any project. Discovers DB credentials and schema from the codebase (Node, PHP, or other), then generates and runs queries. Use when the user mentions SQL, database, queries, migrations, schema, or ORM (Prisma, Laravel, TypeORM, etc.).
---

# SQL (Any Project)

## Workflow

Follow these phases when helping with SQL, regardless of project language or framework.

### Phase 1: Credentials

- Determine how the project supplies database credentials:
  - **Node**: Often `.env` with `DATABASE_URL` (e.g. Postgres connection string).
  - **PHP**: `.env`, `config/database.php`, or env vars such as `DB_HOST`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`.
  - **Other**: Look for `.env`, `config/`, or env vars prefixed with `DB_`, `DATABASE_`, or `POSTGRES_`/`MYSQL_`.
- Resolve credentials from the project root or the service that owns the DB (e.g. `project-name/.env` in a monorepo).
- Do not log, echo, or expose credentials in output.

### Phase 2: Base table

- If the user’s request clearly references tables, use those as the base.
- If no table is specified, ask which table they want to use before writing the query.

### Phase 3: Schema discovery

- Find schema definitions in the project so column names, types, and relations are correct:
  - **Prisma**: `**/schema.prisma`
  - **Laravel / PHP**: `**/migrations/*.php` or Eloquent models
  - **TypeORM / other ORMs**: entity files or migration SQL
  - **Raw SQL**: `**/*.sql` (DDL or schema dumps)
- Use the discovered schema to align queries and avoid invalid columns or types.

### Phase 4: Query and run

- Generate SQL that matches the request and the discovered schema.
- Run with the appropriate client, using credentials from Phase 1:
  - **PostgreSQL** (e.g. `DATABASE_URL` or `POSTGRES_*`):
    ```bash
    psql "${DATABASE_URL}" -c 'SELECT ...'
    ```
  - **MySQL** (e.g. `DB_*` or `MYSQL_*`):
    ```bash
    mysql -h "${DB_HOST}" -u "${DB_USERNAME}" -p"${DB_PASSWORD}" "${DB_DATABASE}" -e "SELECT ..."
    ```
- Quote the SQL safely for the shell (prefer single-quoted strings for the SQL when using `-c`/`-e` to avoid shell expansion).

## Notes

- If the project uses multiple databases or services, ask which one to use when it’s ambiguous.
- For migrations or DDL, prefer the project’s migration tool (e.g. Prisma migrate, Laravel migrations) when the user wants changes applied through the framework.
