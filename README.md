# Database Design
This is a mini lecture designed to help students as they begin personal projects. For most students this is the first time they have actually tried to build thier own schema. The main goals of this mini lecture are as follows:

1. To help students identify and avoid complicating pitfalls. 
2. To help students distinguish between "like" data and related data.
3. To demonstrate best practices.
4. To debunk common misconceptions.
5. To help students build more robust databases.

## Resources
* [lucidchart.com](https://lucidchart.com/) - You can create a free account that lets you build all kinds of charts including database diagrams.

## Instructions
Below is an example of a poorly designed database, a better design, and some information including questions and queries about both patterns. 

Walk through the poor table design and demonstrate how it could be queried. Point out the difficulties that come from the poor design to help students really grasp how it will affect the dataflow of their applications.

Work with students to create the better schema. Talk about how the the better design solves the problems created by the poor design. Demonstrate some queries to solidify this more scalable and maintainable pattern.

### Bad Example 
  students_classes_teachers

| student_id | student_first | student_last | student_gender | class1 | class2  | class3  | teacher1 | teacher2   | teacher3... |
| ---------- | ------------- | ------------ | -------------- | ------ | ------- | ------- | -------- | ---------- | ----------- |
| 1          | Nelly         | Narwol       | female         | Math   | English | Science | Mr. Jay  | Mrs. Thomp | Miss Lindy  |
| 2          | Greg          | Greggory     | male           | Music  | Math    | Science | Mrs. Net | Mr. Krimly | Miss Lindy  |

#### Problems Created by the Poor Design
  1. What do you name your table when it holds everything?
  2. How can you query a specific teacher/class?
  3. How could you query all the students that are in a certain class?
  4. Where do you store other class information?
  5. Creates duplicate data that is hard to differentiate when querying.
  6. If we wanted to search for a specific teacher or class the code gets quite wet, and the results have duplicates which are difficult to make sense of. 
  7. This table does not adequately represent the relationships between classes and teachers, merely that they are related to a particular student. It could be assumed that teacher1 corresoponds with class1, but this is still very difficult to search for.

  #### Some Possible Queries

  Find a specific teacher.
  ```
  select * 
  from students_classes_teachers
  where 
    teacher1 = $1 or 
    teacher2 = $1 or
    teacher3 = $1 or
    teacher4 = $1 or
    teacher5 = $1 or
    teacher6 = $1 or
    teacher7 = $1 or
    teacher8 = $1;
  ```
  Find a specific class.
  ```
  select * 
  from students_classes_teachers
  where 
    class1 = $1 or
    class2 = $1 or
    class3 = $1 or
    class4 = $1 or
    class5 = $1 or
    class6 = $1 or
    class7 = $1 or
    class8 = $1;
  ```
### Better Design

#### Strategies
1. "All classes in separate tables" >> this means separating "like" data in their own tables. It's better to have many smaller tables that can be queried through their relationships rather than have all "unlike", but related, data in the same table.
2. Try not to alter tables in production. This may seem relatively harmless, but it is dangerous. You could inadvertantly delete data trying to refactor your database.
3. Don't dynamically create/drop tables; for example we shouldn't be creating a new 'cart' table for each user of our app.

> Future employers may use a different naming convention, but for consistency use _snake\_case_.

> *Schema* is the blueprint of our database; it includes not only the data types of our fields/columns, but also the structure of data relationships.

> "All classes in separate tables." This means separating "like" data in their own tables. It's better to have many smaller tables that can be queried through their relationships rather than have all "unlike", but related, data in the same table.

> Use junction tables to track data relationships.

  students

  | id  | first_name | last_name | gender |
  | --- | ---------- | --------- | ------ |
  | 1   | Nelly      | Narwol    | female |
  | 2   | Greg       | Greggory  | male   |

  classes

  | id | subject | description       | 
  | -- | ------- | ----------------- |
  | 1  | Math    | Numbers and stuff |
  | 2  | English | Lots of reading   |

  teachers

  | id | first_name | last_name |
  | -- | ---------- | --------- |
  | 1  | Kenneth    | Jay       |
  | 2  | Linda      | Lindy     |

  students_classes

  | id | student_id | class_id |
  | -- | ---------- | -------- |
  | 1  | 1          | 1        |
  | 2  | 1          | 2        |

  teachers_classes

  | id | teacher_id | class_id |
  | -- | ---------- | -------- |
  | 1  | 1          | 1        |
  | 2  | 1          | 2        |

#### Solutions of the Better Design
  1. Table names describe the "class" of data it contains.
  2. We can query classes/teachers for a specific class or teacher.
  3. Using a join we can query students and classes to find all students attending a certain class.
  4. We can store more detailed information about classes and teachers and students.
  5. Avoids duplicates in our query results.
  6. Scalable and easy to maintain.
  7. The relationships between tables are available in junction tables.

#### Some Possible Queries

Find information of a specific teacher.
```
select * from teachers
where first_name = $1 and last_name = $2;
```
Find students attending a specific class.
```
select * from classes c
join students_classes sc on sc.class_id = c.id
join students s on s.id = sc.student_id
where c.id = $1;
```
Find teachers teaching a specific class.
```
select * from classes c
join teachers_classes tc on tc.class_id = c.id
join teachers t on t.id = sc.teacher_id
where c.id = $1;
``` 
You can join as many tables as you need.

## Concluding Recap
#### Naming
1. When labeling, create table names that are unique and describe the "class" of information. Create column names that make sense, they don't have to be unique from other tables. You can also use the nouns/adjectives rule; make table names a noun and column names adjectives to the table name.   
2. I like to label my primary key "table_name_id", and corresponding foreign keys the same. (The ultimate rule is consistency and to follow the convention set by your company.)
#### One to Many
One student enrolled in many classes.
1. "Classes" of data will be kept in the same table and related data will be referenced by a foreign key.
  > It is good to know that foreign keys can be constrained, however this has more to do with postgreSQL syntax and is outside the scope of this lesson as well as the scope of most personal projects.)
#### Many to Many
Many students enrolled in many classes.
1. Create junction tables to track the relationships between other tables.
2. Always think about how you will access the data!


#### Scalable and Maintainable
  From here on out if you want to add a new feature to your app (for example assignments in our student tracker) you can simply create a new table. Assignments are a many to many relationship, many assignments for many students, so we will need a table to keep the assignment information and then a junction table to track the relationship between each assignment and the student it corresoponds to. The beauty is we don't need to alter any of our existing tables to track the new information which is why this pattern of database design is easy to maintain!
## Contributions
  If you see any problems or have great ideas that can be added please let us know!