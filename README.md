# celebal-TaskC
--Query for the Task1
-- Step 1: Create the table
IF OBJECT_ID('Projects', 'U') IS NOT NULL
    DROP TABLE Projects;

CREATE TABLE Projects (
    Task_ID INT,
    Start_Date DATE,
    End_Date DATE
);

-- Step 2: Insert sample data
INSERT INTO Projects (Task_ID, Start_Date, End_Date) VALUES 
(1, '2015-10-01', '2015-10-02'),
(2, '2015-10-02', '2015-10-03'),
(3, '2015-10-03', '2015-10-04'),
(4, '2015-10-13', '2015-10-14'),
(5, '2015-10-14', '2015-10-15'),
(6, '2015-10-28', '2015-10-29'),
(7, '2015-10-30', '2015-10-31');

-- Step 3: Query to group projects and show output
WITH ProjectWithRow AS (
    SELECT *,
           ROW_NUMBER() OVER (ORDER BY Start_Date) AS rn
    FROM Projects
),
ProjectGroups AS (
    SELECT *,
           DATEADD(DAY, -rn, Start_Date) AS grp
    FROM ProjectWithRow
),
GroupedProjects AS (
    SELECT 
        MIN(Start_Date) AS Project_Start,
        MAX(End_Date) AS Project_End,
        COUNT(*) AS Duration
    FROM ProjectGroups
    GROUP BY grp
)
SELECT 
    CONVERT(VARCHAR(10), Project_Start, 120) AS Start_Date,
    CONVERT(VARCHAR(10), Project_End, 120) AS End_Date
FROM GroupedProjects
ORDER BY Duration ASC, Project_Start ASC;

-- ==============================
-- Task 2: Students + Friends + Packages
-- ==============================

-- Drop tables if exist
DROP TABLE IF EXISTS Friends;
DROP TABLE IF EXISTS Packages;
DROP TABLE IF EXISTS Students;

-- Create Students table
CREATE TABLE Students (
    ID INT,
    Name VARCHAR(100)
);

-- Create Friends table
CREATE TABLE Friends (
    ID INT,
    Friend_ID INT
);

-- Create Packages table
CREATE TABLE Packages (
    ID INT,
    Salary FLOAT
);

-- Insert sample data
INSERT INTO Students (ID, Name) VALUES 
(1, 'Ashley'),
(2, 'Samantha'),
(3, 'Julia'),
(4, 'Scarlet');

INSERT INTO Friends (ID, Friend_ID) VALUES 
(1, 2),
(2, 3),
(3, 4),
(4, 1);

INSERT INTO Packages (ID, Salary) VALUES 
(1, 15.20),
(2, 10.06),
(3, 11.55),
(4, 12.12);

-- Query for Task 2 Output
PRINT '--- Task 2 Output ---';
SELECT s.Name
FROM Friends f
JOIN Students s ON f.ID = s.ID
JOIN Packages p1 ON f.ID = p1.ID
JOIN Packages p2 ON f.Friend_ID = p2.ID
WHERE p2.Salary > p1.Salary
ORDER BY p2.Salary;

-- ==============================
-- Task 3: Symmetric Pairs
-- ==============================

-- Drop if exists
DROP TABLE IF EXISTS Functions;

-- Create Functions table
CREATE TABLE Functions (
    X INT,
    Y INT
);

-- Insert data
INSERT INTO Functions (X, Y) VALUES
(20, 20),
(20, 20),
(20, 21),
(23, 22),
(22, 23),
(21, 20);

-- Query for Task 3 Output
PRINT '--- Task 3 Output ---';
SELECT DISTINCT f1.X, f1.Y
FROM Functions f1
JOIN Functions f2 
  ON f1.X = f2.Y AND f1.Y = f2.X
WHERE f1.X <= f1.Y
ORDER BY f1.X;

--Query for Task 4 output
-- Section 1: Drop Tables (Optional, but useful for re-running the script)
-- Use these commands if you need to clear existing tables and start fresh.
-- You can comment them out if you are running this script for the very first time.
DROP TABLE IF EXISTS Submission_Stats;
DROP TABLE IF EXISTS View_Stats;
DROP TABLE IF EXISTS Challenges;
DROP TABLE IF EXISTS Colleges;
DROP TABLE IF EXISTS Contests;
GO


-- Section 2: Create Tables
-- These commands create the necessary tables based on your schema.
CREATE TABLE Contests (
    contest_id INT PRIMARY KEY,
    hacker_id INT,
    name VARCHAR(255)
);

