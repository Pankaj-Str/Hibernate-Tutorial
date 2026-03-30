# Create a Basic Hibernate Project


(Using **Maven + Pure Hibernate 6 + MySQL + Annotations** – No Spring Boot)

---

### Step 1: Prerequisites (Must Have Installed)

1. **JDK 17 or 21** (recommended 17)  
2. **MySQL Server** installed and running (XAMPP / MySQL Workbench / MySQL Installer)  
3. **Maven** installed (comes with IntelliJ/Eclipse)  
4. **IDE**: IntelliJ IDEA Community (recommended) or Eclipse  

**Make sure MySQL is running** and you know your root password.

---

### Step 2: Create a New Maven Project

**In IntelliJ IDEA:**
1. Open IntelliJ → **File** → **New** → **Project**
2. Select **Maven** → **Create from archetype** → **maven-archetype-quickstart**
3. Click **Next**
4. Fill details:
   - **GroupId**: `com.example`
   - **ArtifactId**: `hibernate-basic`
   - **Version**: `1.0-SNAPSHOT`
5. Click **Next** → **Finish**

Wait for Maven to import all files.

**Project structure** should look like this:
```
hibernate-basic
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── example
│   │   │           └── App.java
│   │   └── resources
│   └── test
├── pom.xml
└── ...
```

---

### Step 3: Update `pom.xml` (Add Dependencies)

Replace the entire content of **`pom.xml`** with this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>hibernate-basic</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- Hibernate Core (Latest stable as of 2026) -->
        <dependency>
            <groupId>org.hibernate.orm</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>6.6.45.Final</version>
        </dependency>

        <!-- MySQL Connector -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>9.6.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.13.0</version>
            </plugin>
        </plugins>
    </build>
</project>
```

→ Right-click on project → **Maven** → **Reload project** (or click the refresh icon in Maven tab).

---

### Step 4: Create Database in MySQL

1. Open MySQL Workbench or command line.
2. Run these commands:

```sql
CREATE DATABASE hibernate_demo;
USE hibernate_demo;
```

(We will let Hibernate create the table automatically using `hbm2ddl.auto=update`)

---

### Step 5: Create `hibernate.cfg.xml`

1. Right-click on **`src/main/resources`** → **New** → **File**
2. Name: `hibernate.cfg.xml`
3. Paste this exact content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>

        <!-- Database Connection Settings -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/hibernate_demo?useSSL=false&amp;serverTimezone=UTC</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">your_password_here</property>   <!-- ← CHANGE THIS -->

        <!-- Hibernate Settings -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
        <property name="hibernate.hbm2ddl.auto">update</property>

        <!-- Mapping Entity Classes -->
        <mapping class="com.example.Student"/>

    </session-factory>
</hibernate-configuration>
```

**Important**: Change `your_password_here` to your actual MySQL root password.

---

### Step 6: Create Entity Class (`Student.java`)

1. Right-click on **`src/main/java/com/example`** → **New** → **Java Class**
2. Name: `Student`
3. Paste this code:

```java
package com.example;

import jakarta.persistence.*;

@Entity
@Table(name = "student")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Long id;

    @Column(name = "name", length = 100)
    private String name;

    @Column(name = "age")
    private int age;

    // No-argument constructor (required by Hibernate)
    public Student() {}

    // Constructor with fields
    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

---

### Step 7: Create Hibernate Utility Class (`HibernateUtil.java`)

1. Right-click on **`src/main/java/com/example`** → **New** → **Java Class**
2. Name: `HibernateUtil`
3. Paste this code:

```java
package com.example;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {

    private static final SessionFactory sessionFactory;

    static {
        try {
            // Create Configuration and load hibernate.cfg.xml
            Configuration configuration = new Configuration();
            configuration.configure("hibernate.cfg.xml");   // loads the config file

            // Build SessionFactory (this is the heavy object created only once)
            sessionFactory = configuration.buildSessionFactory();

            System.out.println("✅ SessionFactory created successfully!");

        } catch (Exception e) {
            System.err.println("❌ SessionFactory creation failed!");
            throw new ExceptionInInitializerError(e);
        }
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }

    public static void shutdown() {
        if (sessionFactory != null) {
            sessionFactory.close();
            System.out.println("SessionFactory closed.");
        }
    }
}
```

---

### Step 8: Create Main Class to Test (Update `App.java`)

Replace the entire content of **`src/main/java/com/example/App.java`** with:

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.Transaction;

public class App {
    public static void main(String[] args) {

        System.out.println("🚀 Starting Basic Hibernate Project...");

        // Get Session from SessionFactory
        try (Session session = HibernateUtil.getSessionFactory().openSession()) {

            Transaction transaction = null;

            try {
                // Begin transaction
                transaction = session.beginTransaction();

                // Create a Student object
                Student student = new Student("Rahul Sharma", 22);

                // Save to database
                session.persist(student);

                // Commit transaction
                transaction.commit();

                System.out.println("✅ Student saved successfully!");
                System.out.println("Student Details: " + student);

            } catch (Exception e) {
                if (transaction != null) {
                    transaction.rollback();
                }
                e.printStackTrace();
            }

        } finally {
            // Close SessionFactory when done
            HibernateUtil.shutdown();
        }

        System.out.println("🎉 Project executed successfully!");
    }
}
```

---

### Step 9: Run the Project

1. Right-click on **`App.java`** → **Run 'App.main()'**
2. You should see output like:

```
🚀 Starting Basic Hibernate Project...
Hibernate: create table if not exists student ...
✅ SessionFactory created successfully!
Hibernate: insert into student (name,age) values (?,?)
✅ Student saved successfully!
Student Details: Student{id=1, name='Rahul Sharma', age=22}
SessionFactory closed.
🎉 Project executed successfully!
```

---

### Step 10: Verify in Database

Open MySQL Workbench → Run:

```sql
USE hibernate_demo;
SELECT * FROM student;
```

You will see the record inserted.

---

