Step-by-step: diagnose like a professional
1️⃣ Confirm SQL Server is in Mixed Mode

Run this in SSMS:

SELECT SERVERPROPERTY('IsIntegratedSecurityOnly');


Result:

1 → ❌ Windows-only auth (SQL logins WILL FAIL)

0 → ✅ Mixed Mode (required)

If result is 1:

Fix it:

SSMS → right-click server → Properties

Security → Authentication

Select SQL Server and Windows Authentication

OK

Restart SQL Server

Without this, rag_user will never work.

2️⃣ Verify the login actually exists
SELECT name, is_disabled
FROM sys.sql_logins
WHERE name = 'rag_user';


Expected:

row exists

is_disabled = 0

If no row → login was never created
If disabled → enable it:

ALTER LOGIN rag_user ENABLE;

3️⃣ Reset password (remove all doubt)
ALTER LOGIN rag_user
WITH PASSWORD = 'Haha100!' UNLOCK;


(SQL Server does not care that you reuse the same password.)

4️⃣ Map login to the database (CRITICAL)

Being a server login is not enough.

USE [adaptive rag];
GO
CREATE USER rag_user FOR LOGIN rag_user;
ALTER ROLE db_owner ADD MEMBER rag_user;
GO


If the user already exists, you’ll get an error — that’s fine.

5️⃣ Confirm database name EXACTLY
SELECT name FROM sys.databases;


If the name is actually:

Adaptive RAG

adaptive_rag

adaptive-rag

Then your connection string is wrong.

SQL Server is case-insensitive, but spaces and characters matter.

6️⃣ Try connecting with SSMS using SQL auth

This is a non-negotiable test.

In SSMS:

Authentication: SQL Server Authentication

Login: rag_user

Password: Haha100!

Server: YUEFANG

If this fails → Python will fail too.
Fix it here first.

7️⃣ If you installed a NEW instance (very common gotcha)

If you followed my earlier advice and created:

YUEFANG\RAGSQL


Then this is WRONG:

SERVER=YUEFANG


Correct:

"SERVER=YUEFANG\\RAGSQL;"


18456 happens constantly when people connect to the wrong instance.