CREATE TABLE Colleges (
    college_id INT PRIMARY KEY,
    contest_id INT,
    FOREIGN KEY (contest_id) REFERENCES Contests(contest_id)
);

CREATE TABLE Challenges (
    challenge_id INT PRIMARY KEY,
    college_id INT,
    contest_id INT,
    FOREIGN KEY (college_id) REFERENCES Colleges(college_id),
    FOREIGN KEY (contest_id) REFERENCES Contests(contest_id)
);

CREATE TABLE View_Stats (
    challenge_id INT,
    total_views INT,
    total_unique_views INT,
    FOREIGN KEY (challenge_id) REFERENCES Challenges(challenge_id)
);

CREATE TABLE Submission_Stats (
    challenge_id INT,
    total_submissions INT,
    total_accepted_submissions INT,
    FOREIGN KEY (challenge_id) REFERENCES Challenges(challenge_id)
);
GO


-- Section 3: Insert Sample Data
-- These commands populate the tables with the provided sample data.
INSERT INTO Contests (contest_id, hacker_id, name) VALUES
(66406, 79153, 'Angela'),
(66556, 60292, 'Rose'),
(94828, 80275, 'Frank');

INSERT INTO Colleges (college_id, contest_id) VALUES
(11219, 66406),
(32473, 66556),
(56685, 94828);

INSERT INTO Challenges (challenge_id, college_id, contest_id) VALUES
(18765, 11219, 66406),
(47127, 11219, 66406),
(60292, 32473, 66556),
(72974, 32473, 66556),
(75516, 56685, 94828);

INSERT INTO View_Stats (challenge_id, total_views, total_unique_views) VALUES
(47127, 25, 19),
(47127, 15, 14),
(18765, 43, 10),
(75516, 72, 13),
(60292, 11, 10),
(72974, 41, 15),
(75516, 75, 11);

INSERT INTO Submission_Stats (challenge_id, total_submissions, total_accepted_submissions) VALUES
(75516, 34, 12),
(47127, 27, 10),
(47127, 56, 18),
(75516, 74, 13),
(72974, 63, 8),
(72974, 28, 24),
(60292, 82, 14),
(47127, 28, 11);
GO


-- Section 4: Task 4 Query
-- This query uses CTEs to correctly aggregate sums and avoid inflation.
WITH one_college_contests AS (
  SELECT contest_id
  FROM Colleges
  GROUP BY contest_id
  HAVING COUNT(*) = 1
),
challenge_stats AS (
  SELECT ch.challenge_id, co.contest_id
  FROM Challenges ch
  JOIN Colleges co ON ch.college_id = co.college_id
),
agg_stats AS (
  SELECT cs.contest_id,
         SUM(COALESCE(ss.total_submissions, 0)) AS total_submissions,
         SUM(COALESCE(ss.total_accepted_submissions, 0)) AS total_accepted_submissions,
         SUM(COALESCE(vs.total_views, 0)) AS total_views,
         SUM(COALESCE(vs.total_unique_views, 0)) AS total_unique_views
  FROM challenge_stats cs
  LEFT JOIN Submission_Stats ss ON cs.challenge_id = ss.challenge_id
  LEFT JOIN View_Stats vs ON cs.challenge_id = vs.challenge_id
  GROUP BY cs.contest_id
)
SELECT con.contest_id,
       con.hacker_id,
       con.name,
       a.total_submissions,
       a.total_accepted_submissions,
       a.total_views,
       a.total_unique_views
FROM Contests con
JOIN one_college_contests occ ON con.contest_id = occ.contest_id
JOIN agg_stats a ON con.contest_id = a.contest_id
WHERE a.total_submissions + a.total_accepted_submissions + a.total_views + a.total_unique_views > 0
ORDER BY con.contest_id;

