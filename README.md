# SampleDemo — Phase 1

A Spring Boot (v4.1.1, Java 21) service backed by MySQL 8.0, with database schema
managed by Flyway migrations.

## Phase 1 — What was done

Phase 1 sets up the database layer and gets the application starting cleanly against
a MySQL container.

1. **MySQL via Docker** — runs MySQL 8.0 in a container (`docker-compose.yml`).
2. **Database connectivity** — configured the JDBC datasource in `application.yml`.
3. **Flyway migrations** — added versioned SQL scripts that build the schema on startup.
4. **Fixed app startup** — resolved two issues that stopped the app from running
   (see [Troubleshooting](#troubleshooting)).

## Prerequisites

- Docker
- JDK 21
- Maven wrapper (`mvnw`, included)

## Getting started

1. **Start MySQL:**
   ```bash
   docker compose up -d
   ```

2. **Run the application:**
   ```bash
   ./mvnw spring-boot:run
   ```

3. **App URL:** http://localhost:8081

On startup, Flyway automatically applies any pending migrations before the app
finishes loading.

## Configuration

Key settings in `src/main/resources/application.yml`:

| Setting | Value | Purpose |
|---------|-------|---------|
| `server.port` | `8081` | HTTP port the app listens on |
| `spring.datasource.url` | `jdbc:mysql://localhost:3306/myappdb?...` | MySQL connection |
| `spring.jpa.hibernate.ddl-auto` | `validate` | Hibernate only validates; Flyway owns the schema |
| `spring.flyway.locations` | `classpath:db/migration` | Where migration scripts live |

**JDBC URL flags:**
- `useSSL=false` — no TLS (fine for local dev; use `true` in production).
- `allowPublicKeyRetrieval=true` — required by MySQL 8's `caching_sha2_password` auth.

## Database migrations

Flyway scripts live in `src/main/resources/db/migration/` and run in version order.

| Version | File | Change |
|---------|------|--------|
| V1 | `V1__create_users_table.sql` | Create `users` table |
| V2 | `V2__add_status_to_users.sql` | Add `status` column to `users` |
| V3 | `V3__create_orders_table.sql` | Create `orders` table (FK → `users`) |

**Naming convention:** `V<version>__<description>.sql` (double underscore). Add new
changes as new files with the next version number — never edit an already-applied file.

**Verify:** connect to `myappdb` (user `myuser`) in your IDE's database tool. You
should see `users`, `orders`, and Flyway's own `flyway_schema_history` table.

## Troubleshooting

Two issues were fixed during Phase 1:

1. **`Public Key Retrieval is not allowed`** — MySQL 8 uses SHA-2 auth. Fixed by
   adding `allowPublicKeyRetrieval=true` to the JDBC URL and aligning the
   database/username/password with `docker-compose.yml`.

2. **Migrations never ran (no Flyway logs)** — Spring Boot 4 splits auto-configuration
   into per-integration modules. `flyway-core`/`flyway-mysql` alone are not enough; the
   `spring-boot-flyway` module is required to auto-configure and trigger Flyway. Added it
   to `pom.xml`.

## Tech stack

- Spring Boot 4.1.1 (Web MVC, Data JPA)
- Java 21
- MySQL 8.0
- Flyway 12.4.0
