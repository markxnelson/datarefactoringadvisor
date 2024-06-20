# Collect Database Workload

## CREATE SQL TUNING SET

The provided PL/SQL code is using the `DBMS_SQLTUNE` package in Oracle to create a SQL Tuning Set (STS) named `tkdradata`. A SQL Tuning Set is a database object that contains a set of SQL statements along with their execution statistics and context information.

Create the SQL Tuning Set by running the following in the SQL Console. Past the text in the console worksheet and hit the **Run** button

```sql
BEGIN
  DBMS_SQLTUNE.CREATE_SQLSET (
      sqlset_name => 'MY_SQLTUNE_DATASET_NAME', 
      description => 'SQL data from MY_DATABASE_NAME schema'
  );
END;
/
```
Here's a breakdown of what the code does:
1. `BEGIN` and `END;` are the delimiters that define the start and end of the PL/SQL block.

2. `DBMS_SQLTUNE.CREATE_SQLSET` is a procedure from the `DBMS_SQLTUNE` package that creates a new SQL Tuning Set.

3. `sqlset_name => 'MY_SQLTUNE_DATASET_NAME'` is a parameter that specifies the name of the SQL Tuning Set to be created. In this case, the name is `MY_SQLTUNE_DATASET_NAME`.

4. `description => 'SQL data from MY_DATABASRE_NAME schema'` is an optional parameter that provides a description for the SQL Tuning Set. Here, the description is "SQL data from tkdradata schema".

5. The `/` at the end is a terminator that signals the end of the PL/SQL block in Oracle.

After executing this code, a new SQL Tuning Set named `MY_SQLTUNE_DATASET_NAME` will be created in the database. Initially, it will be empty, but you can populate it with SQL statements and their execution statistics using other procedures from the `DBMS_SQLTUNE` package.

SQL Tuning Sets are useful for various purposes, such as:

1. **Data Refactoring**: You can analyze the SQL statements in the tuning set to create a graph of join activity between tables.  The more join activity, the higher the affinity between the 2 tables.  Run community detection on the graph to iddentify communities (i.e. a bounded context for a microservice)

2. **SQL Tuning**: You can analyze the SQL statements in the tuning set to identify performance issues and apply tuning techniques like creating indexes, restructuring queries, or using hints.

3. **Workload Capture and Replay**: SQL Tuning Sets can be used to capture a production workload and replay it in a test environment for testing or tuning purposes.

Overall, the provided PL/SQL code is a preparatory step for working with SQL Tuning Sets in Oracle, which can be a valuable tool for SQL performance analysis and tuning.

## RUN LOAD

This is a University Schema and a work in Progress

For now, create the Student Schema by running the following in the SQL Console. Past the text in the console worksheet and hit the **Run Script** button

