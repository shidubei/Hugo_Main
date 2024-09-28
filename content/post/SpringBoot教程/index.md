+++
title = 'SpringBoot教程'
date = 2024-09-26T23:25:11+08:00
draft = true
+++

# Spring Boot笔记

# 1.JPA

## 1.1 JPA简介

**JPA简单来说，是一种Java的ORM映射语法，通过使用JPA语法，可以将Java中的类和对象与数据库中的数据表，属性，值相对应，接下来对JPA中的部分语法进行总结和解释**

## 1.2 @Entity

**我们知道，在数据库设计中，我们会设计class diagram，也会涉及ERD(Entity Ralation Diagram)，这些都意味中数据库中的一项数据表可以视作一个类/实体来看待。**

**因此，在JPA中，通常使用@Entity注解来表示该类为一个实体，可能对应数据库中的一个数据表(通常在设计时，程序中的类名和数据库中的表名要一致)**

```java
@Entity
public class department{

}
```

**于此同时，在数据库中，对应的数据表设计为**

```sql
create table department
```

**需要注意的是，被@Entity注解的类一般和数据库中的表要同名，这样才方便JPA在解析时找到对应的类和数据表**

### 1.2.1 @Table

**如果类和数据表不方便同名，需要在编程中给出注解，指定该类对应的数据表名，编码如下**

```java
@Entity
@Table(name="dept")
public class department{}
```

```sql
//数据库中的表
create table dept
```

**通过@Entity注解，JPA能够将类和数据表一一对应**

## 1.3 @Id

**在数据库设计中，我们不仅仅会设计数据表名，还会设计到数据的属性，其中，我们需要设计某些属性作为数据表的主键。那么在Java程序中，我们需要用什么来标注一个主键呢？**

**没错，就是我们的@Id注解。【注意：@Id注解表示的是注解的属性为主键，该属性名字不一定非要为Id,课程中的Id可能会迷惑为只有Id属性才能被@Id注解，实际上@Id注解的是主键】**

**对应的Java编码大致如下：**

```java
@Entity
public class department{
  //@Id注解的是主键，即使主键名字可能不是Id,也要用@Id注解，如这里我们用部门名称作为主键
  @Id  
  private String name;
  ...
}
```

**数据库中的编码**

```sql
create table department(
    name varchar(50) not null,
    ...
    primary key(name)
);
```

### 1.3.1 @Column

**需要注意的是，在JPA设计时，类中的成员变量和数据表中列名不一定需要保持一致，在JPA解析时，JPA解析成员变量映射到同名的数据表中的列，如果不同名，可以使用@Column来指定映射列，如上诉的@Table一样**

**例如，在Java中的编码**

```java
@Entity
public class department{
    @Id
    @Column(name="departmentName")
    private String name;
    ...
}
```

**数据库中的编码**

```sql
create table department(
    departmentName varchar(50) not null,
    ...
    primary key(departmentName)
);
```

**可能你要问，数据库的主键除了单主键，还存在复合主键，这时候怎么用JPA中的注解来映射呢？**

**哈哈，这一点待笔者查询资料后在后面补充说明吧，目前的@Id记住用于指定主键即可**

### 1.3.2 @GeneratedValue

**我们在设计数据库时，如果不确定该怎么设计主键的话，可能要求数据库自动帮我们生成主键，如1，2，3...等，在MySQL中通常这样设计**

```sql
create table employee(
    //注意，MySQL中是这样写的，但其他数据库软件中可能存在其他写法
    id int(10) not null auto_increment,
    ...
)
```

**因此，在Java编程中，我们也需要告诉JPA该主键是自动生成的，这就需要用到@GenerateValue注解，该注解的作用是定义主键的生产策略，在Java中编码如下**

```java
@Entity
public class Employee{
    @Id
    //表示生产策略为自动生成自增字段（对应M），还有其他生成策略
    @GenereateValue(strategy=GenerationType.IDENTITY)
    private int id;
}
```

