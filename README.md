# SQL-learning-journal-day---35

# Penetration Test Assignment - hack.sql

## Overview

This SQL script performs a covert penetration test on a small enterprise’s SQLite database powering a website. The goal was to:

- Alter the password of the website’s administrative account (`admin`) to a new password (`oops!`), securely hashed.
- Erase any logs recording the password change to avoid detection.
- Add false log data framing a user (`emily33`) by making it appear as if the admin password was changed to emily33's password.

---

## Approach

### 1. Update the Admin Password
- Used an `UPDATE` statement to change the password in the `users` table.
- Passwords are stored as MD5 hashes; the new password hash was generated externally.

### 2. Remove Real Password Change Logs
- The database has triggers that automatically log user updates in the `user_logs` table.
- Deleted the real log entry created by the password change to avoid detection.

### 3. Insert a Fake Log Entry
- Inserted a crafted log record to frame the user `emily33`.
- The fake log entry shows the admin password changed to emily33’s hashed password.
- This misleads anyone auditing logs, throwing off suspicion.

---

## Key SQL Snippets

```sql
-- 1. Update admin password to 'oops!' (hashed)
UPDATE "users"
SET "password" = 'e5d8870e5bdd26602cab8dbe07a942c1'
WHERE "username" = 'admin';

-- 2. Delete real log entry of this change
DELETE FROM "user_logs"
WHERE "id" = 52;

-- 3. Insert fake log framing 'emily33'
INSERT INTO "user_logs" ("type", "old_username", "new_username", "old_password", "new_password")
SELECT 'update',
       "username",
       "username",
       "password",
       (SELECT "password" FROM "users" WHERE "username" = 'emily33')
FROM "users"
WHERE "username" = 'admin';