```
-- Students Table
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    DateOfBirth DATE NOT NULL,
    Gender VARCHAR(10) NOT NULL,
    Email VARCHAR(100) UNIQUE NOT NULL,
    PhoneNumber VARCHAR(20)
);

-- Addresses Table
CREATE TABLE Addresses (
    AddressID INT PRIMARY KEY,
    StudentID NUMBER CONSTRAINT fk_addresses_students REFERENCES Students(StudentID),
    Street VARCHAR(100) NOT NULL,
    City VARCHAR(50) NOT NULL,
    State VARCHAR(50) NOT NULL,
    ZipCode VARCHAR(10) NOT NULL,
    Country VARCHAR(50) NOT NULL
);

-- Colleges Table
CREATE TABLE Colleges (
    CollegeID INT PRIMARY KEY,
    CollegeName VARCHAR(100) NOT NULL
);

-- Departments Table
CREATE TABLE Departments (
    DepartmentID INT PRIMARY KEY,
    DepartmentName VARCHAR(100) NOT NULL,
    CollegeID NUMBER CONSTRAINT fk_departments_colleges REFERENCES Colleges(CollegeID)
);

-- Majors Table
CREATE TABLE Majors (
    MajorID INT PRIMARY KEY,
    MajorName VARCHAR(100) NOT NULL,
    DepartmentID NUMBER CONSTRAINT fk_majors_departments REFERENCES Departments(DepartmentID)   
);


-- StudentMajors Table
CREATE TABLE StudentMajors (
    StudentMajorID INT PRIMARY KEY,
    StudentID NUMBER CONSTRAINT fk_studentmajors_students REFERENCES Students(StudentID),
    MajorID NUMBER CONSTRAINT fk_studentmajors_majors REFERENCES Majors(MajorID)
);

-- Courses Table
CREATE TABLE Courses (
    CourseID NUMBER PRIMARY KEY,
    CourseName VARCHAR2(100) NOT NULL,
    CourseDescription CLOB,
    Credits NUMBER NOT NULL,
    DepartmentID NUMBER CONSTRAINT fk_courses_departments REFERENCES Departments(DepartmentID)
);

-- Enrollments Table
CREATE TABLE Enrollments (
    EnrollmentID INT PRIMARY KEY,
    StudentID NUMBER CONSTRAINT fk_enrollments_students REFERENCES Students(StudentID),
    CourseID NUMBER CONSTRAINT fk_enrollments_courses REFERENCES Courses(CourseID),
    SemesterYear VARCHAR(10) NOT NULL,
    Grade VARCHAR(5)
);

-- Transcripts Table
CREATE TABLE Transcripts (
    TranscriptID INT PRIMARY KEY,
    StudentID NUMBER CONSTRAINT fk_transcripts_students REFERENCES Students(StudentID),
    TranscriptDate DATE NOT NULL,
    GPA DECIMAL(4, 2) NOT NULL
);

-- StudentFinances Table
CREATE TABLE StudentFinances (
    StudentFinanceID INT PRIMARY KEY,
    StudentID NUMBER CONSTRAINT fk_studentfinances_students REFERENCES Students(StudentID),
    TuitionFee DECIMAL(10, 2) NOT NULL,
    RoomFee DECIMAL(10, 2),
    MealPlan DECIMAL(10, 2),
    OtherFees DECIMAL(10, 2),
    FinancialAidAmount DECIMAL(10, 2)
);
```

And create the Faculty And Staff Schema by running the following in the SQL Console

```
-- Roles Table
CREATE TABLE Roles (
    RoleID NUMBER PRIMARY KEY,
    RoleName VARCHAR2(50) NOT NULL,
    RoleDescription VARCHAR2(200)
);

-- Faculty Table
CREATE TABLE Faculty (
    FacultyID NUMBER PRIMARY KEY,
    FirstName VARCHAR2(50) NOT NULL,
    LastName VARCHAR2(50) NOT NULL,
    Title VARCHAR2(50) NOT NULL,
    DepartmentID NUMBER REFERENCES Departments(DepartmentID),
    HireDate DATE NOT NULL,
    TenureStatus VARCHAR2(20) CHECK (TenureStatus IN ('Tenured', 'Tenure-Track', 'Non-Tenure')),
    RoleID NUMBER REFERENCES Roles(RoleID)
);

-- Staff Table
CREATE TABLE Staff (
    StaffID NUMBER PRIMARY KEY,
    FirstName VARCHAR2(50) NOT NULL,
    LastName VARCHAR2(50) NOT NULL,
    JobTitle VARCHAR2(100) NOT NULL,
    DepartmentID NUMBER REFERENCES Departments(DepartmentID),
    HireDate DATE NOT NULL,
    EmploymentType VARCHAR2(20) CHECK (EmploymentType IN ('Full-Time', 'Part-Time')),
    RoleID NUMBER REFERENCES Roles(RoleID)
);

-- FacultyCourses Table
CREATE TABLE FacultyCourses (
    FacultyCourseID NUMBER PRIMARY KEY,
    FacultyID NUMBER REFERENCES Faculty(FacultyID),
    CourseID NUMBER REFERENCES Courses(CourseID),
    SemesterYear VARCHAR2(10) NOT NULL
);

-- Publications Table
CREATE TABLE Publications (
    PublicationID NUMBER PRIMARY KEY,
    PublicationTitle VARCHAR2(200) NOT NULL,
    PublicationDate DATE NOT NULL,
    PublicationType VARCHAR2(50) NOT NULL,
    FacultyID NUMBER REFERENCES Faculty(FacultyID)
);

-- Committees Table
CREATE TABLE Committees (
    CommitteeID NUMBER PRIMARY KEY,
    CommitteeName VARCHAR2(100) NOT NULL,
    CommitteeDescription CLOB
);

-- CommitteeMembers Table
CREATE TABLE CommitteeMembers (
    CommitteeMemberID NUMBER PRIMARY KEY,
    CommitteeID NUMBER REFERENCES Committees(CommitteeID),
    FacultyID NUMBER REFERENCES Faculty(FacultyID),
    MembershipStartDate DATE NOT NULL,
    MembershipEndDate DATE
);


-- FacultyCredentials Table
CREATE TABLE FacultyCredentials (
    CredentialID NUMBER PRIMARY KEY,
    FacultyID NUMBER REFERENCES Faculty(FacultyID),
    Degree VARCHAR2(50) NOT NULL,
    Institution VARCHAR2(100) NOT NULL,
    YearAwarded NUMBER NOT NULL
);

-- EmployeeBenefits Table
CREATE TABLE EmployeeBenefits (
    BenefitID NUMBER PRIMARY KEY,
    BenefitName VARCHAR2(100) NOT NULL,
    BenefitDescription VARCHAR2(500) NOT NULL,
    BenefitCost NUMBER NOT NULL,
    BenefitCurrency VARCHAR2(3) NOT NULL,
    BenefitEffectiveDate DATE NOT NULL,
    BenefitExpirationDate DATE,
    FacultyID NUMBER REFERENCES Faculty(FacultyID),    StaffID NUMBER REFERENCES Staff(StaffID),
    CONSTRAINT BenefitCurrency CHECK (BenefitCurrency IN ('USD', 'EUR', 'GBP', 'JPY'))
);


```

