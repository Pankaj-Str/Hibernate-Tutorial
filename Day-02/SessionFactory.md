# SessionFactory

---

### What is SessionFactory in Hibernate?

**SessionFactory** is one of the most important objects in Hibernate. It is often called the **"Heart"** or **"Backbone"** of Hibernate.

In simple words:
> **SessionFactory** is a heavy, intelligent factory that creates and manages **Session** objects. It is created only **once** when your application starts and stays alive throughout the entire application lifecycle.

### Why is SessionFactory So Important?

Think of it like this:

- **SessionFactory** = Big Factory / Head Office  
- **Session** = Worker / Employee who does actual work

You don’t talk directly to the database. You open a **Session** from the **SessionFactory**, and that Session does all the database operations (save, get, update, delete).

<img width="2816" height="1536" alt="Session Facuty Characteristics" src="https://github.com/user-attachments/assets/10752940-7f6e-42d8-b1fc-3874c81c52eb" />


### Key Characteristics of SessionFactory

| Feature                  | Description                                                                 |
|-------------------------|-----------------------------------------------------------------------------|
| Created Once            | Built only when application starts                                          |
| Thread-Safe             | Multiple threads can safely use it at the same time                         |
| Immutable               | Once created, you cannot change it                                          |
| Heavy Object            | It is expensive to create (loads all mappings, configurations, dialects)   |
| One per Database        | Usually **one SessionFactory per database**                                 |
| Long-lived              | Lives until the application shuts down                                      |

<img width="2816" height="1536" alt="Charestreistics" src="https://github.com/user-attachments/assets/6128ec65-5910-4b0b-a74b-bd8b9296f1b9" />

### What Does SessionFactory Contain?

When Hibernate builds the SessionFactory, it loads and stores a lot of information inside it:

1. **All Mapping Information**
   - Which Java class is mapped to which database table?
   - Which property maps to which column?
   - All relationships (One-to-Many, Many-to-Many, etc.)

2. **Database Configuration**
   - Connection URL, username, password
   - Dialect (MySQL, PostgreSQL, Oracle, etc.)
   - Connection pool settings

3. **Hibernate Settings**
   - Caching configurations (Second Level Cache)
   - Query cache settings
   - Batch size, fetch strategies, etc.

4. **Metadata**
   - All entity metadata processed from annotations or XML mapping files

Because it contains so much information, creating a SessionFactory is **slow and expensive**. That’s why we create it only once.

### How to Create SessionFactory?

There are two main ways:

#### 1. Using Configuration (Traditional way - Hibernate 4 and below)

```java
Configuration configuration = new Configuration();
configuration.configure("hibernate.cfg.xml");   // or configure() without argument

SessionFactory sessionFactory = configuration.buildSessionFactory();
```

#### 2. Using StandardServiceRegistry (Modern way - Hibernate 5 & 6)

```java
StandardServiceRegistry registry = new StandardServiceRegistryBuilder()
    .configure()                    // reads hibernate.cfg.xml
    .build();

SessionFactory sessionFactory = new MetadataSources(registry)
    .buildMetadata()
    .buildSessionFactory();
```

In **Spring Boot**, you don’t need to write this code. Spring Boot automatically creates and manages the SessionFactory (or EntityManagerFactory) for you.

### Important Methods of SessionFactory

- `openSession()` → Opens a new Session (most commonly used)
- `getCurrentSession()` → Gets Session bound to current thread (used with Spring)
- `getSessionFactoryOptions()` → Returns configuration options
- `close()` → Closes the SessionFactory (usually done when application shuts down)

### Session vs SessionFactory (Quick Comparison)

| Feature               | SessionFactory                          | Session                                |
|-----------------------|-----------------------------------------|----------------------------------------|
| Created               | Once at application startup             | Many times (per request/operation)     |
| Thread Safety         | Thread-safe                             | Not thread-safe                        |
| Lifetime              | Long-lived (until app shuts down)       | Short-lived (open → work → close)      |
| Purpose               | Creates Sessions                        | Performs actual database operations    |
| Memory Usage          | Heavy                                   | Lightweight                            |
| Methods               | `openSession()`                         | `save()`, `get()`, `update()`, `delete()` |

### Best Practices for SessionFactory

1. **Create only once** – Usually in a singleton or as a Spring Bean.
2. **Never create inside a method** – It is very costly.
3. **Close it properly** when the application shuts down.
4. In Spring Boot, let Spring manage it (no manual creation needed).
5. Use **one SessionFactory per database**. If your app connects to multiple databases, create multiple SessionFactories.

### Real-World Analogy

Imagine a **Car Manufacturing Company**:

- **SessionFactory** = The entire Factory (built once, very expensive to build)
- **Session** = One Assembly Line Worker (you can have many workers)
- The Factory knows everything: how to make cars, what parts to use, quality rules, etc.
- Workers (Sessions) come and go, but the Factory remains the same.

Just like that, SessionFactory knows everything about your database and mappings, while Session does the actual work of saving/fetching data.

---

