# Lab-05_3 大學課程註冊系統

## 情境說明

設計一個大學課程註冊系統的資料庫，管理學生、課程、講師以及學生的選課記錄。

初步收集的資料包含：

- 學生：學號、學生姓名、主修科系名稱、主修科系辦公室地點  
- 課程：課程代碼、課程名稱、學分數、開課系所代碼、開課系所名稱  
- 講師：講師編號、講師姓名、講師所屬系所代碼、講師所屬系所名稱、講師辦公室號碼  
- 選課記錄：學號、學生姓名、課程代碼、課程名稱、學期、授課講師編號、授課講師姓名、成績  

假設：

- 一位講師可能教授多門課程，一門課程也可能由多位講師講授（例如，不同班級或共同授課）  
- 學生選修特定學期的特定課程，由特定講師授課  

---

## 1. 函數相依性分析

- 學號 → 學生姓名、主修科系名稱、主修科系辦公室地點  
- 課程代碼 → 課程名稱、學分數、開課系所代碼、開課系所名稱  
- 講師編號 → 講師姓名、講師所屬系所代碼、講師所屬系所名稱、講師辦公室號碼  
- (學號, 課程代碼, 學期) → 授課講師編號、成績  
- 授課講師編號 → 授課講師姓名  

---

## 2. 正規化至 BCNF 說明

- 原始資料表中包含許多重複資料與部分函數相依，造成資料異常風險。  
- 依函數相依性分析拆分以下資料表：  
  - 學生表（學號為主鍵）  
  - 系所表（系所代碼為主鍵，管理系所名稱與辦公室地點）  
  - 課程表（課程代碼為主鍵）  
  - 講師表（講師編號為主鍵）  
  - 授課表（課程代碼、講師編號、學期為複合主鍵，紀錄授課關係）  
  - 選課記錄表（學號、課程代碼、學期為複合主鍵，紀錄學生選課與成績）  

- 此設計達到 BCNF，消除所有部分與傳遞函數相依，降低資料異常。  

---

## 3. 最終 ERD 簡述

- 學生、系所、課程、講師皆為獨立實體。  
- 授課表連接課程與講師，表示多對多授課關係並區分學期。  
- 選課記錄表連接學生與授課課程，並包含成績。  

---

## 4. MariaDB 建表語句範例

```sql
CREATE TABLE Departments (
  DeptCode VARCHAR(10) PRIMARY KEY,
  DeptName VARCHAR(100),
  OfficeLocation VARCHAR(100)
);

CREATE TABLE Students (
  StudentID VARCHAR(20) PRIMARY KEY,
  StudentName VARCHAR(100),
  DeptCode VARCHAR(10),
  FOREIGN KEY (DeptCode) REFERENCES Departments(DeptCode)
);

CREATE TABLE Courses (
  CourseCode VARCHAR(20) PRIMARY KEY,
  CourseName VARCHAR(100),
  Credits INT,
  DeptCode VARCHAR(10),
  FOREIGN KEY (DeptCode) REFERENCES Departments(DeptCode)
);

CREATE TABLE Lecturers (
  LecturerID VARCHAR(20) PRIMARY KEY,
  LecturerName VARCHAR(100),
  DeptCode VARCHAR(10),
  OfficeNumber VARCHAR(50),
  FOREIGN KEY (DeptCode) REFERENCES Departments(DeptCode)
);

CREATE TABLE CourseAssignments (
  CourseCode VARCHAR(20),
  LecturerID VARCHAR(20),
  Semester VARCHAR(10),
  PRIMARY KEY (CourseCode, LecturerID, Semester),
  FOREIGN KEY (CourseCode) REFERENCES Courses(CourseCode),
  FOREIGN KEY (LecturerID) REFERENCES Lecturers(LecturerID)
);

CREATE TABLE Enrollment (
  StudentID VARCHAR(20),
  CourseCode VARCHAR(20),
  Semester VARCHAR(10),
  LecturerID VARCHAR(20),
  Grade VARCHAR(5),
  PRIMARY KEY (StudentID, CourseCode, Semester),
  FOREIGN KEY (StudentID) REFERENCES Students(StudentID),
  FOREIGN KEY (CourseCode, LecturerID, Semester) REFERENCES CourseAssignments(CourseCode, LecturerID, Semester)
);