And run the queries

```
-- STUDENT QUERIES

BEGIN
  FOR i IN 1..100 LOOP 
    EXECUTE IMMEDIATE 'SELECT s.FirstName, s.LastName, c.CourseName, d.DepartmentName, e.Grade
                       FROM Students s
                       JOIN Enrollments e ON s.StudentID = e.StudentID
                       JOIN Courses c ON e.CourseID = c.CourseID
                       JOIN Departments d ON c.DepartmentID = d.DepartmentID';
  END LOOP;
END;
/


-- STUDENT PERSONAL INFO
BEGIN
  FOR i IN 1..100 LOOP 
    EXECUTE IMMEDIATE 'SELECT s.FirstName,s.LastName,a.Street,a.City,a.State,a.ZipCode,a.Country,t.GPA,sf.TuitionFee,sf.FinancialAidAmount
                       FROM Students s
                       JOIN Addresses a ON s.StudentID = a.StudentID
                       JOIN Transcripts t ON s.StudentID = t.StudentID
                       JOIN StudentFinances sf ON s.StudentID = sf.StudentID';
  END LOOP;
END;
/

-- STUDENT ENROLLMENTS
BEGIN
  FOR i IN 1..10 LOOP 
    EXECUTE IMMEDIATE 'SELECT s.StudentID, s.FirstName, s.LastName, e.CourseID, e.SemesterYear, sm.MajorID
                       FROM Students s
                       JOIN Enrollments e ON s.StudentID = e.StudentID
                       JOIN StudentMajors sm ON s.StudentID = sm.StudentID';
  END LOOP;
END;
/


-- STUDENT COURSES
BEGIN
  FOR i IN 1..10 LOOP
    EXECUTE IMMEDIATE 'SELECT s.FirstName, s.LastName, c.CollegeName, d.DepartmentName, m.MajorName, co.CourseName
                        FROM Students s
                        JOIN StudentMajors sm ON s.StudentID = sm.StudentID
                        JOIN Majors m ON sm.MajorID = m.MajorID
                        JOIN Departments d ON m.DepartmentID = d.DepartmentID
                        JOIN Colleges c ON d.CollegeID = c.CollegeID
                        JOIN Courses co ON d.DepartmentID = co.DepartmentID';
  END LOOP;
END;

-- FACULTY QUERIES

-- All Roles for a faculty member
BEGIN
  FOR i IN 1..10 LOOP 
    EXECUTE IMMEDIATE 'SELECT f.FirstName, f.LastName, r.RoleName
                        FROM Faculty f
                        JOIN Roles r ON f.RoleID = r.RoleID
                        WHERE f.FacultyID = 1';
  END LOOP;
END;
/

-- All Roles for a Staff Member
BEGIN
  FOR i IN 1..10 LOOP 
    EXECUTE IMMEDIATE 'SELECT s.FirstName, s.LastName, r.RoleName
                        FROM Staff s
                        JOIN Roles r ON s.RoleID = r.RoleID
                        WHERE s.StaffID = 1';
  END LOOP;
END;
/

-- All Employee Benefits for a Staff Member
BEGIN
  FOR i IN 1..10 LOOP 
    EXECUTE IMMEDIATE 'SELECT s.FirstName, s.LastName, eb.BenefitName, eb.BenefitDescription, eb.BenefitCost, eb.BenefitCurrency
                        FROM Staff s
                        JOIN EmployeeBenefits eb ON s.StaffID = eb.StaffID
                        WHERE s.StaffID = 1';
  END LOOP;
END;
/

-- All Employee Benefits for a Faculty Member
BEGIN
  FOR i IN 1..10 LOOP 
    EXECUTE IMMEDIATE 'SELECT f.FirstName, f.LastName, eb.BenefitName, eb.BenefitDescription, eb.BenefitCost, eb.BenefitCurrency
                        FROM Faculty f
                        JOIN EmployeeBenefits eb ON f.FacultyID = eb.FacultyID
                        WHERE f.FacultyID = 1';
  END LOOP;
END;
/

-- Committees and Committee Members for a Faculty Member
BEGIN
  FOR i IN 1..10 LOOP 
    EXECUTE IMMEDIATE 'SELECT c.CommitteeName, LISTAGG(f.FirstName) WITHIN GROUP (ORDER BY f.LastName) AS CommitteeMembers
                        FROM Committees c
                        JOIN CommitteeMembers cm ON c.CommitteeID = cm.CommitteeID
                        JOIN Faculty f ON cm.FacultyID = f.FacultyID
                        WHERE f.FacultyID = 1
                        GROUP BY c.CommitteeName';
  END LOOP;
END;
/


-- Courses for a Faculty Member
BEGIN
  FOR i IN 1..10 LOOP
    EXECUTE IMMEDIATE 'SELECT f.FirstName, f.LastName, c.CourseName, c.CourseDescription, c.Credits
                        FROM Faculty f
                        JOIN FacultyCourses fc ON f.FacultyID = fc.FacultyID
                        JOIN Courses c ON fc.CourseID = c.CourseID
                        WHERE f.FacultyID = 1';
  END LOOP;
END;
/

Publications for a Faculty Member
BEGIN
  FOR i IN 1..10 LOOP
    EXECUTE IMMEDIATE 'SELECT f.FirstName, f.LastName, p.PublicationTitle, p.PublicationType, p.PublicationDate, fc.Degree, fc.Institution, fc.YearAwarded
                        FROM Faculty f
                        JOIN Publications p ON f.FacultyID = p.FacultyID
                        JOIN FacultyCredentials fc ON f.FacultyID = fc.FacultyID
                        WHERE f.FacultyID = 1';
  END LOOP;
END;
/

```

