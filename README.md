# DBMSLab

## Exercise 1

### SQL

#### Create Employee table

```sql
SQL> CREATE TABLE EMPLOYEE(
  2  	SSN NUMBER PRIMARY KEY,
  3  	NAME VARCHAR(25),
  4  	SALARY INTEGER,
  5  	DNO NUMBER 
  6  );
```

#### Create Project table

```sql
SQL> CREATE TABLE PROJECT(
  2  	PNO NUMBER PRIMARY KEY,
  3  	PNAME VARCHAR(25)
  4  );
```

#### Create Works_On table

```sql
SQL> CREATE TABLE WORKS_ON(
  2  	SSN NUMBER,
  3  	PNO NUMBER,
  4  	HOURS NUMBER,
  5  	PRIMARY KEY(SSN, PNO),
  6  	FOREIGN KEY(SSN) REFERENCES EMPLOYEE(SSN) ON DELETE CASCADE,
  7  	FOREIGN KEY(PNO) REFERENCES PROJECT(PNO) ON DELETE CASCADE
  8  );
```

#### Obtain SSN of Employees assigned to Database Projects

```sql
SQL> SELECT SSN 
  2  FROM WORKS_ON
  3  WHERE PNO IN (
  4  	SELECT PNO 
  5  	FROM PROJECT
  6  	WHERE UPPER(PNAME) = 'DATABASE'
  7  );
```

#### Find the number of employees working in each department

```sql
SQL> SELECT DNO AS DEPARTMENT_NO, COUNT(DNO) AS NO_OF_EMPLOYEES
  2  FROM EMPLOYEE
  3  GROUP BY DNO
  4  ORDER BY DNO;
```

#### Update the project details of employee with SSN=1 to PNO=3

```sql
SQL> UPDATE WORKS_ON
  2  SET PNO = 3
  3  WHERE SSN = 1;
```

### MongoDB
```js
> use company

> db.createCollection('Employee')

> db.Employee.insertMany([
    {
      'SSN':1, 
      'Name': 'Alice', 
      'DNO':1, 
      'DName':'Admin', 
      'PNO': 1
    }, 
    {
      'SSN':2, 
      'Name':'Bob', 
      'DNO':1, 
      'DName':'Admin', 
      'PNO': 2
    }, 
    {
      'SSN':3, 
      'Name':'Chris', 
      'DNO':2, 
      'DName':'Research', 
      'PNO':2
    }
]);
```

#### List all the employees of Department named 'Admin'

```js
> db.Employee.find({"DName":'Admin'}, {'SSN':1, 'Name':1, '_id':0}).pretty()
```

#### Name the employees working on Project Number: 2

```js
> db.Employee.find({'PNO':2}, {'SSN':1, 'Name':1, '_id':0}).pretty()
```

### PL/SQL

#### Write a program that gives all employees in Department 1 a 15% pay increase. Display a message displaying how many employees were awarded the increase.

```sql
BEGIN
  UPDATE EMPLOYEE
  SET SALARY = SALARY*1.15
  WHERE DNO=1;
  
  DBMS_OUTPUT.PUT_LINE(SQL%ROWCOUNT || ' EMPLOYEE(S) WERE AWARDED THE INCREASE');
END;
/
```


## Exercise 2

### SQL

#### Create Part table

```sql
SQL> CREATE TABLE PART(
  2  	PID INTEGER PRIMARY KEY,
  3  	PNAME VARCHAR(25),
  4  	COLOR VARCHAR(25)
  5  );
```

#### Create Supplier table

```sql
SQL> CREATE TABLE SUPPLIER(
  2  	SID INTEGER PRIMARY KEY,
  3  	SNAME VARCHAR(25)
  4  );
```

#### Create Supply table

```sql
SQL> CREATE TABLE SUPPLY(                    
  2  	SID INTEGER REFERENCES SUPPLIER(SID) ON DELETE CASCADE,
  3  	PID INTEGER REFERENCES PART(PID) ON DELETE CASCADE,
  4  	QUANTITY INTEGER,
  5  	PRIMARY KEY(SID, PID)
  6  );
```
  
#### Obtain the part identifiers of parts supplied by supplier 'Tejas'

