#SQL Performance Explained Notes#
>**Author**
>Tom Reznick
----------

[TOC]

##Anatomy of an Index##

* Index is built using the `create index` statement.
* Requires its own disk space
* Holds a copy of indexed table data
* Undergoes constant change on `insert`, `update`, and `delete` statements
* needs to combine a doubly-linked list and a search tree

###Index Leaf Nodes###

  * Can't store data sequentially, b/c `index`
    * b/c we're on disk, we can't just use a memory pointer
    * If we needed to keep the sequence in order, we'd need to move a lot of disk blocks
    * We need a logical order
  * Doubly Linked List
    * Each node refers to the preceding and following nodes
    * Each node is a DB block or Page
    * Can read preceding and following node
    * Therefore can read forwards and backwards
    * Changing two pointers allows for fast insertion
    * Each block stores many sequential records pointing to DB blocks
###B-tree###
  * allows for fast traversal of the tree, since it's always balanced and is often very flat
* Slow indexes
  * leaf node chan: always needs to look in the records of a leaf node because index may be 1:n
  * Data may be scattered, and so we may need to look across a large disk
  * In Oracle:
    1)
      ```SQL
        INDEX UNIQUE SCAN
      ```
      does tree traversal to the leaf node
   2)
      ```SQL
        INDEX RANGE SCAN 
      ``` 
      does tree traversal and follows the leaf node chain, this can return a lot of records
   3)
      ```SQL
        TABLE ACCESS BY INDEX ROWID
      ```
      retrieves the rows of the table from the index range scan, this can bottleneck if the preceding results return many rows

##The Where Clause##

  * Different operators in a `WHERE` clause affect index usage

###The Equality Operator###

  * Most trivial and frequently used SQL operator

  * Primary Keys
    * Schema
    ```SQL
    CREATE TABLE employees (
      employee_id   NUMBER          NOT NULL,
      first_name    VARCHAR2(1000)  NOT NULL,
      last_name     VARCHAR2(1000)  NOT NULL,
      date_of_birth DATE            NOT NULL,
      phone_number  VARCHAR2(1000)  NOT NULL,
      CONSTRAINT employees_pk PRIMARY KEY (employee_id)
    );
    ```
    * DB automatically indexes on PK
    * QUERY:
    ```SQL
      SELECT first_name, last_name
        FROM employees,
        WHERE employee_id = 123
    ```
    * EXECUTION PLAN:
    ```
    ----------------------------------------------------------------
    |ID| Operation                    | Name         | Rows | Cost |
    ----------------------------------------------------------------
    | 0| SELECT STATEMENT             |              |   1  |  2   |
    | 1|  TABLE ACCESS BY INDEX ROWID | EMPLOYEES    |   1  |  2   |
    |*2|    INDEX UNIQUE SCAN         | EMPLOYEES_PK |   1  |  1   |
    ---------------------------------------------------------------
   
    Predicate Information (identified by operation id):
    ---------------------------------------------------
      2 - access("EMPLOYEE_ID"=123)
    ```
    * uses index scan unique
    
