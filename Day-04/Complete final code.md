# Full working project

### With all mapping types we covered:

- Basic Entity Mapping  
- @Column attributes  
- @Embeddable (Address)  
- One-to-One (Student ↔ Laptop)  
- One-to-Many / Many-to-One (Department ↔ Students)  
- Many-to-Many (Student ↔ Course)  

### 1. `pom.xml` (Already correct from previous step – no change needed)

### 2. `hibernate.cfg.xml` (src/main/resources)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>

        <!-- Database Connection -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/hibernate_demo?useSSL=false&amp;serverTimezone=UTC</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">your_password_here</property>   <!-- ← CHANGE THIS -->

        <!-- Hibernate Settings -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
        <property name="hibernate.hbm2ddl.auto">update</property>

        <!-- All Entity Mappings -->
        <mapping class="com.example.Student"/>
        <mapping class="com.example.Laptop"/>
        <mapping class="com.example.Department"/>
        <mapping class="com.example.Course"/>
        <mapping class="com.example.Address"/>   <!-- Not needed for @Embeddable, but harmless -->

    </session-factory>
</hibernate-configuration>
```

### 3. `Address.java` (Embeddable)

```java
package com.example;

import jakarta.persistence.Embeddable;

@Embeddable
public class Address {

    private String city;
    private String state;
    private String pincode;

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

### 4. `Department.java` (One-to-Many side)

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

    public Department() {}

    public Department(String name) {
        this.name = name;
    }

    public void addStudent(Student student) {
        students.add(student);
        student.setDepartment(this);
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public List<Student> getStudents() { return students; }
    public void setStudents(List<Student> students) { this.students = students; }
}
```

### 5. `Laptop.java` (One-to-One)

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
    @JoinColumn(name = "student_id")
    private Student student;

    public Laptop() {}

    public Laptop(String brand, String model) {
        this.brand = brand;
        this.model = model;
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getBrand() { return brand; }
    public void setBrand(String brand) { this.brand = brand; }

    public String getModel() { return model; }
    public void setModel(String model) { this.model = model; }

    public Student getStudent() { return student; }
    public void setStudent(Student student) { this.student = student; }
}
```

### 6. `Course.java` (Many-to-Many owning side)

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
        name = "student_course",
        joinColumns = @JoinColumn(name = "course_id"),
        inverseJoinColumns = @JoinColumn(name = "student_id")
    )
    private Set<Student> students = new HashSet<>();

    public Course() {}

    public Course(String name) {
        this.name = name;
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public Set<Student> getStudents() { return students; }
    public void setStudents(Set<Student> students) { this.students = students; }
}
```

### 7. `Student.java` (Main Entity with all relationships)

```java
package com.example;

import jakarta.persistence.*;
import java.time.LocalDateTime;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "student")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "full_name", length = 150, nullable = false)
    private String name;

    private int age;

    @Column(unique = true)
    private String email;

    @Column(name = "created_at")
    private LocalDateTime createdAt;

    @Embedded
    private Address address;

    @OneToOne(mappedBy = "student", cascade = CascadeType.ALL)
    private Laptop laptop;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;

    @ManyToMany(mappedBy = "students")
    private Set<Course> courses = new HashSet<>();

    public Student() {}

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }

    public Address getAddress() { return address; }
    public void setAddress(Address address) { this.address = address; }

    public Laptop getLaptop() { return laptop; }
    public void setLaptop(Laptop laptop) { this.laptop = laptop; }

    public Department getDepartment() { return department; }
    public void setDepartment(Department department) { this.department = department; }

    public Set<Course> getCourses() { return courses; }
    public void setCourses(Set<Course> courses) { this.courses = courses; }
}
```

### 8. `HibernateUtil.java`

```java
package com.example;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {

    private static final SessionFactory sessionFactory;

    static {
        try {
            Configuration configuration = new Configuration();
            configuration.configure("hibernate.cfg.xml");
            sessionFactory = configuration.buildSessionFactory();
            System.out.println("✅ SessionFactory created successfully!");
        } catch (Exception e) {
            System.err.println("❌ Failed to create SessionFactory!");
            throw new ExceptionInInitializerError(e);
        }
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }

    public static void shutdown() {
        if (sessionFactory != null && !sessionFactory.isClosed()) {
            sessionFactory.close();
        }
    }
}
```

### 9. `App.java` (Main Class – Demo)

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.Transaction;
import java.time.LocalDateTime;

public class App {
    public static void main(String[] args) {

        System.out.println("🚀 Hibernate Mapping Demo Started...");

        try (Session session = HibernateUtil.getSessionFactory().openSession()) {

            Transaction tx = null;

            try {
                tx = session.beginTransaction();

                // Create Department
                Department csDept = new Department("Computer Science");

                // Create Student
                Student student = new Student();
                student.setName("Amit Kumar");
                student.setAge(21);
                student.setEmail("amit.kumar@gmail.com");
                student.setCreatedAt(LocalDateTime.now());
                student.setAddress(new Address("Mumbai", "Maharashtra", "400001"));

                // Create Laptop
                Laptop laptop = new Laptop("Dell", "XPS 15");
                laptop.setStudent(student);
                student.setLaptop(laptop);

                // Add Student to Department
                csDept.addStudent(student);

                // Create Courses
                Course javaCourse = new Course("Java Programming");
                Course dbCourse = new Course("Database Systems");

                // Many-to-Many
                student.getCourses().add(javaCourse);
                student.getCourses().add(dbCourse);
                javaCourse.getStudents().add(student);
                dbCourse.getStudents().add(student);

                // Save everything (cascade will handle most)
                session.persist(csDept);
                session.persist(javaCourse);
                session.persist(dbCourse);

                tx.commit();

                System.out.println("✅ All data saved successfully!");
                System.out.println("Student ID: " + student.getId());

            } catch (Exception e) {
                if (tx != null) tx.rollback();
                e.printStackTrace();
            }

        } finally {
            HibernateUtil.shutdown();
            System.out.println("🎉 Demo Completed!");
        }
    }
}
```

---

**How to Run:**

1. Update your MySQL password in `hibernate.cfg.xml`
2. Right-click on `App.java` → **Run 'App.main()'**
3. Check your `hibernate_demo` database in MySQL. You will see all tables created with proper relationships.

**Tables that will be created:**
- `student`
- `laptop`
- `department`
- `course`
- `student_course` (join table)