```sql
SQL> SELECT PART.PID, PART.PNAME, PART.COLOR
  2  FROM PART, SUPPLY, SUPPLIER
  3  WHERE PART.PID = SUPPLY.PID AND 
  4  SUPPLY.SID = SUPPLIER.SID AND
  5  UPPER(SNAME) = 'TEJAS';
```
  
#### Obtain the Names of suppliers who supply 'Noodles'

```sql
SQL> SELECT SNAME FROM SUPPLIER WHERE SID IN (
  2  	SELECT SID FROM SUPPLY WHERE PID IN (
  3  	  SELECT PID FROM PART WHERE UPPER(PNAME) = 'NOODLES'
  4  	)
  5  );
```

#### Delete the parts which are in brown color.\

```sql
SQL> DELETE PART WHERE COLOR='BROWN';
```

### MongoDB

```js
> use warehousedb

> db.createCollection('warehouse');

> db.warehouse.insert({'PID': 1, 'PNAME': 'WOOD', 'SID': 2, 'SNAME': 'TANU'})

> db.warehouse.insert({'PID': 1, 'PNAME': 'WOOD', 'SID': 3, 'SNAME': 'PINKY'})

> db.warehouse.insert({'PID': 2, 'PNAME': 'NOODLES', 'SID': 3, 'SNAME': 'PINKY'})
```

#### Update the parts identifier

```js
> db.warehouse.update({'PID':2}, {$set: {'PID':3}}, {$multi:true});

> db.warehouse.find({}, {'PID':1, '_id':0}).pretty()
```

#### Display all suppliers who supply the part with part identifier: PID = 1

```js
> db.warehouse.find({'PID':1}, {'SID':1, 'SNAME':1, '_id':0}).pretty()
```

### PL/SQL

#### WriteaPL/SQL program to display the contents of the above tables and then update the quantity of parts shipped by 5%

```sql
DECLARE
	CURSOR C1 IS SELECT * FROM PART;
	CURSOR C2 IS SELECT * FROM SUPPLIER;
	CURSOR C3 IS SELECT * FROM SUPPLY;

	PART_RECORD C1%ROWTYPE;
	SUPPLIER_RECORD C2%ROWTYPE;
	SUPPLY_RECORD C3%ROWTYPE;

BEGIN
	OPEN C1;
	OPEN C2;
	OPEN C3;
	
	DBMS_OUTPUT.PUT_LINE(‘PART TABLE:’);
	DBMS_OUTPUT.PUT_LINE(‘PID   PNAME     COLOR’);
	LOOP
		FETCH C1 INTO PART_RECORD;
		EXIT WHEN C1%NOTFOUND;
		DBMS_OUTPUT.PUT_LINE(PART_RECORD.PID || ‘ ‘ || PART_RECORD.PNAME ||
		‘ ‘ || PART_RECORD.COLOR’);
	END LOOP;


	DBMS_OUTPUT.PUT_LINE(‘SUPPLIER TABLE:’);
	DBMS_OUTPUT.PUT_LINE(‘SID   SNAME’);
	LOOP
		FETCH C2 INTO SUPPLIER_RECORD;
		EXIT WHEN C2%NOTFOUND;
		DBMS_OUTPUT.PUT_LINE(SUPPLIER_RECORD.SID || ‘ ' || SUPPLIER_RECORD.SNAME’);
	END LOOP;

	DBMS_OUTPUT.PUT_LINE(‘SUPPLY TABLE:’);
	DBMS_OUTPUT.PUT_LINE(‘SID   PID     QUANTITY’);
	LOOP
		FETCH C3 INTO SUPPLY_RECORD;
		EXIT WHEN C3%NOTFOUND;
		DBMS_OUTPUT.PUT_LINE(SUPPLY_RECORD.SID || ‘ ‘ || SUPPLY_RECORD.PID || 
		‘ ‘ || SUPPLY_RECORD.QUANTITY’);
	END LOOP;

	UPDATE SUPPLY
	SET QUANTITY = QUANTITY*1.05;

	DBMS_OUTPUT.PUT_LINE(SQL%ROWCOUNT || ‘ ROW(S) UPDATED’);
END;
/
```
