# Pony-SQLite3_simplified
A simple and easy way to work with SQL without real knowledge of it, just basic understanding of Python and Pony-ORM.  
  
This code is acctually a fast implementation I did for an SQL test that I had no time to study or enough knowledge about it.  
<br />
Libraries used
```python
pony
pandas
random
os
sys
sqlite3
datetime
faker
```

<br />

An already created SQL code to generate Database if necessary
```python

squery = '''
CREATE TABLE Worker (
    WORKER_ID INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
    FIRST_NAME CHAR(25),
    LAST_NAME CHAR(25),
    SALARY INT(15),
    JOINING_DATE DATETIME,
    DEPARTMENT CHAR(25)
);

INSERT INTO Worker 
    (WORKER_ID, FIRST_NAME, LAST_NAME, SALARY, JOINING_DATE, DEPARTMENT) VALUES
        (001, 'Monika', 'Arora', 100000, '14-02-20 09.00.00', 'HR'),
        (002, 'Niharika', 'Verma', 80000, '14-06-11 09.00.00', 'Admin'),
        (003, 'Vishal', 'Singhal', 300000, '14-02-20 09.00.00', 'HR'),
        (004, 'Amitabh', 'Singh', 500000, '14-02-20 09.00.00', 'Admin'),
        (005, 'Vivek', 'Bhati', 500000, '14-06-11 09.00.00', 'Admin'),
        (006, 'Vipul', 'Diwan', 200000, '14-06-11 09.00.00', 'Account'),
        (007, 'Satish', 'Kumar', 75000, '14-01-20 09.00.00', 'Account'),
        (008, 'Geetika', 'Chauhan', 90000, '14-04-11 09.00.00', 'Admin');

CREATE TABLE Bonus (
    WORKER_REF_ID INT,
    BONUS_AMOUNT INT(10),
    BONUS_DATE DATETIME,
    FOREIGN KEY (WORKER_REF_ID)
        REFERENCES Worker(WORKER_ID)
        ON DELETE CASCADE
);

INSERT INTO Bonus 
    (WORKER_REF_ID, BONUS_AMOUNT, BONUS_DATE) VALUES
        (001, 5000, '16-02-20'),
        (002, 3000, '16-06-11'),
        (003, 4000, '16-02-20'),
        (001, 4500, '16-02-20'),
        (002, 3500, '16-06-11');
CREATE TABLE Title (
    WORKER_REF_ID INT,
    WORKER_TITLE CHAR(25),
    AFFECTED_FROM DATETIME,
    FOREIGN KEY (WORKER_REF_ID)
        REFERENCES Worker(WORKER_ID)
        ON DELETE CASCADE
);

INSERT INTO Title 
    (WORKER_REF_ID, WORKER_TITLE, AFFECTED_FROM) VALUES
 (001, 'Manager', '2016-02-20 00:00:00'),
 (002, 'Executive', '2016-06-11 00:00:00'),
 (008, 'Executive', '2016-06-11 00:00:00'),
 (005, 'Manager', '2016-06-11 00:00:00'),
 (004, 'Asst. Manager', '2016-06-11 00:00:00'),
 (007, 'Executive', '2016-06-11 00:00:00'),
 (006, 'Lead', '2016-06-11 00:00:00'),
 (003, 'Lead', '2016-06-11 00:00:00');
 '''
```

<br />