Once these queries have completed, Login as ADMIN and populate the tuning set

## Load Data From Cursor Cache to SQL Tuning Set 

**Must Run as ADMIN**

The provided PL/SQL code is using the `DBMS_SQLTUNE` package in Oracle to load SQL statements from the cursor cache into a SQL Tuning Set (STS) named `MY_SQLTUNE_DATASET_NAME`.


```sql
DECLARE
  cur DBMS_SQLTUNE.SQLSET_CURSOR;
BEGIN
  OPEN cur FOR
    SELECT VALUE(P)
      FROM table(
        DBMS_SQLTUNE.SELECT_CURSOR_CACHE(
          'parsing_schema_name=upper(''USER_NAME'') and sql_text not like ''%OPT_DYN%''',
            NULL, NULL, NULL, NULL, 1, NULL,
          'ALL', 'NO_RECURSIVE_SQL')) P;

      DBMS_SQLTUNE.LOAD_SQLSET(sqlset_name => 'college-sql-snapshot1',
                          populate_cursor => cur,
                          sqlset_owner => 'college');      
END;
```

Here's a breakdown of what the code does:
1. `DECLARE` and `BEGIN`...`END` are the delimiters that define the start and end of the PL/SQL block.
2. `cur DBMS_SQLTUNE.SQLSET_CURSOR` declares a cursor variable `cur` of the type `DBMS_SQLTUNE.SQLSET_CURSOR`.
3. `OPEN cur FOR` opens the cursor `cur` and assigns the result of the following query to it.
4. `SELECT VALUE(P) FROM table(...)` is a way to unnest the collection returned by the `DBMS_SQLTUNE.SELECT_CURSOR_CACHE` function.
5. `DBMS_SQLTUNE.SELECT_CURSOR_CACHE` is a function that retrieves SQL statements from the cursor cache based on the specified filters.
   - `'parsing_schema_name=upper(''USER_NAME'') and sql_text not like ''%OPT_DYN%'''` is a filter condition that selects SQL statements from the cursor cache where the parsing schema name is 'TKDRADATA' (case-insensitive) and the SQL text does not contain the string 'OPT_DYN'.
   - The remaining parameters (`NULL, NULL, NULL, NULL, 1, NULL`) are placeholders for other optional filters.
   - `'ALL'` specifies that all SQL statements matching the filters should be returned.
   - `'NO_RECURSIVE_SQL'` specifies that recursive SQL statements should be excluded.