--Query for the Task5 output
WITH DailySubmissionCounts AS (
    SELECT 
        submission_date,
        COUNT(DISTINCT hacker_id) AS total_hackers
    FROM Submissions
    GROUP BY submission_date
),
HackerDailySubmissions AS (
    SELECT 
        submission_date,
        hacker_id,
        COUNT(submission_id) AS submission_count
    FROM Submissions
    GROUP BY submission_date, hacker_id
),
MaxDailySubmissions AS (
    SELECT 
        submission_date,
        MAX(submission_count) AS max_subs
    FROM HackerDailySubmissions
    GROUP BY submission_date
),
TopHackers AS (
    SELECT 
        hds.submission_date,
        hds.hacker_id,
        hds.submission_count
    FROM HackerDailySubmissions hds
    JOIN MaxDailySubmissions mds 
        ON hds.submission_date = mds.submission_date 
       AND hds.submission_count = mds.max_subs
),
FinalOutput AS (
    SELECT 
        dsc.submission_date,
        dsc.total_hackers,
        th.hacker_id,
        h.name,
        ROW_NUMBER() OVER (
            PARTITION BY th.submission_date 
            ORDER BY th.hacker_id
        ) AS rn
    FROM DailySubmissionCounts dsc
    JOIN TopHackers th 
        ON dsc.submission_date = th.submission_date
    JOIN Hackers h 
        ON th.hacker_id = h.hacker_id
)
SELECT 
    submission_date,
    total_hackers,
    hacker_id,
    name
FROM FinalOutput
WHERE rn = 1
ORDER BY submission_date;

--Query for the Task6 
-- 1. Drop the table if it already exists (optional but prevents errors)
IF OBJECT_ID('STATION', 'U') IS NOT NULL
    DROP TABLE STATION;

-- 2. Create the STATION table
CREATE TABLE STATION (
    ID INT,
    CITY VARCHAR(21),
    STATE VARCHAR(2),
    LAT_N FLOAT,
    LONG_W FLOAT
);

-- 3. Insert sample data into STATION
INSERT INTO STATION (ID, CITY, STATE, LAT_N, LONG_W) VALUES
(1, 'Seattle', 'WA', 47.60, 122.33),
(2, 'Los Angeles', 'CA', 34.05, 118.25),
(3, 'New York', 'NY', 40.71, 74.00),
(4, 'Miami', 'FL', 25.76, 80.19);

-- 4. Task 6: Query to calculate Manhattan Distance
SELECT 
    ROUND(ABS(MIN(LAT_N) - MAX(LAT_N)) + ABS(MIN(LONG_W) - MAX(LONG_W)), 4) 
    AS Manhattan_Distance
FROM STATION;

--Query for the Task7
-- Task 7: Print all prime numbers <= 1000 in a single line, separated by '&'

WITH Numbers AS (
    SELECT 2 AS Num
    UNION ALL
    SELECT Num + 1 FROM Numbers WHERE Num + 1 <= 1000
),
Primes AS (
    SELECT Num
    FROM Numbers n
    WHERE NOT EXISTS (
        SELECT 1
        FROM Numbers d
        WHERE d.Num < n.Num AND d.Num > 1 AND n.Num % d.Num = 0
    )
)
SELECT STRING_AGG(CAST(Num AS VARCHAR), '&') AS PrimeNumbers
FROM Primes
OPTION (MAXRECURSION 1000);

--Query for the Task8
CREATE TABLE WORK (
    Name VARCHAR(100),
    Occupation VARCHAR(100)
);

INSERT INTO WORk (Name, Occupation) VALUES
('Samantha', 'Doctor'),
('Julia', 'Actor'),
('Maria', 'Actor'),
('Meera', 'Singer'),
('Ashely', 'Professor'),
('Ketty', 'Professor'),
('Christeen', 'Professor'),
('Jane', 'Actor'),
('Jenny', 'Doctor'),
('Priya', 'Singer');
WITH Ranked AS (
    SELECT 
        Name,
        Occupation,
        ROW_NUMBER() OVER (PARTITION BY Occupation ORDER BY Name) AS rn
    FROM OCCUPATIONS
)
SELECT
    MAX(CASE WHEN Occupation = 'Doctor' THEN Name END) AS Doctor,
    MAX(CASE WHEN Occupation = 'Professor' THEN Name END) AS Professor,
    MAX(CASE WHEN Occupation = 'Singer' THEN Name END) AS Singer,
    MAX(CASE WHEN Occupation = 'Actor' THEN Name END) AS Actor
FROM Ranked
GROUP BY rn
ORDER BY rn;

--Query for the Task9
-- Create a new table with a unique name
CREATE TABLE TODO (
    N INT,
    P INT
);
GO

-- Insert sample data
INSERT INTO TODO (N, P) VALUES
(1, 2),
(3, 2),
(6, 8),
(9, 8),
(2, 5),
(8, 5),
(5, NULL);
GO

-- Query to determine node types
SELECT 
    N,
    CASE
        WHEN P IS NULL THEN 'Root'
        WHEN N NOT IN (SELECT DISTINCT P FROM TODO WHERE P IS NOT NULL) THEN 'Leaf'
        ELSE 'Inner'
    END AS NodeType