Main code
```python
db_file = 'test.db'
conn, curs = create_connection(db_file, rewrite=True) #rewrite is used to recreate db every time the main code runs, without issues.
## curs.close()
## conn.close()

db = Database()
#...............................................................ENTITIES...................................................................
class Worker(db.Entity):
#     _table_ = "enrollments"
    WORKER_ID = PrimaryKey(int)
    FIRST_NAME = Optional(str)
    LAST_NAME = Optional(str)
    SALARY = Optional(int)
    JOINING_DATE = Optional(datetime.datetime)
    DEPARTMENT = Optional(str)
    BONUSES = Set('Bonus')
    TITLES = Set('Title')
    
class Bonus(db.Entity):    
    WORKER_REF_ID = PrimaryKey(Worker)
    BONUS_AMOUNT = Optional(int)
    BONUS_DATE = Optional(datetime.datetime)   
        
class Title(db.Entity): 
    WORKER_REF_ID = PrimaryKey(Worker)
    WORKER_TITLE = Optional(str)
    AFFECTED_FROM = Optional(datetime.datetime)
#__________________________________________________________________________________________________________________________________________

# If SQL_queries_exist=True the above SQL code will be used
gen_fake_dat = execute_commands(squery, SQL_queries_exist=True)#<-<--<---<----<-----<------<---------------------------------------------<<

db.bind(provider='sqlite', filename=f"{os.path.abspath(os.getcwd())}/{db_file}", create_db=True)
db.generate_mapping(create_tables=True, check_tables=gen_fake_dat)

#If there is not an SQL code for the insertion of values in the db, then with SQL_queries_exist=False a specified number of random values will be added
if gen_fake_dat:
    fake_fillin(db.entities, 5)# 5 random values in each.
```
### Output for SQL_queries_exist=True sample  

    GET NEW CONNECTION
    RELEASE CONNECTION
    GET CONNECTION FROM THE LOCAL POOL
    PRAGMA foreign_keys = false
    BEGIN IMMEDIATE TRANSACTION
    CREATE INDEX "idx_bonus__worker_ref_id" ON "Bonus" ("WORKER_REF_ID")
    
    CREATE INDEX "idx_title__worker_ref_id" ON "Title" ("WORKER_REF_ID")
    
    COMMIT
    PRAGMA foreign_keys = true
    CLOSE CONNECTION
    RELEASE CONNECTION


<br />

Database representation
```python
ER_draw(db_file)
for ent in list(db.entities.keys()):
    sql_present(ent_name=str_to_class(ent).__name__, entity_present=True)
```


    
![png](/img/output_4_1.png)
    



<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Worker</th>
      <th>WORKER_ID</th>
      <th>FIRST_NAME</th>
      <th>LAST_NAME</th>
      <th>SALARY</th>
      <th>JOINING_DATE</th>
      <th>DEPARTMENT</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Monika</td>
      <td>Arora</td>
      <td>100000</td>
      <td>14-02-20 09.00.00</td>
      <td>HR</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Niharika</td>
      <td>Verma</td>
      <td>80000</td>
      <td>14-06-11 09.00.00</td>
      <td>Admin</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Vishal</td>
      <td>Singhal</td>
      <td>300000</td>
      <td>14-02-20 09.00.00</td>
      <td>HR</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Amitabh</td>
      <td>Singh</td>
      <td>500000</td>
      <td>14-02-20 09.00.00</td>
      <td>Admin</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Vivek</td>
      <td>Bhati</td>
      <td>500000</td>
      <td>14-06-11 09.00.00</td>
      <td>Admin</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>Vipul</td>
      <td>Diwan</td>
      <td>200000</td>
      <td>14-06-11 09.00.00</td>
      <td>Account</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>Satish</td>
      <td>Kumar</td>
      <td>75000</td>
      <td>14-01-20 09.00.00</td>
      <td>Account</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>Geetika</td>
      <td>Chauhan</td>
      <td>90000</td>
      <td>14-04-11 09.00.00</td>
      <td>Admin</td>
    </tr>
  </tbody>
</table>
</div>




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Bonus</th>
      <th>WORKER_REF_ID</th>
      <th>BONUS_AMOUNT</th>
      <th>BONUS_DATE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>5000</td>
      <td>16-02-20</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>3000</td>
      <td>16-06-11</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>4000</td>
      <td>16-02-20</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>4500</td>
      <td>16-02-20</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>3500</td>
      <td>16-06-11</td>
    </tr>
  </tbody>