**其中生成策略还有以下几种：**

- **GenerationType.SEQUENCE:根据数据库的序列生成主键，通常和Oracle，PostgreSQL等相联系**

- **GenerationType.Auto:默认策略，根据底层数据库自动选择生成策略(IDENTITY或者SEQUENCE)**

## 1.4 OneToOne/OneToMany/ManyToOne/ManyToMany

**在数据库设计中，我们除了设计单独的表之外，还需要设计表与表之间的联系。**

**比如：一个国家只能拥有一个首都，这是一对一关系；一个用户可以拥有多个订单，这是一对多关系；一个学生可以选修多个课程，一个课程可以被多个学生选修，这是多对多关系。在数据库设计中，我们通常使用外键这一属性来帮助我们确定关系，而在Java编程中，JPA提供一系列的注解来帮助我们映射这些关系**

### 1.4.1 Owning Side/Inverse Side/mappedBy/ @JoinColumn

**为了帮助我们理解这些关系，我们先理解下Owning Side和Inverse Side(有点抽象，笔者只提供个人的理解作为参考)**

**我们知道，在数据库设计中，通常使用外键将两个表联系起来**

**如下：**

```sql
create table student(
    id int(10) not null,
    name varchar(50) not null,
    teamId int(10),
    primary key(id),
    foreign key(teamId) references team(id)
);
create table team(
    id int(10) not null,
    teamName varchar(10) not null,
    primary key(id)
); 
```

**而在Java程序中，在编码时，可能会出现这样的编码**

```java
public class Student{
    private int id;
    private String name;
    private Team team;
    ...
}

public class Team{
    private int id;
    private String teamName;
    private List<Student> students;
    ...
}
```

**不难发现，在设计java程序时，我们可能在Student类和Team类中都引用对方作为成员变量，也就是bidirectional entity relationship（双向实体关系），那么问题来了，作为程序员，我们是知道谁是外键，但JPA程序不知道，于是在映射外键时，就可能会出现错误。**

**因此，在设计编码时，我们需要让JPA明确谁是外键的拥有方,谁是外键的映射方**

**外键的拥有方即Ownning Side**

**外键的映射方即Inverse Side**

**在JPA中，我们使用mappedBy这一参数来映射外键关系**

**mapppedBy:从字面翻译来看，就是被谁映射，因此，我们通常在映射方使用mappedBy来指向外键的拥有方**

**以上面的Student和Team为例，Student表拥有外键，是Ownning Side,Team表则是Inverse Side**

```java
// Ownning Side
@Entity
public class Student{
    @Id
    private int id;
    private String name;
    @ManyToOne
    @JoinColumn(name="teamId")
    private Team team;
}


// Inverse Side
@Entity
public class Capital{
    @Id
    private int id;
    private String teamName;
    //利用mappedBy，明确外键关系由Student的team字段维护，mappedBy后面跟着的名字
    //和Ownning Side中的字段一致
    @OneToMany(mappedBy="team")
    private List<Student> students;
}
```

**以上则是关于mappedBy属性的使用以及OwnningSide和InverseSide的分析，有一点点绕，需要注意理解**

**<mark>【或者这样理解，mappedBy从翻译来看，是被动的，被xxx映射，因此mappedBy肯定出现在映射方，被拥有方映射】</mark>**

**此外，在编码时，你注意到这里我们用了一个@JoinColumn注解，该注解的作用是告诉JPA该字段作为外键。因为在JPA中，默认会自动生成一个外键列名（关联的实体名字加上_id后缀），这个生成的外键名字可能和我们实际数据库中的外键名字不一致，因此我们需要@JoinColumn来指定对应外键名称**

**如上述数据库设计中，我们设计外键名称为TeamId，与默认生成的外键名不一致，因此使用@JoinColumn(name="TeamId")来指定该字段和数据表中TeamId外键相映射**

**除此之外，@JoinColumn还有其他属性：**

- **name:定义外键列的名称，不指定时JPA会默认生成**

