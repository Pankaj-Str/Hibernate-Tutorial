# Hibernate Mapping
**(Hibernate 6.6+ with Annotations – No XML Mapping)**

---

### What is Hibernate Mapping?

**Mapping** = Telling Hibernate **how to connect your Java classes (objects) to database tables**.

- Without mapping → Hibernate doesn’t know which class goes to which table, which field goes to which column.
- With mapping → You write Java classes with annotations, and Hibernate automatically creates/updates tables and converts objects ↔ database rows.

We use **JPA Annotations** (from `jakarta.persistence.*`) because they are standard and clean. Hibernate also supports extra annotations, but for beginners we stick to JPA ones.

---

### Step 0: Make Sure Your Project is Ready

1. Open your `hibernate-basic` project in IntelliJ.
2. Make sure MySQL is running and `hibernate_demo` database exists.
3. In `hibernate.cfg.xml`, change password if needed and keep `hbm2ddl.auto=update`.
4. Right-click project → Maven → Reload project (to make sure dependencies are there).

Now start the tutorial.

---

### Step 1: Basic Entity & Table Mapping (Already done, but let’s revise)

Your `Student.java` already has basic mapping.

**Key Annotations Explained:**

```java
@Entity                    // This class is a database entity (table)
@Table(name = "student")   // Optional: custom table name
public class Student {

    @Id                    // This is the Primary Key
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // Auto-increment in MySQL
    private Long id;

    @Column(name = "name", length = 100, nullable = false)  // Custom column
    private String name;

    // getters + setters + no-arg constructor + toString()
}
```

**Update `hibernate.cfg.xml`** (already has this line):
```xml
<mapping class="com.example.Student"/>
```

**Test it** (run `App.java` from previous tutorial).  
You should see `student` table created with columns `id`, `name`, `age`.

---

### Step 2: Primary Key Generation Strategies (Very Important)

There are 4 strategies. We will use **IDENTITY** for MySQL (best for beginners).

Open `Student.java` and make sure it has:
```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)  // Best for MySQL
private Long id;
```

**Other strategies (you can try later):**
- `GenerationType.SEQUENCE` → Uses database sequence (good for Oracle/PostgreSQL)
- `GenerationType.TABLE` → Uses a separate table for IDs (slow, old)
- `GenerationType.AUTO` → Hibernate decides (in Hibernate 6 it often uses SEQUENCE)

**Best Practice for MySQL**: Always use `IDENTITY`.

---

### Step 3: Column Mapping with @Column (All Attributes)

Update `Student.java` to see all useful `@Column` options:

```java
@Column(
    name = "full_name",           // column name in DB
    length = 150,                 // max length
    nullable = false,             // NOT NULL
    unique = false,               // unique constraint?
    columnDefinition = "VARCHAR(150) DEFAULT 'Unknown'"  // raw SQL definition
)
private String name;

@Column(name = "email", unique = true)
private String email;

@Column(name = "created_at")
private LocalDateTime createdAt;   // Add this field too
```

**Don’t forget to add import:**
```java
import java.time.LocalDateTime;
```

**Update getters/setters + constructor + toString() accordingly.**

**Add to `hibernate.cfg.xml`** (no change needed if already mapped).

**Test**: Run `App.java` again → New columns will be added automatically.

---

### Step 4: Component / Embedded Object Mapping (@Embeddable)

A Student has an **Address**. Address is not a separate entity, it is part of Student (Value Type).

**Step 4.1: Create Address.java**

Right-click `com.example` → New → Java Class → Name: `Address`

```java
package com.example;

import jakarta.persistence.Embeddable;

@Embeddable   // This class will be embedded inside another entity
public class Address {

    private String city;
    private String state;
    private String pincode;

    // No-arg constructor (mandatory)
    public Address() {}

    public Address(String city, String state, String pincode) {
        this.city = city;
        this.state = state;
        this.pincode = pincode;
    }

    // Getters and Setters
    public String getCity() { return city; }
    public void setCity(String city) { this.city = city; }
    public String getState() { return state; }
    public void setState(String state) { this.state = state; }
    public String getPincode() { return pincode; }
    public void setPincode(String pincode) { this.pincode = pincode; }
}
```

**Step 4.2: Update Student.java**

Add this field:
```java
@Embedded   // Embed the Address object
private Address address;
```

Add getter/setter + update constructors and toString().

**Step 4.3: Test it**

Update `App.java` main method (replace the student creation part):