</table>
</div>




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Title</th>
      <th>WORKER_REF_ID</th>
      <th>WORKER_TITLE</th>
      <th>AFFECTED_FROM</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Manager</td>
      <td>2016-02-20 00:00:00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Executive</td>
      <td>2016-06-11 00:00:00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8</td>
      <td>Executive</td>
      <td>2016-06-11 00:00:00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5</td>
      <td>Manager</td>
      <td>2016-06-11 00:00:00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>Asst. Manager</td>
      <td>2016-06-11 00:00:00</td>
    </tr>
    <tr>
      <th>5</th>
      <td>7</td>
      <td>Executive</td>
      <td>2016-06-11 00:00:00</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6</td>
      <td>Lead</td>
      <td>2016-06-11 00:00:00</td>
    </tr>
    <tr>
      <th>7</th>
      <td>3</td>
      <td>Lead</td>
      <td>2016-06-11 00:00:00</td>
    </tr>
  </tbody>
</table>
</div>


<br />

Examples  
Every example uses a Pony and a raw SQLite implementation
```python
select(raw_sql('substr(w.FIRST_NAME,1,3)') for w in Worker).without_distinct().show() #Pony
sql_present('Select substr(FIRST_NAME,1,3) from Worker;') # SQLite
```

    SELECT substr(w.FIRST_NAME,1,3)
    FROM "Worker" "w"
    
    raw_sql('substr(w.FIRST_NAME,1,3)')
    -----------------------------------
    Mon                                
    Nih                                
    Vis                                
    Ami                                
    Viv                                
    Vip                                
    Sat                                
    Gee                                




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>substr(FIRST_NAME,1,3)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Mon</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Nih</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Vis</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ami</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Viv</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Vip</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Sat</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Gee</td>
    </tr>
  </tbody>
</table>
</div>


<br />


```python
select((w.DEPARTMENT, sum(w.SALARY)) for w in Worker).show()
sql_present('SELECT DEPARTMENT, sum(Salary) from worker group by DEPARTMENT;')
```

    SELECT "w"."DEPARTMENT", coalesce(SUM("w"."SALARY"), 0)
    FROM "Worker" "w"
    GROUP BY "w"."DEPARTMENT"
    
    w.DEPARTMENT|sum(w.SALARY)
    ------------+-------------
    Account     |275000       
    Admin       |1170000      
    HR          |400000       




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>DEPARTMENT</th>
      <th>sum(Salary)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Account</td>
      <td>275000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Admin</td>
      <td>1170000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>HR</td>
      <td>400000</td>
    </tr>
  </tbody>
</table>
</div>


<br />

In this example we can also check out that using the outputted SQL code from the Pony execution inside sql_present weilds the same result
```python
select((w.FIRST_NAME, w.SALARY) for w in Worker if w.SALARY == max(w.SALARY for w in Worker)).without_distinct().show()

sql_present('''SELECT "w"."FIRST_NAME", "w"."SALARY"
FROM "Worker" "w"
WHERE "w"."SALARY" = (
    SELECT MAX("w-2"."SALARY")
    FROM "Worker" "w-2"
    )''')

sql_present('SELECT FIRST_NAME, SALARY from Worker WHERE SALARY=(SELECT max(SALARY) from Worker);')
```

    SELECT "w"."FIRST_NAME", "w"."SALARY"
    FROM "Worker" "w"
    WHERE "w"."SALARY" = (
        SELECT MAX("w-2"."SALARY")
        FROM "Worker" "w-2"
        )
    
    w.FIRST_NAME|w.SALARY
    ------------+--------
    Amitabh     |500000  
    Vivek       |500000  




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>FIRST_NAME</th>
      <th>SALARY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Amitabh</td>
      <td>500000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Vivek</td>
      <td>500000</td>
    </tr>
  </tbody>
</table>
</div>




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>FIRST_NAME</th>
      <th>SALARY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Amitabh</td>
      <td>500000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Vivek</td>
      <td>500000</td>
    </tr>
  </tbody>
</table>
</div>  

<br />