- **referencedColumnName:指定引用的实体类那个列作为外键目标，通常引用的是关联实体的主键，不指定时默认为关联实体的主键**

### 1.4.2 @OneToOne

**首先是一对一关系，在JPA中用@OneToOne来表示，在实际的编码中，可能出现在两个一对一的类互相引用时使用，如国家和首都的对应关系**

```java
@Entity
public class Country{
    @Id
    private int id;
    ...
    @OneToOne
    private Capital capital;
}


@Entity
public class Capital{
    @Id
    private int id;
    ...
    @OneToOne(mappedBy="capital")
    private Country country
}
```

**值得注意的是，一对一关系对外键的设计没有特别的要求，只是看方便查询与否，比如这里我们设计Country存储外键，也可以使用Capital存储外键**

### 1.4.3 @OneToMany&@ManyToOne

**@OneToMany通常和@ManyToOne一起出现，因为一对多通常也意味着多对一关系的存在**

**在一对多和多对一关系中，常常出现List或Set等集合表示多的关系，例如用户和订单**

```java
@Entity
public class User{
    @Id
    private int id;
    ...
    @OneToMany(mappedBy="user")
    private List<Order> orders;
}

@Entity
public class Order{
    @Id
    private int id;
    ...
    @ManyToOne
    private User user;
}
```

**注意，在一对多和多对一关系中，通常多的那方持有外键**

### 1.4.4 @ManyToMany/ @JoinTable

**讲解完一对一和一对多关系之后，我们需要注意到多对多关系，不同于一对一关系或者一对多关系，多对多关系通常需要一个中间表来存储外键。如班级和学生**

```sql
create table student(
    id int(10) not null,
    name varchar(50),
    primary key(id)  
);


create table course(
    id int(10) not null,
    name varchar(50),
    primary key(id)
);


create table student_course(
    student_id int(10) not null,
    course_id int(10) not null,
    primary key(student_id,course_id),
    foreign key(student_id) references student(id),
    foreign key(course_id) references student(id)
);
```

**不难看出，如果要我们设计java程序的话，外键理应存放在中间表中，而在JPA里面，通常使用JoinTable来指向中间表,Java编程如下**

```java
@Entity
public class Student{
    @Id
    private int id;
    private String name;

    @ManyToMany
    @JoinTable(name="student_course",
     joinColumns=@JoinColumn(name="student_id"),
     inverseJoinColumns=@JoinColumn(name="course_id"))
    private List<Course> courses;
}

@Entity
public class Course{
    @Id
    private int id;
    private String name;

    @ManyToMany(mappedBy="courses")
    private List<Course> courses;
}
```

**首先，因为多对多关系采取中间表来进行存储外键，因此在JPA外键映射中，任何一方都可以作为外键的持有方或者映射方，这里我们选取Student作为外键的持有方，Course作为外键的映射方**

**关键在于利用@JoinTable来映射中间表，这里给出上诉属性的解释**

* **name：指定中间表的名称，通常为类名_关联类名**

* **joinColumns:定义当前实体在中间表中的外键列，如student在中间表的外键列为student_id**

* **InverseJoinColumns:定义关联实体在中间表的外键列**

<mark>（有疑问：命名是否有规则）</mark>

**通过以上的代码注解，可以实现多对多关系的映射**

## 1.5 Inheritence的写法

**在数据库设计中,我们知道一个实体可能继承自另一个实体,这种继承关系也需要在JPA中被映射.而在数据库设计中,我们知道要将这种继承关系转化为表的关系有三种方式: Single Table,Class Table以及Concrete Table.**

**而在Java程序中,我们知道父类和子类,有时候父类并不提供具体的属性和方法,而是作为一个抽象类存在,如:**

```java
public abstract class Shape{
    private int id;
    private String color;
    
    public abstract double area();
}


public class Circle extends Shape{
    private double radius;

    @Override
    public double area(){
      return Math.PI*radius*radius;
}
}
```