Load the SQL Tuning Set by running the following in the SQL Console. Past the text in the console worksheet and hit the **Run** button

```sql
  DBMS_SQLTUNE.LOAD_SQLSET(sqlset_name => 'MY_SQLTUNE_DATASET_NAME',
                          populate_cursor => cur,
                          sqlset_owner => 'USER_NAME');      
END;
```

6. `DBMS_SQLTUNE.LOAD_SQLSET` is a procedure that loads SQL statements into a SQL Tuning Set.
   - `sqlset_name => 'MY_SQLTUNE_DATASET_NAME'` specifies the name of the SQL Tuning Set to be populated.
   - `populate_cursor => cur` specifies the cursor variable `cur` that contains the SQL statements to be loaded into the tuning set.
   - `sqlset_owner => 'USER_NAME'` specifies the owner of the SQL Tuning Set, which is 'TKDRADATA' in this case.

So, the overall effect of this PL/SQL code is to:

1. Retrieve SQL statements from the cursor cache where the parsing schema is 'TKDRADATA' and the SQL text does not contain 'OPT_DYN'.
2. Load those SQL statements into a SQL Tuning Set named `tkdradata` owned by the `tkdradata` schema.

This process can be useful for capturing and analyzing SQL statements that are currently executing or have recently executed in the database. By loading these statements into a SQL Tuning Set, you can perform further analysis, tuning, or testing on them using the various features and procedures provided by the `DBMS_SQLTUNE` package.


## Verify

The following SQL query retrieves the number of distinct SQL statements (identified by their SQL_IDs) present in each SQL Tuning Set in the database.

```sql
select sqlset_name, count(distinct sql_id)
from dba_sqlset_plans
group by sqlset_name;
```
The output of this query will show the name of each SQL Tuning Set and the corresponding count of distinct SQL statements (SQL_IDs) that are part of that tuning set.

## Summary

The SQL Tuning Set is populated with data from collecting the statements found in the cursor cache.  Next step, create a graph from the data




## More SQL Tuning Set Queries

To see what is in a SQL Tuning Set, Login as ADMIN and run

```sql
SELECT sql_text
FROM dba_sqlset_statements
WHERE sqlset_name = 'MY_SQLTUNE_DATASET_NAME'
ORDER BY sql_id;
```

ADMIN can populate and view

USER_NAME can create and delete.  To delete the SQL Tuning Set for a user, run

```sql
EXEC DBMS_SQLSET.DROP_SQLSET('MY_SQLTUNE_DATASET_NAME');
```