### Output for SQL_queries_exist=False sample
    GET NEW CONNECTION
    RELEASE CONNECTION
    GET CONNECTION FROM THE LOCAL POOL
    PRAGMA foreign_keys = false
    BEGIN IMMEDIATE TRANSACTION
    CREATE TABLE "Worker" (
      "WORKER_ID" INTEGER NOT NULL PRIMARY KEY,
      "FIRST_NAME" TEXT NOT NULL,
      "LAST_NAME" TEXT NOT NULL,
      "SALARY" INTEGER,
      "JOINING_DATE" DATETIME,
      "DEPARTMENT" TEXT NOT NULL
    )
    
    CREATE TABLE "Bonus" (
      "id" INTEGER PRIMARY KEY AUTOINCREMENT,
      "WORKER_REF_ID" INTEGER NOT NULL REFERENCES "Worker" ("WORKER_ID") ON DELETE CASCADE,
      "BONUS_AMOUNT" INTEGER,
      "BONUS_DATE" DATETIME
    )
    
    CREATE INDEX "idx_bonus__worker_ref_id" ON "Bonus" ("WORKER_REF_ID")
    
    CREATE TABLE "Title" (
      "id" INTEGER PRIMARY KEY AUTOINCREMENT,
      "WORKER_REF_ID" INTEGER NOT NULL REFERENCES "Worker" ("WORKER_ID") ON DELETE CASCADE,
      "WORKER_TITLE" TEXT NOT NULL,
      "AFFECTED_FROM" DATETIME
    )
    
    CREATE INDEX "idx_title__worker_ref_id" ON "Title" ("WORKER_REF_ID")
    
    SELECT "Bonus"."id", "Bonus"."WORKER_REF_ID", "Bonus"."BONUS_AMOUNT", "Bonus"."BONUS_DATE"
    FROM "Bonus" "Bonus"
    WHERE 0 = 1
    
    SELECT "Title"."id", "Title"."WORKER_REF_ID", "Title"."WORKER_TITLE", "Title"."AFFECTED_FROM"
    FROM "Title" "Title"
    WHERE 0 = 1
    
    SELECT "Worker"."WORKER_ID", "Worker"."FIRST_NAME", "Worker"."LAST_NAME", "Worker"."SALARY", "Worker"."JOINING_DATE", "Worker"."DEPARTMENT"
    FROM "Worker" "Worker"
    WHERE 0 = 1
    
    COMMIT
    PRAGMA foreign_keys = true
    CLOSE CONNECTION

<br />

Database representation
```python
ER_draw(db_file)
for ent in list(db.entities.keys()):
    sql_present(ent_name=str_to_class(ent).__name__, entity_present=True)
```


    
![png](/img/output_4_0.png)
    





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Worker</th>
      <th>WORKER_ID</th>
      <th>FIRST_NAME</th>
      <th>LAST_NAME</th>
      <th>SALARY</th>
      <th>JOINING_DATE</th>
      <th>DEPARTMENT</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>FIRST_NAME_1</td>
      <td>LAST_NAME_1</td>
      <td>2</td>
      <td>2020-07-15 02:15:23</td>
      <td>DEPARTMENT_1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>FIRST_NAME_2</td>
      <td>LAST_NAME_2</td>
      <td>2</td>
      <td>2020-08-31 06:01:18</td>
      <td>DEPARTMENT_2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>FIRST_NAME_3</td>
      <td>LAST_NAME_3</td>
      <td>-4</td>
      <td>2020-01-18 19:36:28</td>
      <td>DEPARTMENT_3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>FIRST_NAME_4</td>
      <td>LAST_NAME_4</td>
      <td>-4</td>
      <td>2020-02-09 13:09:19</td>
      <td>DEPARTMENT_4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>FIRST_NAME_5</td>
      <td>LAST_NAME_5</td>
      <td>-5</td>
      <td>2020-11-22 22:22:37</td>
      <td>DEPARTMENT_5</td>
    </tr>
  </tbody>
</table>
</div>






<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Bonus</th>
      <th>id</th>
      <th>WORKER_REF_ID</th>
      <th>BONUS_AMOUNT</th>
      <th>BONUS_DATE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>2020-10-26 01:04:42</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2</td>
      <td>1</td>
      <td>2020-07-15 21:22:23</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>3</td>
      <td>1</td>
      <td>2020-05-23 20:32:32</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2</td>
      <td>-1</td>
      <td>2020-01-04 01:25:17</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>4</td>
      <td>-4</td>
      <td>2020-11-15 08:05:55</td>
    </tr>
  </tbody>
</table>
</div>