**这时,我们在对应的数据库设计中需要设计一个表来对应Shape吗?Shape实体压根不存储具体的值啊,具体的值都被Shape的子类给实现,因此,在设计数据表时,Shape实体在数据库中不存在对应的表.**

**那么问题来了,我们该如何让JPA知道Shape实体不存在对应的表,但Shape的属性又被子类继承呢.于是引出@MappedSuperclass的注解.**

### 1.5.1 @MappedSuperclass

**首先,@MappedSuperclass是JPA中对于基类的注解,但本身并不会映射到数据库表里面,<mark>该注解的主要作用是允许子类继承父类中的被映射为数据库表字段的字段以及JPA注解,避免重复的代码</mark>**

**最为重要的,@MappedSuperclass并不会生成表,也就是说,我们在数据库设计时,并不需要专门设计一个表来对应父类,但@MappedSuperclass中的映射字段会被子类继承,<mark>还有一点需要注意的是,被@MappedSuperclass注解的类不能够通过JPQL来查询.</mark>**

**例子:**

```java
@MappedSuperclass
public abstract class Shape{
    @Id
    @GenerateValue(strategy=GenerationType.IDENTITY)
    private long id;
    private String color;
   
    public abstract double area();
}


@Entity
public class Circle extends Shape{
    private double radius;
    
    @Override
    public double area(){
      return Math.PI*radius*radius;
    }
}
```

**对应的数据库设计为:**

```sql
create table Circle(
    id int auto_increment,
    color varchar(50),
    radius double,
    primary key(id)
)
```

**不难发现,实际的数据表中并没有Shape表,但Circle表中却出现了Shape类中的属性.**

****

**虽然但是,有抽象类继承,也有真正的实体类继承呀,有可能父类和子类都需要在数据库中有对应的表,这种情况下,我们的JPA应该怎么去写呢,接下来介绍的注解即解决这个问题**

### 1.5.2 @Inheritance/@DiscriminatorColumn/@DiscriminatorValue

**我们知道数据库设计继承关系时,有单表继承的设计(Single Table),即将父类和子类的属性放在一个表里面,通过定义一个不同的Type来进行区分,比如:**

```sql
create table person(
    id int auto_increment,
    name varchar(50) not null,
    person_type varchar(25),
    cap double, // student的字段
    subject varchar(255) // teacher的字段
)
```

**对于单表继承,JPA中对应的写法如下:**

```java
@Entity
@Inheritance(strategy=InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name="person_type")
public abstract class Person{
    @Id
    @GenerateValue(strategy=GenerationType.IDENTITY)
    private int id;
    private String name;
}


@Entity
@DiscriminatorValue("student")
public class Student extends Person{
    private double cap;
}


@Entity
@DiscriminatorValue("teacher")
public class Teacher extends Person{
    private String subject;
}
```

**@Inheritance用于指定实体类继承关系的存储策略**

* **InheritanceType.SINGLE_TABLE:单表继承策略,即所有的父类和子类的数据存储在一个数据表当中**

* **InheritanceType.JOINED:类表继承,每个父类和子类都创建一个表,父类和子类的表通过外键连接**

* **InheritanceType.TABLE_PER_CLASS:确定表继承,即只为子类创建独立的表,而每个子类的表当中存储父类和子类的字段**



**@DiscriminatorColumn用于指定区分列,如同我们上面代码定义的"person_type",通过这个注解,我们知道继承的表中,区分列是谁**

**@DiscriminatorValue用于指定子类在区分列中的值,比如Student类在表中对应的值就是student,Teacher类在表中对应的值就是teacher,@DiscriminatorValue一般和@DiscriminatorColumn连用,表示该类在表的区分列中的对应值是什么.**

# 2.SpringBoot JPA

**这部分将介绍一些SpringBoot框架中使用JPA的代码编写**

## 2.1 Repository

**Repository接口是SpringBoot框架提供的管理数据库的接口，在SpringBoot中的定义如下：**

