############################################################### ADD / DROP / ALTER Constraints #########################################


/* ----------------------------------------------------------
   SETUP: create the two tables exactly as you shared
-----------------------------------------------------------*/
DROP TABLE IF EXISTS tasks;
DROP TABLE IF EXISTS users;

CREATE TABLE users (
  user_id     INT NOT NULL AUTO_INCREMENT,
  name        VARCHAR(100) NOT NULL,
  email       VARCHAR(255) NOT NULL,
  national_id VARCHAR(32) NULL,                     -- UNIQUE but nullable (multiple NULLs allowed)
  status      ENUM('active','inactive') NOT NULL DEFAULT 'active',
  created_at  TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT PK_users PRIMARY KEY (user_id),
  CONSTRAINT UQ_users_email UNIQUE (email),
  CONSTRAINT UQ_users_national UNIQUE (national_id),
  CONSTRAINT CK_users_email CHECK (email LIKE '%@%') -- simple format guard
) ENGINE=InnoDB;

CREATE TABLE tasks (
  task_id         INT NOT NULL AUTO_INCREMENT,
  title           VARCHAR(200) NOT NULL,
  created_by      INT  NOT NULL,   -- FK to users (strict parent)
  assigned_to     INT  NULL,       -- FK to users (optional parent)
  status          ENUM('todo','doing','done') NOT NULL DEFAULT 'todo',
  estimated_hours INT  NOT NULL DEFAULT 1,
  created_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

  CONSTRAINT PK_tasks PRIMARY KEY (task_id),
  CONSTRAINT UQ_tasks_title_creator UNIQUE (title, created_by),           -- composite UNIQUE
  CONSTRAINT CK_tasks_title_nonempty CHECK (LENGTH(title) > 0),
  CONSTRAINT CK_tasks_hours_pos      CHECK (estimated_hours > 0),

  CONSTRAINT FK_tasks_created_by
    FOREIGN KEY (created_by)
    REFERENCES users(user_id)
    ON DELETE NO ACTION              -- behaves like RESTRICT in MySQL
    ON UPDATE CASCADE,

  CONSTRAINT FK_tasks_assigned_to
    FOREIGN KEY (assigned_to)
    REFERENCES users(user_id)
    ON DELETE SET NULL               -- keep task, null the assignee if user deleted
    ON UPDATE CASCADE
) ENGINE=InnoDB;

/* (Optional) seed a little data so FK demos make sense */
INSERT INTO users (name, email, national_id) VALUES
  ('Alice', 'alice@example.com', 'NAT-001'),
  ('Bob',   'bob@example.com',    NULL);
INSERT INTO tasks (title, created_by, assigned_to) VALUES
  ('Prepare slides', 1, 2),
  ('Collect feedback', 2, NULL);


/* ==========================================================
   1) PRIMARY KEY (PK)
   - Drop PK
   - Re-add PK (same column)
   - (Example) Make a COMPOSITE PK (drop old PK first)
   (Note: PK name is informational; you change by drop+add)
========================================================== */

-- Drop PK on users
ALTER TABLE users DROP PRIMARY KEY;

-- Re-add PK on users(user_id)
ALTER TABLE users
  ADD CONSTRAINT PK_users PRIMARY KEY (user_id);

-- Change PK on tasks to a composite (task_id, created_by) [DEMO],
-- then revert back to original single-column PK.
ALTER TABLE tasks DROP PRIMARY KEY;
ALTER TABLE tasks
  ADD CONSTRAINT PK_tasks PRIMARY KEY (task_id, created_by);

-- Revert: back to single-column PK on task_id
ALTER TABLE tasks DROP PRIMARY KEY;
ALTER TABLE tasks
  ADD CONSTRAINT PK_tasks PRIMARY KEY (task_id);


/* ==========================================================
   2) UNIQUE
   - Add UNIQUE (single & composite)
   - Drop UNIQUE (DROP INDEX <name>)
   - Rename UNIQUE (RENAME INDEX)
========================================================== */

-- Add a composite UNIQUE on users(name, email) just for demo
ALTER TABLE users
  ADD CONSTRAINT UQ_users_name_email UNIQUE (name, email);

-- Drop that composite UNIQUE
ALTER TABLE users
  DROP INDEX UQ_users_name_email;

-- Rename existing UNIQUE (email) to a clearer name
ALTER TABLE users
  RENAME INDEX UQ_users_email TO UQ_users_email_unique;

