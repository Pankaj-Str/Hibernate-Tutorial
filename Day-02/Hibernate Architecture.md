
# Hibernate Architecture

Hibernate works like a **bridge** between your Java application and the Database. It hides all the complex JDBC code and lets you work only with Java objects.

Let’s understand the architecture in a very simple way, step by step.

### 1. Configuration (Setup Phase)
This is the first step where Hibernate gets all the necessary information to connect to the database.

- You provide details like:
  - Database URL
  - Username and Password
  - Database Dialect (e.g., MySQL, PostgreSQL)
  - Mapping files or annotated classes

This information is usually kept in:
- `hibernate.cfg.xml` file, or
- `application.properties` (in Spring Boot)

Once Hibernate reads this configuration, it prepares itself to work with the database.

### 2. SessionFactory (The Factory)
- After reading the configuration, Hibernate creates a **SessionFactory**.
- **SessionFactory** is like a **big factory** that knows everything about your database and mappings.
- It is created only **once** when your application starts.
- It is **thread-safe** and **immutable** (cannot be changed).
- Usually, you have **one SessionFactory per database**.

Think of SessionFactory as the "manager" or "head office" of Hibernate.

### 3. Session (The Worker)
- Whenever you want to talk to the database (save, fetch, update, or delete data), you ask the SessionFactory to give you a **Session**.
- **Session** is a short-lived object. It is like a **worker** who actually does the job.
- You open a Session, do your work, and then close it.
- Each user request or thread usually gets its own Session.
- It provides important methods like:
  - `session.save(object)`
  - `session.get(Class, id)`
  - `session.update(object)`
  - `session.delete(object)`

**Important**: Session is **not thread-safe**. That’s why we create a new Session for every operation.

### 4. Transaction
- A Transaction is used when you want to perform multiple operations as a single unit.
- Example: Transfer money from Account A to Account B → Both debit and credit must happen together.
- If anything fails, the whole transaction is rolled back.
- You start a transaction with `session.beginTransaction()` and commit it with `tx.commit()`.

### 5. Query / Criteria API
- When you want to search or fetch data using conditions, you use:
  - **HQL (Hibernate Query Language)** – Similar to SQL but uses Java class names instead of table names.
  - **Criteria API** – Modern, type-safe way to build queries in Java code.

### Simple Flow of Hibernate Architecture

```
Java Application
       ↓
Configuration (hibernate.cfg.xml or properties)
       ↓
SessionFactory (created once)
       ↓
     Open Session
       ↓
   Begin Transaction
       ↓
   Perform Operations (save, get, update, delete)
       ↓
   Commit / Rollback Transaction
       ↓
   Close Session
```

### Visual Analogy (Easy to Remember)

Imagine a company:

- **Configuration** → Company rules and setup files
- **SessionFactory** → Head Office / Factory (built once)
- **Session** → Individual Employee / Worker (works for one task)
- **Transaction** → A Project that must be completed fully or cancelled
- **Database** → Warehouse where actual data is stored

### Summary

1. You give configuration (how to connect to DB).
2. Hibernate creates **SessionFactory** (smart factory).
3. You open a **Session** from the factory to do work.
4. You use **Transaction** to make operations safe.
5. You perform CRUD using Java objects.
6. Hibernate automatically converts everything into SQL and talks to the database.

Hibernate does all the heavy work (SQL generation, connection management, object mapping) in the background.

---