```java
interface Repository<Class,ID>{}
```

**其中，Class表示要操作的类，ID表示要操作类的ID的类型（Integer,String等等）**

**除此之外,SpringBoot里面还提供其它的Repository接口,列表如下**

| 名字                               | 作用                                            |
| -------------------------------- | --------------------------------------------- |
| Repository<T,ID>                 | 所有Spring Data Repository接口的顶级父接口,没有任何方法       |
| CrudRepository<T,ID>             | 继承自Repository接口,提供了增删改查(CRUD)的操作              |
| PagingAndSortingRepository<T,ID> | 继承了CrudRepository接口,除了增删改查之外,还提供分页和排序的功能      |
| JpaRepository                    | 继承了PagingAndSortingRepository接口,提供了JPA特定的操作方法 |

**以上是Spring Boot框架中常见的几种Repository库,方便使用**

**此外,在编码时,Spring Boot的JPA编码和JPA一致,需要注意的是结构问题**

**在编程时,我们一般区分实体类(实体层)和接口类(数据访问层),一般来说,一个实体对应一个数据访问接口,接口和实体放在不同的包,以workshop来看,项目中的结构如下:**

```cmd

src/main/java
├── sg.nus.iss.jpa.getstarted.workshop
│   └── SpringJpaGetstartedWorkshopApplication.java
├── sg.nus.iss.jpa.getstarted.workshop.model.domain
│   ├── Address.java
│   ├── Course.java
│   ├── Customer.java
│   ├── Department.java
│   └── Student.java
└── sg.nus.iss.jpa.getstarted.workshop.model.repository
    ├── AddressRepository.java
    ├── CourseRepository.java
    ├── CustomerRepository.java
    ├── DepartmentRepository.java
    └── StudentRepository.java
```

**可以看出,实体类存放在domainn包下,而所有的访问接口都存放在repository包下,之后深入了解SpringBoot后,会对项目的架构有更深入的了解**

## 2.2 Derived Query Methods(派生查询方法)

**Spring Data JPA所提供的一种允许通过解析方法名,自动生成查询语句而无需编写JPQL或者SQL语句的功能.**

**一般来说,派生查询方法的方法命名遵循以下格式:**

```textile
findBy+属性名+查询条件
```

**其中属性名和查询条件不能省略,下面给出几个例子**

| 查询条件             | 例子                                      |
| ---------------- | --------------------------------------- |
| And              | findByLastnameAndFIrstname              |
| Or               | findByLastnameOrFirstname               |
| Is,Equals        | findByFirstnameIs/findByFirstnameEquals |
| Between          | findByStartDateBetween                  |
| LessThan         | findByAgeLessThan                       |
| LessThanEqual    | findByAgeLessThanEqual                  |
| GreaterThan      | findByAgeGreaterThan                    |
| GreaterThanEqual | findByAgeGreaterThanEqual               |
| OrderBy          | findByOrderByLastnameDesc               |

## 2.3 Custom Queries

**除了Spring Data JPA提供的派生语法,我们也可以自己编写方法来实现查询**

### 2.3.1 @Query和@Param

**这个注解用于写SQL或者JPQL查询语句,表明在调用该方法时,执行的查询语句是什么,比如**

```java
@Query("select c from Course c")
public List<Course> findAllCourse();
```

**这样,我们调用findAllCourse方法时,就会执行对应的Query语句了**

**当然,我们还希望某些方法能够接受参数,改变查询语句的条件,这时候,我们就要用到@Param注解了,它提供映射参数的功能,能将我们传入的参数对应到Query语句中**

```java
@Query("select c from Course c where c.name= :name")
public List<Course> findCourseByName(@Param("name" String name);
```

**这样一来,我们传入的name参数,就通过@Param注解,替换了@Query语句中的:name,如我们传入Math,语句就会变为**

```jpql
select c from Course c where c.name = 'Math'
```

**以上则是Spring Boot中的JPA使用,其它的JPQL知识待笔者整理**
