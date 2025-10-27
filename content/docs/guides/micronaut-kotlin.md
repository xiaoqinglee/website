---
title: Connect a Micronaut Kotlin application to Neon Postgres
subtitle: Learn how to make server-side queries to Postgres from a Micronaut Kotlin application
enableTableOfContents: true
updatedOn: '2025-06-30T11:30:21.907Z'
---

[Micronaut](https://micronaut.io/) is a modern, JVM-based, full-stack framework for building modular, easily testable microservice and serverless applications. This guide describes how to create a Neon Postgres database and connect to it from a Micronaut Kotlin application.

The final application will expose REST endpoints to perform CRUD (Create, Read, Update, Delete) operations on a `book` table in your Neon database.

To create a Neon project and access it from a Micronaut Kotlin application, you will:

1. [Create a Neon project](#create-a-neon-project)
2. [Create a Micronaut Kotlin project](#create-a-micronaut-kotlin-project)
3. [Configure your database connection](#configure-your-database-connection)
4. [Build the application components](#build-the-application-components)
5. [Run and test the application](#run-and-test-the-application)

<Steps>

## Create a Neon project

If you do not have one already, create a Neon project.

1. Navigate to the [Projects](https://console.neon.tech/app/projects) page in the Neon Console.
2. Click **New Project**.
3. Specify your project settings and click **Create Project**.

Save your connection details. You will need them in a later step.

## Create a Micronaut Kotlin project

You can create a new Micronaut project using either the Micronaut CLI or the [Micronaut Launch](https://launch.micronaut.io/) website.

For this guide, we will use the Micronaut CLI.

Run the following command in your terminal. This command creates a new application and includes features for PostgreSQL connectivity, JDBC connection pooling (Hikari), database migrations (Flyway), data access, and YAML configuration.

```bash
mn create-app with-micronaut-kotlin --lang=kotlin --jdk=21 --features=postgres,jdbc-hikari,flyway,data-jdbc,yaml
```

After creating the project, open the `build.gradle.kts` file and add the following configuration inside the `kotlin` block to ensure compatibility with JDK 21 and prevent potential build errors:

```kotlin
// build.gradle.kts

kotlin {
    jvmToolchain(21)
}
```

## Configure your database connection

The project creation process generated a configuration file at `src/main/resources/application.yml`. You need to edit this file to add your Neon database credentials.

Add the `url`, `username`, and `password` fields under the `datasources.default` section. Your updated `application.yml` file should look like this:

```yaml {8-10}
# src/main/resources/application.yml

micronaut:
  application:
    name: with-micronaut-kotlin
datasources:
  default:
    url: 'jdbc:postgresql://<your-endpoint.neon.tech>/<dbname>?sslmode=require&channelBinding=require'
    username: '<your-db-username>'
    password: '<your-db-password>'
    driver-class-name: org.postgresql.Driver
    db-type: postgres
    dialect: POSTGRES
flyway:
  datasources:
    default:
      enabled: true
```

> Replace `<your-endpoint.neon.tech>`, `<dbname>`, `<your-db-username>`, and `<your-db-password>` with your actual Neon database connection details you saved earlier.

## Build the application components

Now you can create the components for a simple book inventory API: an entity, a repository, a controller, and a database migration script.

### 1. Create the database schema with Flyway

Flyway handles database migrations automatically when the application starts. Create a SQL file at `src/main/resources/db/migration/V1__create_book_table.sql` to define your table schema and add some initial data.

```sql
-- src/main/resources/db/migration/V1__create_book_table.sql

CREATE TABLE IF NOT EXISTS book (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(255) NOT NULL
);

INSERT INTO book (title, author) VALUES ('The Hobbit', 'J.R.R. Tolkien');
INSERT INTO book (title, author) VALUES ('1984', 'George Orwell');
```

### 2. Create the entity

Create a data class that maps to the `book` table. The `@Serdeable` annotations are required for Micronaut to handle JSON serialization and deserialization for your API.

```kotlin
// src/main/kotlin/com/example/entity/Book.kt

package com.example.entity

import io.micronaut.data.annotation.GeneratedValue
import io.micronaut.data.annotation.Id
import io.micronaut.data.annotation.MappedEntity
import io.micronaut.serde.annotation.Serdeable

@MappedEntity
@Serdeable
data class Book(
    @field:Id
    @field:GeneratedValue
    var id: Long? = null,
    var title: String,
    var author: String
)
```

### 3. Create the repository

Create a repository interface that extends `CrudRepository`. This interface provides CRUD operations for the `Book` entity.

```kotlin
// src/main/kotlin/com/example/repository/BookRepository.kt

package com.example.repository

import com.example.entity.Book
import io.micronaut.data.jdbc.annotation.JdbcRepository
import io.micronaut.data.model.query.builder.sql.Dialect
import io.micronaut.data.repository.CrudRepository

@JdbcRepository(dialect = Dialect.POSTGRES)
interface BookRepository : CrudRepository<Book, Long> {
    fun findByTitleContains(title: String): List<Book>
}
```

### 4. Create the controller

Finally, create a controller to expose the REST endpoints for interacting with the books.

```kotlin
// src/main/kotlin/com/example/controller/BookController.kt

package com.example.controller

import com.example.entity.Book
import com.example.repository.BookRepository
import io.micronaut.http.annotation.*
import io.micronaut.scheduling.TaskExecutors
import io.micronaut.scheduling.annotation.ExecuteOn

@Controller("/books")
class BookController(private val bookRepository: BookRepository) {

    @Get
    @ExecuteOn(TaskExecutors.IO)
    fun getAll(): List<Book> {
        return bookRepository.findAll().toList()
    }

    @Get("/{id}")
    @ExecuteOn(TaskExecutors.IO)
    fun getById(id: Long): Book? {
        return bookRepository.findById(id).orElse(null)
    }

    @Post
    @ExecuteOn(TaskExecutors.IO)
    fun save(@Body book: Book): Book {
        return bookRepository.save(book)
    }
}
```

## Run and test the application

You are now ready to run your application.

1.  Start the application using the Gradle wrapper:

    ```bash
    ./gradlew run
    ```

    You should see output similar to the following:

    ```bash
    $ ./gradlew run

    [test-resources-service] 15:48:33.940 [main] INFO  i.m.c.DefaultApplicationContext$RuntimeConfiguredEnvironment - Established active environments: [test]

    > Task :run
    __  __ _                                  _
    |  \/  (_) ___ _ __ ___  _ __   __ _ _   _| |_
    | |\/| | |/ __| '__/ _ \| '_ \ / _` | | | | __|
    | |  | | | (__| | | (_) | | | | (_| | |_| | |_
    |_|  |_|_|\___|_|  \___/|_| |_|\__,_|\__,_|\__|
    15:48:43.830 [main] INFO  com.zaxxer.hikari.HikariDataSource - HikariPool-1 - Starting...
    15:48:45.974 [main] INFO  com.zaxxer.hikari.pool.HikariPool - HikariPool-1 - Added connection org.postgresql.jdbc.PgConnection@30506c0d
    15:48:45.975 [main] INFO  com.zaxxer.hikari.HikariDataSource - HikariPool-1 - Start completed.
    15:48:46.126 [main] INFO  i.m.flyway.AbstractFlywayMigration - Running migrations for database with qualifier [default]
    15:48:46.298 [main] INFO  org.flywaydb.core.FlywayExecutor - Database: jdbc:postgresql://endpoint.neon.tech/examples?sslmode=require&channelBinding=require (PostgreSQL 17.5)
    15:48:48.110 [main] INFO  o.f.c.i.s.JdbcTableSchemaHistory - Schema history table "public"."flyway_schema_history" does not exist yetn
    15:48:48.250 [main] INFO  o.f.core.internal.command.DbValidate - Successfully validated 1 migration (execution time 00:00.432s)
    15:48:49.524 [main] INFO  o.f.c.i.s.JdbcTableSchemaHistory - Creating Schema History table "public"."flyway_schema_history" ...
    15:48:51.817 [main] INFO  o.f.core.internal.command.DbMigrate - Current version of schema "public": << Empty Schema >>
    15:48:52.243 [main] INFO  o.f.core.internal.command.DbMigrate - Migrating schema "public" to version "1 - create book table"
    15:48:54.757 [main] INFO  o.f.core.internal.command.DbMigrate - Successfully applied 1 migration to schema "public", now at version v1 (execution time 00:00.969s)
    15:48:55.841 [main] INFO  io.micronaut.runtime.Micronaut - Startup completed in 12788ms. Server Running: http://localhost:8080 :run
    <============-> 92% EXECUTING [38s]
    > :run
    > IDLE
    ```

    The logs indicate the following sequence of events:
    - HikariCP initializes the connection pool to the Neon Postgres database.
    - Flyway checks the database schema and found that the `flyway_schema_history` table does not exist.
    - Flyway creates the `flyway_schema_history` table and applies the migrations present in the migration folder.
    - The `book` table is created as per the migration script (i.e., `V1__create_book_table.sql`).
    - The application starts successfully and is ready to handle requests.

    Now with the application running, you can test the API endpoints.

2.  Test the API endpoints using `curl` or any API client:

    ```bash
    # Get all books
    curl http://localhost:8080/books
    # Expected Output: [{"id":1,"title":"The Hobbit","author":"J.R.R. Tolkien"},{"id":2,"title":"1984","author":"George Orwell"}]

    # Get a specific book by ID
    curl http://localhost:8080/books/1
    # Expected Output: {"id":1,"title":"The Hobbit","author":"J.R.R. Tolkien"}

    # Create a new book
    curl -X POST \
      -H "Content-Type: application/json" \
      -d '{"title":"The Great Gatsby","author":"F. Scott Fitzgerald"}' \
      http://localhost:8080/books
    # Expected Output: {"id":3,"title":"The Great Gatsby","author":"F. Scott Fitzgerald"}
    ```

You have successfully connected a Micronaut Kotlin application to your Neon Postgres database!

## Source code

You can find the source code for the application described in this guide on GitHub.

<DetailIconCards>
<a href="https://github.com/neondatabase/examples/tree/main/with-micronaut-kotlin" description="Get started with Micronaut Kotlin and Neon" icon="github">Get started with Micronaut Kotlin and Neon</a>
</DetailIconCards>
</Steps>

<NeedHelp/>