FROM TODO
ORDER BY N;
GO

--Query for the Task10
CREATE TABLE Industry (
    company_code VARCHAR(10),
    founder VARCHAR(50)
);

CREATE TABLE Head (
    lead_manager_code VARCHAR(10),
    company_code VARCHAR(10)
);

CREATE TABLE Senior_Head (
    senior_manager_code VARCHAR(10),
    lead_manager_code VARCHAR(10),
    company_code VARCHAR(10)
);

CREATE TABLE Leader (
    manager_code VARCHAR(10),
    senior_manager_code VARCHAR(10),
    lead_manager_code VARCHAR(10),
    company_code VARCHAR(10)
);

CREATE TABLE Worker (
    employee_code VARCHAR(10),
    manager_code VARCHAR(10),
    senior_manager_code VARCHAR(10),
    lead_manager_code VARCHAR(10),
    company_code VARCHAR(10)
);
-- Company Table
INSERT INTO Company VALUES 
('C1', 'Monika'),
('C2', 'Samantha');

-- Lead_Manager Table
INSERT INTO Lead_Manager VALUES 
('LM1', 'C1'),
('LM2', 'C2');

-- Senior_Manager Table
INSERT INTO Senior_Manager VALUES 
('SM1', 'LM1', 'C1'),
('SM2', 'LM1', 'C1'),
('SM3', 'LM2', 'C2');

-- Manager Table
INSERT INTO Manager VALUES 
('M1', 'SM1', 'LM1', 'C1'),
('M2', 'SM3', 'LM2', 'C2'),
('M3', 'SM3', 'LM2', 'C2');

-- Employee Table
INSERT INTO Employee VALUES 
('E1', 'M1', 'SM1', 'LM1', 'C1'),
('E2', 'M1', 'SM1', 'LM1', 'C1'),
('E3', 'M2', 'SM3', 'LM2', 'C2'),
('E4', 'M3', 'SM3', 'LM2', 'C2');
SELECT 
    C.company_code,
    C.founder,
    COUNT(DISTINCT LM.lead_manager_code) AS lead_manager_count,
    COUNT(DISTINCT SM.senior_manager_code) AS senior_manager_count,
    COUNT(DISTINCT M.manager_code) AS manager_count,
    COUNT(DISTINCT E.employee_code) AS employee_count
FROM Company C
LEFT JOIN Lead_Manager LM 
    ON C.company_code = LM.company_code
LEFT JOIN Senior_Manager SM 
    ON LM.lead_manager_code = SM.lead_manager_code AND LM.company_code = SM.company_code
LEFT JOIN Manager M 
    ON SM.senior_manager_code = M.senior_manager_code AND SM.company_code = M.company_code
LEFT JOIN Employee E 
    ON M.manager_code = E.manager_code AND M.company_code = E.company_code
GROUP BY C.company_code, C.founder
ORDER BY C.company_code;

--Query for the Task11
-- Drop tables if they already exist
DROP TABLE IF EXISTS Friends;
DROP TABLE IF EXISTS Packages;
DROP TABLE IF EXISTS Students;

-- Create Students table
CREATE TABLE Students (
    ID INT PRIMARY KEY,
    Name VARCHAR(50)
);

-- Create Friends table
CREATE TABLE Friends (
    ID INT,
    Friend_ID INT
);

-- Create Packages table
CREATE TABLE Packages (
    ID INT,
    Salary FLOAT
);
-- Insert sample data into Students
INSERT INTO Students (ID, Name) VALUES
(1, 'Ashley'),
(2, 'Samantha'),
(3, 'Julia'),
(4, 'Scarlet');

-- Insert sample data into Friends
INSERT INTO Friends (ID, Friend_ID) VALUES
(1, 2),
(2, 3),
(3, 4),
(4, 1);

-- Insert sample data into Packages
INSERT INTO Packages (ID, Salary) VALUES
(1, 15.20),
(2, 10.06),
(3, 11.55),
(4, 12.12);
SELECT S.Name
FROM Students S
JOIN Friends F ON S.ID = F.ID
JOIN Packages P1 ON S.ID = P1.ID
JOIN Packages P2 ON F.Friend_ID = P2.ID
WHERE P2.Salary > P1.Salary
ORDER BY P2.Salary;

--Query for the Task12
-- Drop tables if already exist to avoid conflict
DROP TABLE IF EXISTS OccupationTask12;

