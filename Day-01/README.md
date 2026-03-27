

### Hibernate Setup Step-by-Step

Okay brother, I'll explain the **step-by-step setup** for **Hibernate 6** (latest version ~6.6.x).

Nowadays, there are two main ways to set up Hibernate:

1. **Pure Hibernate** (Core only) – Good for basic learning  
2. **Spring Boot + Spring Data JPA** (Recommended for real projects) – This is what most people use today

I’ll explain both, but first I’ll show the **Spring Boot way** (modern & easiest), then the pure Hibernate method.

### 1. Spring Boot + Hibernate (Best & Most Recommended Way)

#### Step 1: Create the Project (Using Spring Initializr)
- Go to: https://start.spring.io
- Select **Maven** project
- Java version: 17 or 21
- Add these dependencies:
  - **Spring Web** (if you want to make REST APIs)
  - **Spring Data JPA**
  - Database driver (MySQL, PostgreSQL, or H2 for testing)
- Group: `com.example`, Artifact: `hibernate-demo`
- Click **Generate** → Download the ZIP → Extract it → Open in IntelliJ / Eclipse / VS Code

#### Step 2: Dependencies in pom.xml  
(The dependencies are added automatically, but here’s what you should have)

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- MySQL Driver -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>
    
    <!-- H2 Database for testing (in-memory) -->
    <!-- <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency> -->
</dependencies>
```

Hibernate 6 is automatically included when you add `spring-boot-starter-data-jpa`.

#### Step 3: Configure application.properties  
(src/main/resources folder)

**For MySQL:**

```properties
# Database Connection
spring.datasource.url=jdbc:mysql://localhost:3306/yourdb?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=yourpassword

# Hibernate Settings
spring.jpa.hibernate.ddl-auto=update     # Options: create, update, validate, none
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
```

**For H2 in-memory database (best for testing):**

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
```

#### Step 4: Create Entity Class (Student.java)

```java
package com.example.hibernatedemo.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private int age;
    private String city;

    // Constructors
    public Student() {}

    public Student(String name, int age, String city) {
        this.name = name;
        this.age = age;
        this.city = city;
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
    public String getCity() { return city; }
    public void setCity(String city) { this.city = city; }
}
```

#### Step 5: Create Repository (StudentRepository.java)

```java
package com.example.hibernatedemo.repository;

import com.example.hibernatedemo.entity.Student;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface StudentRepository extends JpaRepository<Student, Long> {
}
```

#### Step 6: Test the Setup

Add this in your main application class:

```java
@SpringBootApplication
public class HibernateDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(HibernateDemoApplication.class, args);
    }

    @Bean
    public CommandLineRunner testData(StudentRepository repo) {
        return args -> {
            Student s = new Student("Rahul", 22, "Mumbai");
            repo.save(s);
            System.out.println("Student saved successfully: " + repo.findAll());
        };
    }
}
```

Now run the application. Hibernate will automatically create the table and save the data!

---

### 2. Pure Hibernate Setup (Without Spring) – For Learning Only

If you want to learn core Hibernate without Spring:

#### pom.xml Dependencies

```xml
<dependencies>
    <!-- Hibernate Core -->
    <dependency>
        <groupId>org.hibernate.orm</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>6.6.45.Final</version>   <!-- Check latest version -->
    </dependency>

    <!-- MySQL Driver -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <version>9.1.0</version>
    </dependency>
</dependencies>
```

#### hibernate.cfg.xml (in src/main/resources)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>
        <!-- Database Connection -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/yourdb</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">yourpassword</property>

        <!-- Hibernate Properties -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
        <property name="hibernate.hbm2ddl.auto">update</property>

        <!-- Add your Entity Classes here -->
        <mapping class="com.example.Student"/>
    </session-factory>
</hibernate-configuration>
```

Then you build `SessionFactory` and use `Session` + `Transaction` manually.

---

**Pro Tip**:  
- For beginners and real projects → Use **Spring Boot + JPA** (less code, everything is automatic).  
- Learn **Pure Hibernate** only when you want to understand how things work internally.

---

