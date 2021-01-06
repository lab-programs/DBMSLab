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
---
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
---
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
---

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
---
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
---
### PL/SQL

#### Write a PL/SQL program to display the contents of the above tables and then update the quantity of parts shipped by 5%

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
		DBMS_OUTPUT.PUT_LINE(SUPPLIER_RECORD.SID || ' ' || SUPPLIER_RECORD.SNAME’);
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
---
## Exercise 3

### SQL

#### Create Boat table

```
SQL> CREATE TABLE BOAT(
  2  	BID INTEGER PRIMARY KEY,
  3  	BNAME VARCHAR(25),
  4  	BCOLOR VARCHAR(25)
  5  );
```

#### Create Sailor table

```sql
SQL> CREATE TABLE SAILOR(
  2  	SID INTEGER PRIMARY KEY,
  3  	SNAME VARCHAR(25),  
  4   	SAGE NUMBER(2)
  5  );
```

#### Create Reserve table

```sql
SQL> CREATE TABLE RESERVE(
  2  	BID INTEGER,
  3  	SID INTEGER,
  4  	DAY VARCHAR(11),
  5  	PRIMARY KEY(BID, SID),
  6  	FOREIGN KEY(BID) REFERENCES BOAT(BID) ON DELETE CASCADE,
  7  	FOREIGN KEY(SID) REFERENCES SAILOR(SID) ON DELETE CASCADE
  8  );
```

#### Obtain the bid of the boats reserved by ‘BOB’

```sql
SQL> SELECT DISTINCT(BID)
  2  FROM RESERVE, SAILOR
  3  WHERE RESERVE.SID = SAILOR.SID AND SAILOR.SNAME = 'BOB';
```

#### Retrieve the bid of the boats reserved by all the sailors

```sql
SQL> SELECT BID FROM BOAT WHERE NOT EXISTS (
  2  	SELECT SAILOR.SID FROM SAILOR
  3  	MINUS
  4  	SELECT RESERVE.SID FROM RESERVE WHERE RESERVE.BID = BOAT.BID
  5  );
```

#### Find the number of boats reserved by each sailor

```sql
SQL> SELECT SAILOR.SID, SAILOR.SNAME, COUNT(SAILOR.SID) AS NO_OF_BOATS_RESERVED
  2  FROM RESERVE, SAILOR
  3  WHERE RESERVE.SID = SAILOR.SID
  4  GROUP BY SAILOR.SID, SAILOR.SNAME
  5  ORDER BY SAILOR.SID;
```
---
### MongoDB

```js
> use harbor

> db.createCollection('Boat');

> db.Boat.insert({'BID': 1, 'BNAME':'A', 'COLOR':'RED', 'SID':1, 'SNAME':'X'});

> db.Boat.insert({'BID': 1, 'BNAME':'A', 'COLOR':'RED', 'SID':2, 'SNAME':'Y'});

> db.Boat.insert({'BID': 2, 'BNAME':'B', 'COLOR':'WHITE', 'SID':2, 'SNAME':'Y'});
```

#### Obtain the number of boats obtained by sailor 'Y'

```js
> db.Boat.find({'SNAME':'Y'}).pretty().count()
```

#### Retrieve boats of color 'RED'

```js
> db.Boat.find({'COLOR':'RED'}, {'BID':1, 'BNAME':1, '_id':0}).pretty();
```
---
### PL/SQL

#### Write a PL/SQL program to check whether a given number is prime or not

```sql
DECLARE
	N NUMBER := &N;
	FLAG NUMBER := 1;
	I NUMBER;

BEGIN
	FOR I IN 2..N/2 LOOP
		IF MOD(N, I) = 0 THEN
			FLAG = 0
			EXIT
		END IF;
	END LOOP;
	IF FLAG = 0 THEN
		DBMS_OUTPUT.PUT_LINE(N || “ IS COMPOSITE”)
	ELSE
		DBMS_OUTPUT.PUT_LINE(N || “ IS PRIME”)
	END IF;
END; 
/
```
---
## Exercise 4

### SQL

#### Create Part table

```sql
SQL> CREATE TABLE PART(
  2  	PID INTEGER PRIMARY KEY,
  3  	PNAME VARCHAR(25),
  4  	COLOR VARCHAR(25)
  5  );
```

#### Create Warehouse table

```sql
SQL> CREATE TABLE WAREHOUSE (
  2  	WID INTEGER PRIMARY KEY,
  3     WNAME VARCHAR(25),
  4  	WADDRESS VARCHAR(50)
  5  );
```

#### Create Shipment table

```sql
SQL> CREATE TABLE SHIPMENT (
  2  	PID INTEGER,
  3  	WID INTEGER,
  4  	QUANTITY INTEGER,
  5  	SDATE VARCHAR(10),  
  6  	FOREIGN KEY(PID) REFERENCES PART(PID) ON DELETE CASCADE,
  7  	FOREIGN KEY(WID) REFERENCES WAREHOUSE(WID) ON DELETE CASCADE,
  8  	PRIMARY KEY(PID, WID)
  9  );
```