-- Step 1: Create Table
CREATE TABLE OccupationTask12 (
    ID INT,
    Country VARCHAR(50),
    JobFamily VARCHAR(50),
    Cost FLOAT
);

-- Step 2: Insert Sample Data
INSERT INTO OccupationTask12 (ID, Country, JobFamily, Cost) VALUES
(1, 'India', 'Engineering', 5000),
(2, 'India', 'HR', 2000),
(3, 'India', 'Finance', 3000),
(4, 'International', 'Engineering', 8000),
(5, 'International', 'HR', 3000),
(6, 'International', 'Finance', 4000);

-- Step 3: Query to get ratio of cost by job family & country in percentage
SELECT 
    Country,
    JobFamily,
    SUM(Cost) AS TotalCost,
    ROUND(
        100.0 * SUM(Cost) / SUM(SUM(Cost)) OVER(PARTITION BY Country), 
        2
    ) AS Percentage
FROM OccupationTask12
GROUP BY Country, JobFamily
ORDER BY Country, JobFamily;

--Query for the Task13
CREATE TABLE BU_MonthlyData (
    BU_Name VARCHAR(50),
    MonthYear VARCHAR(10),
    Cost DECIMAL(10,2),
    Revenue DECIMAL(10,2)
);
INSERT INTO BU_MonthlyData (BU_Name, MonthYear, Cost, Revenue)
VALUES 
('BU1', '2025-01', 20000, 40000),
('BU1', '2025-02', 22000, 41000),
('BU1', '2025-03', 25000, 43000),
('BU2', '2025-01', 18000, 36000),
('BU2', '2025-02', 19000, 37000),
('BU2', '2025-03', 21000, 39000);
SELECT 
    BU_Name,
    MonthYear,
    Cost,
    Revenue,
    CAST(Cost AS FLOAT) / NULLIF(Revenue, 0) AS CostToRevenueRatio
FROM 
    BU_MonthlyData;

--Query for the Task14
CREATE TABLE Employee_SubBand (
    Emp_ID INT,
    SubBand VARCHAR(20)
);
INSERT INTO Employee_SubBand (Emp_ID, SubBand)
VALUES
(1, 'SB1'),
(2, 'SB1'),
(3, 'SB2'),
(4, 'SB2'),
(5, 'SB2'),
(6, 'SB3'),
(7, 'SB1'),
(8, 'SB3'),
(9, 'SB3'),
(10, 'SB2');
SELECT 
    SubBand,
    COUNT(*) AS Headcount,
    CAST(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER() AS DECIMAL(5,2)) AS Percentage_Of_Headcount
FROM 
    Employee_SubBand
GROUP BY 
    SubBand;

    --Query for the Task15
-- Create EMPLOYEES table
CREATE TABLE EMPLOYEES (
    ID INT,
    Name VARCHAR(50),
    Salary FLOAT
);

-- Insert sample data
INSERT INTO EMPLOYEES (ID, Name, Salary)
VALUES
(1, 'Amit', 55000),
(2, 'Bhavna', 72000),
(3, 'Chirag', 64000),
(4, 'Divya', 82000),
(5, 'Esha', 75000),
(6, 'Farhan', 71000),
(7, 'Geeta', 69000),
(8, 'Hari', 53000),
(9, 'Isha', 60000),
(10, 'Jay', 77000);

-- Query to find top 5 salaries without using ORDER BY
WITH RankedSalaries AS (
    SELECT *,
           DENSE_RANK() OVER (ORDER BY Salary DESC) AS RankPos
    FROM EMPLOYEES
)
SELECT ID, Name, Salary
FROM RankedSalaries
WHERE RankPos <= 5;

--Query for the Task16
-- Drop the table first if it already exists (to avoid errors during re-run)
IF OBJECT_ID('dbo.TestTable', 'U') IS NOT NULL
    DROP TABLE dbo.TestTable;

-- Step 1: Create the table
CREATE TABLE TestTable (
    ID INT,
    ColumnA INT,
    ColumnB INT
);

-- Step 2: Insert sample data
INSERT INTO TestTable VALUES (1, 10, 20);

-- Step 3: Swap values WITHOUT using a third variable (in 3 separate steps)
UPDATE TestTable
SET ColumnA = ColumnA + ColumnB;

UPDATE TestTable
SET ColumnB = ColumnA - ColumnB;

UPDATE TestTable
SET ColumnA = ColumnA - ColumnB;

