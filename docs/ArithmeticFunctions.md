## ABS()

--Index Scan
SELECT ReferenceOrderLineID
FROM Production.TransactionHistory
WHERE ABS(ReferenceOrderID) > 50
