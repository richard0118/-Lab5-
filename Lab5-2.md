# Lab-05_2 線上商店訂單系統

## 情境說明

一家小型線上商店需要一個系統來管理顧客、產品和訂單。  
初步收集的資料包含：

- 顧客：顧客ID、顧客姓名、Email、電話號碼、完整送貨地址 (街道、城市、郵遞區號、國家)  
- 產品：產品ID、產品名稱、產品描述、單價、庫存數量、供應商名稱、供應商聯絡方式  
- 訂單：訂單ID、顧客ID、顧客姓名、訂單日期、訂單總金額、產品ID (多個)、產品名稱 (多個)、購買數量 (對應每個產品)、單價 (對應每個產品)  

---

## 1. 函數相依性分析

- 顧客ID → 顧客姓名、Email、電話號碼、完整送貨地址  
- 產品ID → 產品名稱、產品描述、單價、庫存數量、供應商名稱、供應商聯絡方式  
- 訂單ID → 顧客ID、訂單日期、訂單總金額  
- (訂單ID, 產品ID) → 購買數量、單價、產品名稱  

---

## 2. 正規化步驟說明

- 原始扁平化資料表中包含大量重複資料，例如同一顧客資訊重複出現、訂單中多筆產品明細列在同一表，導致資料異常。  
- 第一步拆分顧客資料表，顧客ID為主鍵，避免顧客資訊重複。  
- 第二步拆分產品資料表，產品ID為主鍵，集中管理產品資訊。  
- 第三步建立訂單資料表，訂單ID為主鍵，紀錄訂單基本資訊。  
- 第四步建立訂單明細資料表，以 (訂單ID, 產品ID) 為主鍵，管理訂單中各產品的數量與價格。  
- 此設計達到第三正規化 (3NF)，消除資料冗餘與更新異常。  

---

## 3. MariaDB 建表語句及範例資料

```sql
-- 顧客資料表
CREATE TABLE Customers (
  CustomerID INT PRIMARY KEY,
  CustomerName VARCHAR(100),
  Email VARCHAR(100),
  Phone VARCHAR(20),
  Street VARCHAR(100),
  City VARCHAR(50),
  PostalCode VARCHAR(20),
  Country VARCHAR(50)
);

-- 產品資料表
CREATE TABLE Products (
  ProductID INT PRIMARY KEY,
  ProductName VARCHAR(100),
  Description TEXT,
  UnitPrice DECIMAL(10,2),
  StockQty INT,
  SupplierName VARCHAR(100),
  SupplierContact VARCHAR(100)
);

-- 訂單資料表
CREATE TABLE Orders (
  OrderID INT PRIMARY KEY,
  CustomerID INT,
  OrderDate DATE,
  TotalAmount DECIMAL(10,2),
  FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

-- 訂單明細資料表
CREATE TABLE OrderDetails (
  OrderID INT,
  ProductID INT,
  Quantity INT,
  UnitPrice DECIMAL(10,2),
  PRIMARY KEY (OrderID, ProductID),
  FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
  FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);

-- 範例資料

INSERT INTO Customers VALUES
(1, 'Alice Chen', 'alice@example.com', '0912345678', '123 Main St', 'Taipei', '100', 'Taiwan'),
(2, 'Bob Wang', 'bob@example.com', '0987654321', '456 Elm St', 'Kaohsiung', '800', 'Taiwan'),
(3, 'Carol Lin', 'carol@example.com', '0922333444', '789 Oak St', 'Taichung', '400', 'Taiwan');

INSERT INTO Products VALUES
(101, 'Wireless Mouse', 'Ergonomic wireless mouse', 599.00, 50, 'TechSupply', 'contact@techsupply.com'),
(102, 'Mechanical Keyboard', 'RGB mechanical keyboard', 2499.00, 30, 'KeyMasters', 'sales@keymasters.com'),
(103, 'USB-C Hub', '7-in-1 USB-C hub', 799.00, 20, 'HubWorld', 'support@hubworld.com');

INSERT INTO Orders VALUES
(1001, 1, '2025-05-01', 1198.00),
(1002, 2, '2025-05-02', 2499.00),
(1003, 3, '2025-05-03', 799.00);

INSERT INTO OrderDetails VALUES
(1001, 101, 2, 599.00),
(1002, 102, 1, 2499.00),
(1003, 103, 1, 799.00);

-- 查詢特定顧客的所有訂單 (例如 CustomerID = 1)
SELECT * FROM Orders WHERE CustomerID = 1;

-- 查詢特定訂單的所有訂單項目及其產品名稱 (例如 OrderID = 1001)
SELECT od.OrderID, od.ProductID, p.ProductName, od.Quantity, od.UnitPrice
FROM OrderDetails od
JOIN Products p ON od.ProductID = p.ProductID
WHERE od.OrderID = 1001;

-- 計算每個產品的總銷售數量
SELECT ProductID, SUM(Quantity) AS TotalSold
FROM OrderDetails
GROUP BY ProductID;

-- 找出購買總金額最高的前三位顧客
SELECT c.CustomerID, c.CustomerName, SUM(o.TotalAmount) AS TotalSpent
FROM Customers c
JOIN Orders o ON c.CustomerID = o.CustomerID
GROUP BY c.CustomerID, c.CustomerName
ORDER BY TotalSpent DESC
LIMIT 3;