-- Step 4: Check the result
SELECT * FROM TestTable;


--Query for the Task17
-- Step 1: Create a SQL Server login (at the server level)
CREATE LOGIN BookStoreUser WITH PASSWORD = 'StrongPassword@123';

-- Step 2: Select your database
USE AdventureWorks2019;  -- Replace with your actual database name

-- Step 3: Create a user in that database mapped to the login
CREATE USER BookStoreUser FOR LOGIN BookStoreUser;

-- Step 4: Add the user to db_owner role
EXEC sp_addrolemember 'db_owner', 'BookStoreUser';
SELECT * 
FROM sys.database_principals 
WHERE name = 'BookStoreUser';

--Query for the Task18
-- Drop table if it exists
IF OBJECT_ID('dbo.EmployeeCost', 'U') IS NOT NULL
    DROP TABLE dbo.EmployeeCost;

-- Create the table
CREATE TABLE EmployeeCost (
    EmployeeID INT,
    BU VARCHAR(50),
    MonthYear DATE,
    Salary DECIMAL(10,2),
    Weight DECIMAL(5,2)
);

-- Insert sample data
INSERT INTO EmployeeCost VALUES 
(101, 'BU1', '2024-01-01', 50000, 1.0),
(102, 'BU1', '2024-01-01', 60000, 0.8),
(103, 'BU1', '2024-01-01', 55000, 0.5),
(104, 'BU2', '2024-01-01', 70000, 1.0),
(105, 'BU2', '2024-01-01', 80000, 0.9),
(101, 'BU1', '2024-02-01', 51000, 1.0),
(102, 'BU1', '2024-02-01', 62000, 0.8),
(104, 'BU2', '2024-02-01', 71000, 1.0),
(105, 'BU2', '2024-02-01', 81000, 0.9);
SELECT 
    BU,
    FORMAT(MonthYear, 'yyyy-MM') AS [Month],
    ROUND(SUM(Salary * Weight) / SUM(Weight), 2) AS WeightedAvgCost
FROM 
    EmployeeCost
GROUP BY 
    BU,
    FORMAT(MonthYear, 'yyyy-MM')
ORDER BY 
    BU, [Month];

--Query for the Task19
-- Step 1: Create sample Employees table with some data
IF OBJECT_ID('Employees', 'U') IS NOT NULL
    DROP TABLE Employees;

CREATE TABLE Employees (
    EmpID INT,
    Salary INT
);

-- Step 2: Insert some sample salaries
INSERT INTO Employees VALUES
(1, 10500),
(2, 20000),
(3, 30000),
(4, 15000),
(5, 12000);

-- Step 3: Calculate the error caused by removing 0s
SELECT CEILING(
    ABS(
        (SELECT AVG(CAST(Salary AS FLOAT)) FROM Employees)
        -
        (SELECT AVG(CAST(REPLACE(CAST(Salary AS VARCHAR), '0', '') AS FLOAT)) FROM Employees)
    )
) AS Salary_Error;

--Query for the Task20
-- Drop old tables if they exist
IF OBJECT_ID('SourceTable', 'U') IS NOT NULL
    DROP TABLE SourceTable;

IF OBJECT_ID('TargetTable', 'U') IS NOT NULL
    DROP TABLE TargetTable;

-- Create source and target tables
CREATE TABLE SourceTable (
    EmpID INT,
    EmpName VARCHAR(100),
    Salary INT
);

CREATE TABLE TargetTable (
    EmpID INT,
    EmpName VARCHAR(100),
    Salary INT
);

-- Insert sample data into SourceTable
INSERT INTO SourceTable (EmpID, EmpName, Salary)
VALUES 
(1, 'Alice', 5000),
(2, 'Bob', 6000),
(3, 'Charlie', 7000);

-- Insert one old row into TargetTable
INSERT INTO TargetTable (EmpID, EmpName, Salary)
VALUES 
(1, 'Alice', 5000);  -- Already exists

-- ✅ Step: Copy only NEW rows from SourceTable to TargetTable
INSERT INTO TargetTable (EmpID, EmpName, Salary)
SELECT s.EmpID, s.EmpName, s.Salary
FROM SourceTable s
WHERE NOT EXISTS (
    SELECT 1
    FROM TargetTable t
    WHERE t.EmpID = s.EmpID AND t.EmpName = s.EmpName AND t.Salary = s.Salary
);

-- Check the final data in TargetTable
SELECT * FROM TargetTable;