```java
Student student = new Student();
student.setName("Rahul Sharma");
student.setAge(22);
student.setEmail("rahul@gmail.com");
student.setCreatedAt(LocalDateTime.now());
student.setAddress(new Address("Mumbai", "Maharashtra", "400001"));

session.persist(student);
```

Run `App.java` → You will see new columns in `student` table: `city`, `state`, `pincode` (no separate table for Address).

---

### Step 5: One-to-One Mapping (Student ↔ Laptop)

**Step 5.1: Create Laptop.java**

```java
package com.example;

import jakarta.persistence.*;

@Entity
@Table(name = "laptop")
public class Laptop {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String brand;
    private String model;

    @OneToOne
    @JoinColumn(name = "student_id")   // Foreign key in laptop table
    private Student student;

    // No-arg constructor, getters, setters, toString()
    public Laptop() {}
    // Constructor, getters, setters...
}
```

**Step 5.2: Update Student.java** (make bidirectional)

Add:
```java
@OneToOne(mappedBy = "student", cascade = CascadeType.ALL)  // mappedBy = owner side
private Laptop laptop;
```

**Step 5.3: Add mapping in hibernate.cfg.xml**

```xml
<mapping class="com.example.Laptop"/>
```

**Step 5.4: Test (Unidirectional + Bidirectional)**

In `App.java`:

```java
Student student = new Student();
// ... set other fields

Laptop laptop = new Laptop();
laptop.setBrand("Dell");
laptop.setModel("XPS 15");
laptop.setStudent(student);   // Owner side

student.setLaptop(laptop);

session.persist(student);   // cascade will save laptop too
```

Run → Two tables created, foreign key in `laptop` table.

---

### Step 6: One-to-Many & Many-to-One (Department ↔ Students)

**Step 6.1: Create Department.java**

```java
package com.example;

import jakarta.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "department")
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Student> students = new ArrayList<>();

    // constructors, getters, setters
    public void addStudent(Student student) {
        students.add(student);
        student.setDepartment(this);
    }
}
```

**Step 6.2: Update Student.java**

Add:
```java
@ManyToOne(fetch = FetchType.LAZY)   // Lazy is recommended
@JoinColumn(name = "department_id")
private Department department;
```

**Step 6.3: Add mapping in hibernate.cfg.xml**

```xml
<mapping class="com.example.Department"/>
```

**Step 6.4: Test**

In `App.java`:

```java
Department dept = new Department();
dept.setName("Computer Science");

Student s1 = new Student(); // set fields
Student s2 = new Student(); // set fields

dept.addStudent(s1);
dept.addStudent(s2);

session.persist(dept);
```

Run → `department` table + `department_id` column in `student` table.

---

### Step 7: Many-to-Many Mapping (Student ↔ Course)

**Step 7.1: Create Course.java**

```java
package com.example;

import jakarta.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "course")
public class Course {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToMany
    @JoinTable(
        name = "student_course",                  // join table name
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Student> students = new HashSet<>();

    // constructors, getters, setters
}
```

**Step 7.2: Update Student.java**

Add:
```java
@ManyToMany(mappedBy = "students")
private Set<Course> courses = new HashSet<>();
```

**Step 7.3: Add mapping in hibernate.cfg.xml**

```xml
<mapping class="com.example.Course"/>
```

**Step 7.4: Test**

In `App.java`:

```java
Course c1 = new Course();
c1.setName("Java Programming");

Course c2 = new Course();
c2.setName("Database Systems");

student.getCourses().add(c1);
student.getCourses().add(c2);

c1.getStudents().add(student);   // both sides
c2.getStudents().add(student);

session.persist(student);
session.persist(c1);
session.persist(c2);
```

Run → Three tables: `student`, `course`, `student_course` (join table).

---

### Step 8: Inheritance Mapping (Quick Overview)

Create three strategies (you can test one by one).

**Example: Person → Student & Teacher**

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)  // Change to JOINED or TABLE_PER_CLASS
@DiscriminatorColumn(name = "person_type")
public abstract class Person {
    @Id @GeneratedValue private Long id;
    private String name;
    // getters setters
}

@Entity
public class Student extends Person { ... }

@Entity
public class Teacher extends Person { ... }
```

Add mappings in cfg.xml and test.

---

### Final Step: Best Practices & Common Mistakes

1. Always have **no-arg constructor**.
2. Use `fetch = FetchType.LAZY` for collections (avoid N+1 problem).
3. Use `cascade = CascadeType.ALL` carefully.
4. Always add both sides in bidirectional relationships.
5. Run with `show_sql=true` to see generated SQL.
6. Never put business logic inside entities.


Run your `App.java` after each step and check tables in MySQL.

