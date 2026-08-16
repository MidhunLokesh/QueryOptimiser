# 📁 DBMS Project  
**SQL Query Parsing and Optimization**


## 🧩 Problem Statement

Design a system capable of processing SQL-like queries by:
- Performing **lexical** and **syntactic analysis** using **Flex** and **Bison**.
- Generating **relational algebra trees** to represent query execution plans.
- Applying **query optimization techniques** inspired by *Silberschatz’s DBMS textbook*.
- Estimating the **execution cost** based on tuples and operations.

---

## 🎯 Goals

1. **Lexical and Syntax Analysis**  
   - Tokenize and parse SQL queries using **Flex** and **Bison**.
   - Transform into structured intermediate forms (Relational Algebra Trees).

2. **Generate Computation Graphs**  
   - Construct and display the logical execution plan as a relational algebra tree.

3. **Query Optimization**  
   - Apply standard techniques:
     - Selection Pushdown
     - Projection Pushdown
     - Join Reordering

4. **Cost Estimation & Plan Selection**  
   - Estimate costs based on:
     - Number of tuples
     - Types of operations
     - Operator depth  
   - Select the **least-cost** tree as the optimal plan.

---

## ⚙️ How It Works

### 🧾 Parsing
- **Flex** scans SQL tokens (keywords, identifiers, operators, literals).
- **Bison** applies grammar rules to structure queries and build the **relational algebra tree**.

### 🌳 Relational Algebra Tree
- Built as binary trees where each node represents a relational operator.
- Internal nodes: Join, Select, Project, etc.
- Leaf nodes: Base tables.

### 📈 Optimization
- Multiple transformation passes generate **equivalent trees** using:
  - **Selection Pushdown**
  - **Projection Pushdown**
  - **Join Reordering**  
- A limit is set to prevent excessive trees: `MAX_RES_TREES = 25`.

### 💰 Cost Estimation
- Cost heuristics applied:
  - Joins are costlier.
  - Projection cost = 0 (unless `DISTINCT` is specified).
  - Writes (Insert/Update/Delete) = 2 × Block Transfer Cost.
- Tree with lowest cost is selected.

---

## 🗃️ Metadata

Read from `documentation.txt` during startup.

**Used for**:
- Validating table and column names
- Optimization decisions (e.g., join reordering, pushdowns)
- Schema-aware transformations

### 📄 Sample Tables:
```text
1. instructor ( ID , name , dept_name , salary )
2. teaches ( ID , course_id , sec_id , semester , year )
3. course ( course_id , title , dept_name , credits )
```
---