#### Obtain the Names of warehouses which have shipped red colored parts.

```sql
SQL> SELECT DISTINCT(WNAME)
  2  FROM WAREHOUSE, SHIPMENT
  3  WHERE WAREHOUSE.WID = SHIPMENT.WID
  4  AND SHIPMENT.PID IN (
  5  	  SELECT PID FROM PART WHERE COLOR='RED'
  6  );
```

#### Retrieve the PNO of the parts shipped by all the warehouses

```sql
SQL> SELECT PID
  2  FROM PART
  3  WHERE NOT EXISTS (
  4  	  SELECT WID FROM WAREHOUSE
  5  	  MINUS
  6  	  SELECT SHIPMENT.WID FROM SHIPMENT WHERE SHIPMENT.PID = PART.PID
  7  );
```

#### Find the number of parts supplied by each warehouse

```sql
SQL> SELECT WAREHOUSE.WID, WNAME, COUNT(WAREHOUSE.WID)
  2  FROM WAREHOUSE, SHIPMENT
  3  WHERE WAREHOUSE.WID = SHIPMENT.WID
  4  GROUP BY WAREHOUSE.WID, WNAME
  5  ORDER BY WAREHOUSE.WID;
```


#### List the warehouse details which ships maximum quantity of parts

```sql
SQL> SELECT WID
  2  FROM (
  3  	  SELECT WID, SUM(QUANTITY) AS TOTAL
  4  	  FROM SHIPMENT
  5  	  GROUP BY WID
  6  )
  7  WHERE TOTAL = (
  8  	  SELECT MAX(TOTAL)
  9  	  FROM (
 10  	  	  SELECT WID, SUM(QUANTITY) AS TOTAL
 11  	  	  FROM SHIPMENT
 12  	  	  GROUP BY WID
 13  	  )
 14  );
```
---
### MongoDB

```js
> use shipment

> db.createCollection('Warehouse');

> db.Warehouse.insert({'WID':1, 'WNAME':'WA', 'PID':1, 'PNAME':'WOOD', 'QUANTITY':10});

> db.Warehouse.insert({'WID':1, 'WNAME':'WA', 'PID':2, 'PNAME':'STEEL', 'QUANTITY':20});

> db.Warehouse.insert({'WID':2, 'WNAME':'WB', 'PID':2, 'PNAME':'STEEL', 'QUANTITY':15});
```

#### Find the parts shipped from warehouse :Wname”

```js
> db.Warehouse.find({'WNAME':'WA'}, {'_id':0, 'PID':1, 'PNAME':1}).pretty();
```

#### List the total quantity supplied from each warehouse

```js
> db.Warehouse.aggregate([
	{
		$group: {
			'_id': '$WID', 
			total: {
				$sum: '$QUANTITY'
			}
		}
	}
]);
```
---
### PL/SQL

#### Using cursors demonstrate the process of copying the contents of one table to a new table

```sql
DECLARE
	W_ID	WAREHOUSE.WID%TYPE;
	W_NAME  WAREHOUSE.WNAME%TYPE;
	W_ADDRESS  WAREHOUSE.WADDRESS%TYPE;

	CURSOR C IS SELECT * FROM WAREHOUSE;

BEGIN
	OPEN C;
	
	LOOP
		FETCH C INTO W_ID, W_NAME, W_ADDRESS;
		EXIT WHEN C%NOTFOUND;

		INSERT INTO NEW_WAREHOUSE(WID, WNAME, WADDRESS)
		VAUES(W_ID, W_NAME, W_ADDRESS);

	END LOOP;
	
	CLOSE C;
END;
/
```
---
## Exercise 5

### SQL

#### Create Book table

SQL> CREATE TABLE BOOK (
  2  	ISBN NUMBER(12) PRIMARY KEY,
  3  	BTITLE VARCHAR(25),
  4  	BAUTHOR VARCHAR(25),
  5  BPUBLISHER VARCHAR(25)
  6  );

#### Create Student table

SQL> CREATE TABLE STUDENT (
  2  	USN NUMBER(10) PRIMARY KEY,
  3  	SNAME VARCHAR(10),
  4  	AGE NUMBER(2)
  5  );

#### Create Borrowed table

SQL> CREATE TABLE BORROWED ( 
  2  	USN NUMBER(10),
  3  	ISBN NUMBER(12),
  4  	BORROWED_DATE VARCHAR(10),
  5  	PRIMARY KEY(USN, ISBN),
  6  	FOREIGN KEY(USN) REFERENCES STUDENT(USN) ON DELETE CASCADE,
  7  	FOREIGN KEY(ISBN) REFERENCES BOOK(ISBN) ON DELETE CASCADE
  8  );