-- Add UNIQUE on tasks(title) alone (note: existing composite still exists)
ALTER TABLE tasks
  ADD CONSTRAINT UQ_tasks_title UNIQUE (title);

-- Drop that single-column UNIQUE
ALTER TABLE tasks
  DROP INDEX UQ_tasks_title;


/* ==========================================================
   3) CHECK
   - Drop CHECK
   - Add tighter CHECK
   - Modify CHECK (drop + add)
   (MySQL 8.0.16+ enforces CHECK)
========================================================== */

-- Drop email CHECK
ALTER TABLE users
  DROP CHECK CK_users_email;

-- Add a stricter email CHECK (very basic still)
ALTER TABLE users
  ADD CONSTRAINT CK_users_email_basic CHECK (email LIKE '%@%.%');

-- Modify CHECK on tasks hours: ensure 1 <= estimated_hours <= 100
ALTER TABLE tasks
  DROP CHECK CK_tasks_hours_pos;

ALTER TABLE tasks
  ADD CONSTRAINT CK_tasks_hours_range CHECK (estimated_hours BETWEEN 1 AND 100);


/* ==========================================================
   4) NOT NULL
   - Make a column NOT NULL / NULL (via MODIFY)
========================================================== */

-- Allow NULLs in users.name (for demo), then revert to NOT NULL
ALTER TABLE users
  MODIFY name VARCHAR(100) NULL;

ALTER TABLE users
  MODIFY name VARCHAR(100) NOT NULL;


/* ==========================================================
   5) DEFAULT
   - Set or drop DEFAULT (ALTER COLUMN ... SET/DROP DEFAULT)
   - Or change DEFAULT via MODIFY
========================================================== */

-- Change default of users.status to 'inactive'
ALTER TABLE users
  ALTER COLUMN status SET DEFAULT 'inactive';

-- Drop default on users.status
ALTER TABLE users
  ALTER COLUMN status DROP DEFAULT;

-- Re-establish via MODIFY (also good to demonstrate enum change if needed)
ALTER TABLE users
  MODIFY status ENUM('active','inactive') NOT NULL DEFAULT 'active';

-- Change default of tasks.estimated_hours to 2
ALTER TABLE tasks
  ALTER COLUMN estimated_hours SET DEFAULT 2;


/* ==========================================================
   6) FOREIGN KEY (FK)
   - Drop FK
   - Re-add FK with different ON DELETE/ON UPDATE actions
   - Rename FK: drop + add with new name
   (To change actions, always drop + add)
========================================================== */

-- A) Change tasks.assigned_to FK from SET NULL -> RESTRICT
ALTER TABLE tasks
  DROP FOREIGN KEY FK_tasks_assigned_to;

ALTER TABLE tasks
  ADD CONSTRAINT FK_tasks_assigned_to
  FOREIGN KEY (assigned_to)
  REFERENCES users(user_id)
  ON DELETE RESTRICT
  ON UPDATE CASCADE;

-- B) Change tasks.created_by FK from NO ACTION -> CASCADE (delete creator deletes tasks)
ALTER TABLE tasks
  DROP FOREIGN KEY FK_tasks_created_by;

ALTER TABLE tasks
  ADD CONSTRAINT FK_tasks_created_by
  FOREIGN KEY (created_by)
  REFERENCES users(user_id)
  ON DELETE CASCADE
  ON UPDATE CASCADE;

-- C) Rename FK by drop+add with a new name (example: to FK_tasks_creator_strict)
ALTER TABLE tasks
  DROP FOREIGN KEY FK_tasks_created_by;

ALTER TABLE tasks
  ADD CONSTRAINT FK_tasks_creator_strict
  FOREIGN KEY (created_by)
  REFERENCES users(user_id)
  ON DELETE NO ACTION
  ON UPDATE CASCADE;


/* ==========================================================
   7) MISC (useful related operations)
   - AUTO_INCREMENT reseed
   - RENAME INDEX (unique or normal)
   - Show constraints quickly
========================================================== */

-- Bump AUTO_INCREMENT start on tasks
ALTER TABLE tasks AUTO_INCREMENT = 1000;

-- Rename the national_id UNIQUE to a clearer name
ALTER TABLE users
  RENAME INDEX UQ_users_national TO UQ_users_national_id;

-- Peek at structure (uncomment in MySQL client)
-- SHOW CREATE TABLE users\G
-- SHOW CREATE TABLE tasks\G