### ⚙️ Sample Query Optimization:
```text
Original Relational Algebra Tree for sql statement 1:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      Comparison: =
        Attribute: i.ID
        Attribute: w.ID
    ⋈-join
      Condition:
        Comparison: =
          Attribute: t.course_id
          Attribute: c.course_id
      ⋈-join
        Condition:
          Comparison: =
            Attribute: i.ID
            Attribute: t.ID
        AS i
          Relation: instructor
        AS t
          Relation: teaches
      AS c
        Relation: course
    AS w
      Relation: workers

Cost of actual tree: 24764
---------------------------------------------------------------------------------------------------

Tree after Pre-Transformations:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      Comparison: =
        Attribute: i.ID
        Attribute: w.ID
    ⋈-join
      Condition:
        Comparison: =
          Attribute: t.course_id
          Attribute: c.course_id
      ⋈-join
        Condition:
          Comparison: =
            Attribute: i.ID
            Attribute: t.ID
        AS i
          Relation: instructor
        AS t
          Relation: teaches
      AS c
        Relation: course
    AS w
      Relation: workers
---------------------------------------------------------------------------------------------------

Tree after Transformations:

Transformed Tree 1:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      Comparison: =
        Attribute: i.ID
        Attribute: w.ID
    π-Project ()
      Attribute: c.credits
        Attribute: i.ID
      ⋈-join
        Condition:
          Comparison: =
            Attribute: t.course_id
            Attribute: c.course_id
        π-Project ()
          Attribute: i.ID
            Attribute: t.course_id
          ⋈-join
            Condition:
              Comparison: =
                Attribute: i.ID
                Attribute: t.ID
            π-Project ()
              Attribute: i.ID
              AS i
                Relation: instructor
            π-Project ()
              Attribute: t.course_id
                Attribute: t.ID
              AS t
                Relation: teaches
        π-Project ()
          Attribute: c.credits
            Attribute: c.course_id
          AS c
            Relation: course
    π-Project ()
      Attribute: w.ID
      AS w
        Relation: workers

Cost of transformed tree 1: 15767116852

Transformed Tree 2:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      And: AND
        Comparison: =
          Attribute: i.ID
          Attribute: w.ID
        Comparison: =
          Attribute: t.course_id
          Attribute: c.course_id
    π-Project ()
      Attribute: c.credits
        Attribute: w.ID
          Attribute: c.course_id
      ⋈-join
        π-Project ()
          Attribute: w.ID
          AS w
            Relation: workers
        π-Project ()
          Attribute: c.credits
            Attribute: c.course_id
          AS c
            Relation: course
    π-Project ()
      Attribute: i.ID
        Attribute: t.course_id
      ⋈-join
        Condition:
          Comparison: =
            Attribute: i.ID
            Attribute: t.ID
        π-Project ()
          Attribute: i.ID
          AS i
            Relation: instructor
        π-Project ()
          Attribute: t.course_id
            Attribute: t.ID
          AS t
            Relation: teaches

Cost of transformed tree 2: 677564624

Transformed Tree 3:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      Comparison: =
        Attribute: t.course_id
        Attribute: c.course_id
    π-Project ()
      Attribute: i.ID
        Attribute: t.course_id
      ⋈-join
        Condition:
          Comparison: =
            Attribute: i.ID
            Attribute: w.ID
        π-Project ()
          Attribute: i.ID
            Attribute: t.course_id
          ⋈-join
            Condition:
              Comparison: =
                Attribute: i.ID
                Attribute: t.ID
            π-Project ()
              Attribute: i.ID
              AS i
                Relation: instructor
            π-Project ()
              Attribute: t.course_id
                Attribute: t.ID
              AS t
                Relation: teaches
        π-Project ()
          Attribute: w.ID
          AS w
            Relation: workers
    π-Project ()
      Attribute: c.credits
        Attribute: c.course_id
      AS c
        Relation: course

Cost of transformed tree 3: 6380766

Transformed Tree 4:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      And: AND
        Comparison: =
          Attribute: i.ID
          Attribute: w.ID
        Comparison: =
          Attribute: i.ID
          Attribute: t.ID
    π-Project ()
      Attribute: i.ID
      AS i
        Relation: instructor
    π-Project ()
      Attribute: c.credits
        Attribute: w.ID
          Attribute: t.ID
      ⋈-join
        Condition:
          Comparison: =
            Attribute: t.course_id
            Attribute: c.course_id
        π-Project ()
          Attribute: c.credits
            Attribute: w.ID
              Attribute: c.course_id
          ⋈-join
            π-Project ()
              Attribute: w.ID
              AS w
                Relation: workers
            π-Project ()
              Attribute: c.credits
                Attribute: c.course_id
              AS c
                Relation: course
        π-Project ()
          Attribute: t.ID
            Attribute: t.course_id
          AS t
            Relation: teaches

Cost of transformed tree 4: 1493216618

Transformed Tree 5:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      And: AND
        Comparison: =
          Attribute: t.course_id
          Attribute: c.course_id
        Comparison: =
          Attribute: i.ID
          Attribute: t.ID
    π-Project ()
      Attribute: t.course_id
        Attribute: t.ID
      AS t
        Relation: teaches
    π-Project ()
      Attribute: c.credits
        Attribute: i.ID
          Attribute: c.course_id
      ⋈-join
        Condition:
          Comparison: =
            Attribute: i.ID
            Attribute: w.ID
        π-Project ()
          Attribute: i.ID
          AS i
            Relation: instructor
        π-Project ()
          Attribute: c.credits
            Attribute: c.course_id
              Attribute: w.ID
          ⋈-join
            π-Project ()
              Attribute: w.ID
              AS w
                Relation: workers
            π-Project ()
              Attribute: c.credits
                Attribute: c.course_id
              AS c
                Relation: course

Cost of transformed tree 5: 1459170766

Transformed Tree 6:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      Comparison: =
        Attribute: t.course_id
        Attribute: c.course_id
    π-Project ()
      Attribute: i.ID
        Attribute: t.course_id
      ⋈-join
        Condition:
          And: AND
            Comparison: =
              Attribute: i.ID
              Attribute: t.ID
            Comparison: =
              Attribute: i.ID
              Attribute: w.ID
        π-Project ()
          Attribute: t.course_id
            Attribute: t.ID
              Attribute: w.ID
          ⋈-join
            π-Project ()
              Attribute: w.ID
              AS w
                Relation: workers
            π-Project ()
              Attribute: t.course_id
                Attribute: t.ID
              AS t
                Relation: teaches
        π-Project ()
          Attribute: i.ID
          AS i
            Relation: instructor
    π-Project ()
      Attribute: c.credits
        Attribute: c.course_id
      AS c
        Relation: course

Cost of transformed tree 6: 29053810929

Transformed Tree 7:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      Comparison: =
        Attribute: t.course_id
        Attribute: c.course_id
    π-Project ()
      Attribute: i.ID
        Attribute: t.course_id
      ⋈-join
        Condition:
          Comparison: =
            Attribute: i.ID
            Attribute: t.ID
        π-Project ()
          Attribute: i.ID
          ⋈-join
            Condition:
              Comparison: =
                Attribute: i.ID
                Attribute: w.ID
            π-Project ()
              Attribute: i.ID
              AS i
                Relation: instructor
            π-Project ()
              Attribute: w.ID
              AS w
                Relation: workers
        π-Project ()
          Attribute: t.course_id
            Attribute: t.ID
          AS t
            Relation: teaches
    π-Project ()
      Attribute: c.credits
        Attribute: c.course_id
      AS c
        Relation: course

Cost of transformed tree 7: 3228902

Transformed Tree 8:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      And: AND
        Comparison: =
          Attribute: i.ID
          Attribute: w.ID
        Comparison: =
          Attribute: i.ID
          Attribute: t.ID
    π-Project ()
      Attribute: i.ID
      AS i
        Relation: instructor
    π-Project ()
      Attribute: c.credits
        Attribute: w.ID
          Attribute: t.ID
      ⋈-join
        π-Project ()
          Attribute: c.credits
            Attribute: t.ID
          ⋈-join
            Condition:
              Comparison: =
                Attribute: t.course_id
                Attribute: c.course_id
            π-Project ()
              Attribute: t.ID
                Attribute: t.course_id
              AS t
                Relation: teaches
            π-Project ()
              Attribute: c.credits
                Attribute: c.course_id
              AS c
                Relation: course
        π-Project ()
          Attribute: w.ID
          AS w
            Relation: workers

Cost of transformed tree 8: 1467344234

Transformed Tree 9:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      And: AND
        Comparison: =
          Attribute: i.ID
          Attribute: w.ID
        Comparison: =
          Attribute: i.ID
          Attribute: t.ID
    π-Project ()
      Attribute: i.ID
      AS i
        Relation: instructor
    π-Project ()
      Attribute: c.credits
        Attribute: w.ID
          Attribute: t.ID
      ⋈-join
        Condition:
          Comparison: =
            Attribute: t.course_id
            Attribute: c.course_id
        π-Project ()
          Attribute: w.ID
            Attribute: t.ID
              Attribute: t.course_id
          ⋈-join
            π-Project ()
              Attribute: w.ID
              AS w
                Relation: workers
            π-Project ()
              Attribute: t.ID
                Attribute: t.course_id
              AS t
                Relation: teaches
        π-Project ()
          Attribute: c.credits
            Attribute: c.course_id
          AS c
            Relation: course

Cost of transformed tree 9: 1470782083

Transformed Tree 10:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      And: AND
        Comparison: =
          Attribute: t.course_id
          Attribute: c.course_id
        Comparison: =
          Attribute: i.ID
          Attribute: t.ID
    π-Project ()
      Attribute: t.course_id
        Attribute: t.ID
      AS t
        Relation: teaches
    π-Project ()
      Attribute: c.credits
        Attribute: i.ID
          Attribute: c.course_id
      ⋈-join
        Condition:
          Comparison: =
            Attribute: i.ID
            Attribute: w.ID
        π-Project ()
          Attribute: w.ID
          AS w
            Relation: workers
        π-Project ()
          Attribute: c.credits
            Attribute: i.ID
              Attribute: c.course_id
          ⋈-join
            π-Project ()
              Attribute: i.ID
              AS i
                Relation: instructor
            π-Project ()
              Attribute: c.credits
                Attribute: c.course_id
              AS c
                Relation: course

Cost of transformed tree 10: 1458985581

Transformed Tree 11:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      And: AND
        Comparison: =
          Attribute: t.course_id
          Attribute: c.course_id
        Comparison: =
          Attribute: i.ID
          Attribute: t.ID
    π-Project ()
      Attribute: t.course_id
        Attribute: t.ID
      AS t
        Relation: teaches
    π-Project ()
      Attribute: c.credits
        Attribute: i.ID
          Attribute: c.course_id
      ⋈-join
        π-Project ()
          Attribute: c.credits
            Attribute: c.course_id
          AS c
            Relation: course
        π-Project ()
          Attribute: i.ID
          ⋈-join
            Condition:
              Comparison: =
                Attribute: i.ID
                Attribute: w.ID
            π-Project ()
              Attribute: w.ID
              AS w
                Relation: workers
            π-Project ()
              Attribute: i.ID
              AS i
                Relation: instructor

Cost of transformed tree 11: 1458438178

Transformed Tree 12:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      Comparison: =
        Attribute: i.ID
        Attribute: w.ID
    ⋈-join
      Condition:
        Comparison: =
          Attribute: t.course_id
          Attribute: c.course_id
      ⋈-join
        Condition:
          Comparison: =
            Attribute: i.ID
            Attribute: t.ID
        AS i
          Relation: instructor
        AS t
          Relation: teaches
      AS c
        Relation: course
    AS w
      Relation: workers

Cost of transformed tree 12: 24764

Transformed Tree 13:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      Comparison: =
        Attribute: i.ID
        Attribute: w.ID
    π-Project ()
      Attribute: c.credits
        Attribute: i.ID
      ⋈-join
        Condition:
          Comparison: =
            Attribute: t.course_id
            Attribute: c.course_id
        ⋈-join
          Condition:
            Comparison: =
              Attribute: i.ID
              Attribute: t.ID
          AS i
            Relation: instructor
          AS t
            Relation: teaches
        AS c
          Relation: course
    π-Project ()
      Attribute: w.ID
      AS w
        Relation: workers

Cost of transformed tree 13: 14205

Transformed Tree 14:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      Comparison: =
        Attribute: i.ID
        Attribute: w.ID
    π-Project ()
      Attribute: c.credits
        Attribute: i.ID
      ⋈-join
        Condition:
          Comparison: =
            Attribute: t.course_id
            Attribute: c.course_id
        π-Project ()
          Attribute: i.ID
            Attribute: t.course_id
          ⋈-join
            Condition:
              Comparison: =
                Attribute: i.ID
                Attribute: t.ID
            AS i
              Relation: instructor
            AS t
              Relation: teaches
        π-Project ()
          Attribute: c.credits
            Attribute: c.course_id
          AS c
            Relation: course
    π-Project ()
      Attribute: w.ID
      AS w
        Relation: workers

Cost of transformed tree 14: 9013716

Transformed Tree 15:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      And: AND
        Comparison: =
          Attribute: i.ID
          Attribute: w.ID
        Comparison: =
          Attribute: t.course_id
          Attribute: c.course_id
    ⋈-join
      AS w
        Relation: workers
      AS c
        Relation: course
    ⋈-join
      Condition:
        Comparison: =
          Attribute: i.ID
          Attribute: t.ID
      AS i
        Relation: instructor
      AS t
        Relation: teaches

Cost of transformed tree 15: 97654571

Transformed Tree 16:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      And: AND
        Comparison: =
          Attribute: i.ID
          Attribute: w.ID
        Comparison: =
          Attribute: t.course_id
          Attribute: c.course_id
    π-Project ()
      Attribute: c.credits
        Attribute: w.ID
          Attribute: c.course_id
      ⋈-join
        AS w
          Relation: workers
        AS c
          Relation: course
    π-Project ()
      Attribute: i.ID
        Attribute: t.course_id
      ⋈-join
        Condition:
          Comparison: =
            Attribute: i.ID
            Attribute: t.ID
        AS i
          Relation: instructor
        AS t
          Relation: teaches

Cost of transformed tree 16: 34890001

Transformed Tree 17:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      And: AND
        Comparison: =
          Attribute: i.ID
          Attribute: w.ID
        Comparison: =
          Attribute: t.course_id
          Attribute: c.course_id
    π-Project ()
      Attribute: c.credits
        Attribute: w.ID
          Attribute: c.course_id
      ⋈-join
        π-Project ()
          Attribute: w.ID
          AS w
            Relation: workers
        π-Project ()
          Attribute: c.credits
            Attribute: c.course_id
          AS c
            Relation: course
    π-Project ()
      Attribute: i.ID
        Attribute: t.course_id
      ⋈-join
        Condition:
          Comparison: =
            Attribute: i.ID
            Attribute: t.ID
        AS i
          Relation: instructor
        AS t
          Relation: teaches

Cost of transformed tree 17: 34886100

Transformed Tree 18:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      And: AND
        Comparison: =
          Attribute: i.ID
          Attribute: w.ID
        Comparison: =
          Attribute: t.course_id
          Attribute: c.course_id
    ⋈-join
      ⋈-join
        Condition:
          Comparison: =
            Attribute: i.ID
            Attribute: t.ID
        AS i
          Relation: instructor
        AS t
          Relation: teaches
      AS w
        Relation: workers
    AS c
      Relation: course

Cost of transformed tree 18: 57708529

Transformed Tree 19:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      Comparison: =
        Attribute: t.course_id
        Attribute: c.course_id
    σ-Select[]
      Comparison: =
        Attribute: i.ID
        Attribute: w.ID
      ⋈-join
        ⋈-join
          Condition:
            Comparison: =
              Attribute: i.ID
              Attribute: t.ID
          AS i
            Relation: instructor
          AS t
            Relation: teaches
        AS w
          Relation: workers
    AS c
      Relation: course

Cost of transformed tree 19: 30172979

Transformed Tree 20:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      Comparison: =
        Attribute: t.course_id
        Attribute: c.course_id
    π-Project ()
      Attribute: i.ID
        Attribute: t.course_id
      σ-Select[]
        Comparison: =
          Attribute: i.ID
          Attribute: w.ID
        ⋈-join
          ⋈-join
            Condition:
              Comparison: =
                Attribute: i.ID
                Attribute: t.ID
            AS i
              Relation: instructor
            AS t
              Relation: teaches
          AS w
            Relation: workers
    π-Project ()
      Attribute: c.credits
        Attribute: c.course_id
      AS c
        Relation: course

Cost of transformed tree 20: 5473594

Transformed Tree 21:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      Comparison: =
        Attribute: t.course_id
        Attribute: c.course_id
    π-Project ()
      Attribute: i.ID
        Attribute: t.course_id
      ⋈-join
        Condition:
          Comparison: =
            Attribute: i.ID
            Attribute: w.ID
        ⋈-join
          Condition:
            Comparison: =
              Attribute: i.ID
              Attribute: t.ID
          AS i
            Relation: instructor
          AS t
            Relation: teaches
        AS w
          Relation: workers
    π-Project ()
      Attribute: c.credits
        Attribute: c.course_id
      AS c
        Relation: course

Cost of transformed tree 21: 14027

Transformed Tree 22:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      Comparison: =
        Attribute: t.course_id
        Attribute: c.course_id
    π-Project ()
      Attribute: i.ID
        Attribute: t.course_id
      ⋈-join
        Condition:
          Comparison: =
            Attribute: i.ID
            Attribute: w.ID
        π-Project ()
          Attribute: i.ID
            Attribute: t.course_id
          ⋈-join
            Condition:
              Comparison: =
                Attribute: i.ID
                Attribute: t.ID
            AS i
              Relation: instructor
            AS t
              Relation: teaches
        π-Project ()
          Attribute: w.ID
          AS w
            Relation: workers
    π-Project ()
      Attribute: c.credits
        Attribute: c.course_id
      AS c
        Relation: course

Cost of transformed tree 22: 7598

Transformed Tree 23:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      And: AND
        And: AND
          Comparison: =
            Attribute: i.ID
            Attribute: w.ID
          Comparison: =
            Attribute: t.course_id
            Attribute: c.course_id
        Comparison: =
          Attribute: i.ID
          Attribute: t.ID
    AS i
      Relation: instructor
    ⋈-join
      ⋈-join
        AS w
          Relation: workers
        AS c
          Relation: course
      AS t
        Relation: teaches

Cost of transformed tree 23: 13220702861

Transformed Tree 24:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      And: AND
        Comparison: =
          Attribute: i.ID
          Attribute: w.ID
        Comparison: =
          Attribute: i.ID
          Attribute: t.ID
    AS i
      Relation: instructor
    σ-Select[]
      Comparison: =
        Attribute: t.course_id
        Attribute: c.course_id
      ⋈-join
        ⋈-join
          AS w
            Relation: workers
          AS c
            Relation: course
        AS t
          Relation: teaches

Cost of transformed tree 24: 19783202901

Transformed Tree 25:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      And: AND
        Comparison: =
          Attribute: i.ID
          Attribute: w.ID
        Comparison: =
          Attribute: i.ID
          Attribute: t.ID
    π-Project ()
      Attribute: i.ID
      AS i
        Relation: instructor
    π-Project ()
      Attribute: c.credits
        Attribute: w.ID
          Attribute: t.ID
      σ-Select[]
        Comparison: =
          Attribute: t.course_id
          Attribute: c.course_id
        ⋈-join
          ⋈-join
            AS w
              Relation: workers
            AS c
              Relation: course
          AS t
            Relation: teaches

Cost of transformed tree 25: 14679033529
---------------------------------------------------------------------------------------------------

Optimized Tree:
π-Project ()
  Attribute: c.credits
    AS course_title
      Attribute: c.title
      AS worker_age
        Attribute: w.age
        AS instructor_salary
          Attribute: i.salary
          AS instructor_name
            Attribute: i.name
            Attribute: i.ID
  ⋈-join
    Condition:
      Comparison: =
        Attribute: t.course_id
        Attribute: c.course_id
    π-Project ()
      Attribute: i.ID
        Attribute: t.course_id
      ⋈-join
        Condition:
          Comparison: =
            Attribute: i.ID
            Attribute: w.ID
        π-Project ()
          Attribute: i.ID
            Attribute: t.course_id
          ⋈-join
            Condition:
              Comparison: =
                Attribute: i.ID
                Attribute: t.ID
            AS i
              Relation: instructor
            AS t
              Relation: teaches
        π-Project ()
          Attribute: w.ID
          AS w
            Relation: workers
    π-Project ()
      Attribute: c.credits
        Attribute: c.course_id
      AS c
        Relation: course

Cost of Optimized tree 22: 7598
```