<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Title</th>
      <th>id</th>
      <th>WORKER_REF_ID</th>
      <th>WORKER_TITLE</th>
      <th>AFFECTED_FROM</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>5</td>
      <td>WORKER_TITLE_1</td>
      <td>2020-04-21 02:22:52</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2</td>
      <td>WORKER_TITLE_2</td>
      <td>2020-08-16 02:56:11</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>3</td>
      <td>WORKER_TITLE_3</td>
      <td>2020-07-24 06:42:35</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>5</td>
      <td>WORKER_TITLE_4</td>
      <td>2020-08-31 07:08:27</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>3</td>
      <td>WORKER_TITLE_5</td>
      <td>2020-02-08 20:10:48</td>
    </tr>
  </tbody>
</table>
</div>


<br />

Examples
Every example uses a Pony and a raw SQLite implementation
```python
select(raw_sql('substr(w.FIRST_NAME,1,3)') for w in Worker).without_distinct().show() #Pony
sql_present('Select substr(FIRST_NAME,1,3) from Worker;') # SQLite
```

    GET NEW CONNECTION
    SWITCH TO AUTOCOMMIT MODE
    SELECT substr(w.FIRST_NAME,1,3)
    FROM "Worker" "w"
    
    raw_sql('substr(w.FIRST_NAME,1,3)')
    -----------------------------------
    FIR                                
    FIR                                
    FIR                                
    FIR                                
    FIR                                





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>substr(FIRST_NAME,1,3)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>FIR</td>
    </tr>
    <tr>
      <th>1</th>
      <td>FIR</td>
    </tr>
    <tr>
      <th>2</th>
      <td>FIR</td>
    </tr>
    <tr>
      <th>3</th>
      <td>FIR</td>
    </tr>
    <tr>
      <th>4</th>
      <td>FIR</td>
    </tr>
  </tbody>
</table>
</div>


<br />


```python
select((w.DEPARTMENT, sum(w.SALARY)) for w in Worker).show()
sql_present('SELECT DEPARTMENT, sum(Salary) from worker group by DEPARTMENT;')
```

    SELECT "w"."DEPARTMENT", coalesce(SUM("w"."SALARY"), 0)
    FROM "Worker" "w"
    GROUP BY "w"."DEPARTMENT"
    
    w.DEPARTMENT|sum(w.SALARY)
    ------------+-------------
    DEPARTMENT_1|2            
    DEPARTMENT_2|2            
    DEPARTMENT_3|-4           
    DEPARTMENT_4|-4           
    DEPARTMENT_5|-5           





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>DEPARTMENT</th>
      <th>sum(Salary)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>DEPARTMENT_1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>DEPARTMENT_2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>DEPARTMENT_3</td>
      <td>-4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>DEPARTMENT_4</td>
      <td>-4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>DEPARTMENT_5</td>
      <td>-5</td>
    </tr>
  </tbody>
</table>
</div>


<br />

In this example we can also check out that using the outputted SQL code from the Pony execution inside sql_present weilds the same result
```python
select((w.FIRST_NAME, w.SALARY) for w in Worker if w.SALARY == max(w.SALARY for w in Worker)).without_distinct().show()

sql_present('''SELECT "w"."FIRST_NAME", "w"."SALARY"
FROM "Worker" "w"
WHERE "w"."SALARY" = (
    SELECT MAX("w-2"."SALARY")
    FROM "Worker" "w-2"
    )''')

sql_present('SELECT FIRST_NAME, SALARY from Worker WHERE SALARY=(SELECT max(SALARY) from Worker);')
```

    SELECT "w"."FIRST_NAME", "w"."SALARY"
    FROM "Worker" "w"
    WHERE "w"."SALARY" = (
        SELECT MAX("w-2"."SALARY")
        FROM "Worker" "w-2"
        )
    
    w.FIRST_NAME|w.SALARY
    ------------+--------
    FIRST_NAME_1|2       
    FIRST_NAME_2|2       



<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>FIRST_NAME</th>
      <th>SALARY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>FIRST_NAME_1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>FIRST_NAME_2</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>FIRST_NAME</th>
      <th>SALARY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>FIRST_NAME_1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>FIRST_NAME_2</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>
