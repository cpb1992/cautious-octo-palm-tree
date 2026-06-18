# cautious-octo-palm-tree

# Video Game Platform Database & Performance Tuning Lab
A hands-on DBA portfolio project demonstrating relational schema design, automated data generation, performance indexing, and role-based security access management.

## 🛠️ Tech Stack
*   **Database Engine:** MySQL 8.0+
*   **Data Generation:** Python 3 (Pandas, Faker)
*   **Concepts Tested:** Primary/Foreign Keys, Indexing (`EXPLAIN`), RBAC Security, Data Anomalies

## 📋 Project Blueprint & Architecture

### 1. Data Generation & Pipeline
To simulate a production workload, I wrote a Python engine utilizing the `Faker` library to synthesize a 10,000-row player database. The pipeline intentionally builds real-world data science complications, including missing user metrics (`NULL` values) and extreme high-activity behaviors (outliers).

*   Source script: `make_game_data.py`
*   Output target: `video_game_stats.csv`

### 2. Relational Schema Design
```sql
CREATE TABLE player_stats (
    player_id INT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    favorite_genre VARCHAR(30),
    hours_played INT NULL,
    win_rate DECIMAL(5,2) NULL,
    account_status VARCHAR(15) DEFAULT 'Active',
    CONSTRAINT chk_win_rate CHECK (win_rate >= 0.00 AND win_rate <= 100.00)
);
```

---

## ⚡ Performance Tuning Case Study (Pillar 3)
**Problem:** The frontend web portal was executing full table scans when looking up users, causing performance bottlenecks at scale.

### Before Optimization (Full Table Scan)
Running a diagnostic audit using `EXPLAIN` revealed that finding a user forced MySQL to search every record:
```sql
EXPLAIN SELECT * FROM player_stats WHERE username = 'PixelKnight';
-- Result Type: ALL (Full Table Scan)
-- Rows Examined: 10,000
```

### The Fix
I deployed a structured B-Tree index targeted at the high-frequency lookup column:
```sql
CREATE INDEX idx_username ON player_stats (username);
```

### After Optimization (Index Scan)
```sql
EXPLAIN SELECT * FROM player_stats WHERE username = 'PixelKnight';
-- Result Type: const / ref (Index Lookup)
-- Rows Examined: 1
```
**Impact:** Reduced rows scanned by **99.99%**, dropping lookup latency from milliseconds to near-instantaneous execution.

---

## 🔒 Automated Least-Privilege Security (Pillar 4)
To eliminate manual intervention and enforce security standards, I designed an automated role-based access control (RBAC) blueprint.

```sql
-- 1. Establish the Role
CREATE ROLE 'data_analyst_role';
GRANT SELECT ON gaming_db.player_stats TO 'data_analyst_role';

-- 2. Map User to Role
CREATE USER 'alex_analyst'@'localhost' IDENTIFIED BY 'SecurePass2026!';
GRANT 'data_analyst_role' TO 'alex_analyst'@'localhost';
FLUSH PRIVILEGES;
```
If an analyst attempts unauthorized deletions (e.g., `DROP TABLE`), MySQL natively drops the query context and throws `ERROR 1142: DROP command denied`.
