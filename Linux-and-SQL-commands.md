# Linux Package Management & Shell I/O — Lab Notes

Part of my ongoing SOC analyst prep (Google Cybersecurity Certificate). This lab covered two things: managing software with APT on a Debian-based system, and getting comfortable with basic input/output in the Bash shell.

## Part 1: Package management with APT

Started by confirming APT was actually available by just running `apt` with no arguments. It came back with version info and usage instructions, which confirmed it was installed — no surprise since this is a Debian-based box and APT is the default there. On a different distro (Red Hat, CentOS, etc.) this would've been YUM or DNF instead.

From there I installed Suricata, a network intrusion detection/analysis tool:

```
sudo apt install suricata
```

Needed `sudo` here since installing software requires elevated privileges. Took a couple minutes, and the install output listed out dependencies being pulled in along with Suricata itself. Confirmed the install worked by just running `suricata`, which printed version and usage info (Suricata 4.1.2 in this case).

Then removed it:

```
sudo apt remove suricata
```

Ran `suricata` again afterward and got `-bash: /usr/bin/suricata: No such file or directory`, confirming the removal actually worked and didn't just look like it worked.

Next installed tcpdump, a packet capture tool:

```
sudo apt install tcpdump
```

To verify what was actually installed on the system, I used:

```
apt list --installed
```

This dumps every installed package, which is a lot since Linux ships with plenty of software by default. Had to scroll/search through it to confirm tcpdump showed up (it did, flagged `[installed]`) and Suricata didn't, since I'd already removed it.

Last step was reinstalling Suricata and re-checking the installed list to make sure both Suricata and tcpdump showed up together. They did.

**Takeaway:** APT install/remove/list is basic stuff, but the point of the exercise was really about verification — not just running a command and assuming it worked, but checking the actual state of the system afterward. That habit matters a lot more once you're doing this on production systems or during an incident.

## Part 2: Shell input/output basics

Second half was lower-level: how the shell handles input and returns output, using `echo` and `expr`.

`echo` just prints back whatever string you give it:

```
echo hello
echo "hello"
```

Both return `hello` — quotes don't change the output here, they just tell the shell to treat everything inside as one grouped string, which matters more once you're passing in special characters.

`expr` handles basic integer math. Framed this in a security-relevant way: say you've got 32 alerts total and only 8 needed real action — how many were false positives?

```
expr 32 - 8
```
→ `24`

Also ran a quick projection — averaging 3500 login attempts a month, what's the yearly total?

```
expr 3500 * 12
```
→ `42000`

Worth noting `expr` only does integer math — no decimals, everything gets rounded down. And every operator/term needs to be space-separated (`expr 32 - 8`, not `expr 32-8`) or it'll error out.

Wrapped up with `clear` to reset the terminal view.

**Takeaway:** Nothing here is advanced, but this is the muscle memory layer — knowing how the shell interprets input and returns output (or throws an error) before moving on to anything more complex like scripting or log parsing. Also a good reminder that command-line math like this is genuinely useful for quick triage calculations (false positive rates, volume projections) instead of pulling out a calculator or spreadsheet mid-shift.

---

# Linux File Navigation, grep, and File Management — Lab Notes

Continuing my SOC analyst prep (Google Cybersecurity Certificate). This batch of labs covered navigating a Linux directory structure, reading file contents, filtering with grep and piping, and basic file/directory management (create, move, remove, edit).

## Lab 1: Navigating directories and reading files

Scenario was investigating files under `/home/analyst` — the kind of thing you'd actually do when pulling a user access report during an investigation.

Started with the basics of orienting myself:

```
pwd
ls
```

`pwd` confirms the working directory, `ls` lists what's in it. From `/home/analyst` I moved into the reports directory:

```
cd /home/analyst/reports
ls
```

That showed a `users` subdirectory (along with the other report folders). Moved into it and read a user file:

```
cd /home/analyst/reports/users
ls
cat Q1_added_users.txt
```

`cat` dumps the whole file to the terminal — useful for quickly checking a user list for a specific employee's department or ID without opening anything in a GUI.

Then navigated over to the logs directory and checked the start of a log file instead of the whole thing:

```
cd /home/analyst/logs
ls
head server_logs.txt
```

`head` defaults to the first 10 lines, which is exactly what you want when you're skimming a log for warning/error entries without scrolling through the entire file.

**Takeaway:** `cd`, `pwd`, `ls`, `cat`, and `head` cover most of what you need to move through a filesystem and pull relevant content out of files quickly — which matters a lot when you're working entirely through a remote shell with no GUI.

## Lab 2: Filtering with grep and piping

Same file structure, but this time the goal was searching rather than just reading.

First, filtering a log file down to just the error lines:

```
cd /home/analyst/logs
grep error server_logs.txt
```

`grep` searches for a string and only prints matching lines — a lot faster than reading the whole log to manually pick out errors.

Next, finding files by name pattern using piping:

```
cd /home/analyst/reports/users
ls | grep Q1
```

This pipes the output of `ls` into `grep`, so instead of grep searching file contents, it's filtering the list of filenames itself — in this case, anything with "Q1" in the name. Did the same thing again for "access":

```
ls | grep access
```

Then searched inside specific files for specific data:

```
grep jhill Q2_deleted_users.txt
grep "Human Resources" Q4_added_users.txt
```

The second one needed quotes around "Human Resources" since it's two words — without quotes, grep would treat "Human" and "Resources" as separate arguments instead of one search string.

**Takeaway:** grep plus piping is the difference between manually scanning files and actually querying them. Being able to pull "every line with X" or "every filename containing Y" out of a directory full of logs and reports is a core part of triage — you're rarely reading everything start to finish, you're searching for what matters.

## Lab 3: File and directory management

This one was less about reading data and more about basic housekeeping — creating, moving, and removing files and directories, plus editing a file directly in the shell.

Created a new subdirectory for future log storage:

```
mkdir logs
ls
```

Removed a directory that was no longer needed:

```
rm -r temp
ls
```

(`-r` for recursive, since removing a directory means removing everything inside it too.)

Moved a file from one directory to another:

```
cd notes
mv Q3patches.txt ../reports/
```

`mv` handles both moving and renaming in Linux — there's no separate "move" vs "rename" command, it's the same operation depending on whether the destination path changes the name or not.

Removed a single file:

```
rm tempnotes.txt
```

Created a new empty file:

```
touch tasks.txt
```

`touch` is normally used to update a file's timestamp, but if the file doesn't exist yet it creates it empty — handy for quickly scaffolding a file before editing it.

Then opened it in nano to actually add content:

```
nano tasks.txt
```

Typed in a couple of lines, then saved and exited with `CTRL+X`, confirmed the save with `Y`, and confirmed the filename with `ENTER`. (Normally you'd use `CTRL+O` to save and `CTRL+X` to exit, but in a browser-based shell `CTRL+O` gets intercepted by the browser itself, so `CTRL+X` → `Y` → `ENTER` is the workaround.) Cleared the screen afterward since nano can leave some leftover text artifacts in a web terminal, then confirmed the file contents with `cat`.

**Takeaway:** `mkdir`, `rm`, `mv`, `touch`, and basic nano usage round out the file management side of working in a shell. Combined with navigation and grep from the earlier labs, this covers a decent chunk of what you'd actually be doing day to day investigating or organizing files on a Linux system with no GUI to fall back on.

---

# Perform a SQL Query — 

## Overview
As part of the **Databases and SQL** module in Google's Cybersecurity Certificate, I used SQL to investigate two aspects of a fictional organization's security posture:

1. **Device patch status** — identifying which employee machines needed an OS update.
2. **Login activity** — reviewing the `log_in_attempts` table for signs of unusual or unauthorized access.

The goal was to practice writing basic `SELECT` and `ORDER BY` statements — foundational skills for a SOC analyst who needs to pull and sort security-relevant data from a database quickly.

## Database Schema

**`machines`** — hardware and patch information

| Column | Description |
|---|---|
| `device_id` | Unique identifier for the device |
| `operating_system` | OS running on the device |
| `email_client` | Email client installed |
| `OS_patch_date` | Date the OS was last patched |
| `employee_id` | Employee assigned to the device |

**`log_in_attempts`** — user login activity

| Column | Description |
|---|---|
| `event_id` | Unique identifier for the login event |
| `username` | User attempting to log in |
| `login_date` | Date of the login attempt |
| `login_time` | Time of the login attempt |
| `country` | Country the attempt originated from |
| `ip_address` | IP address of the attempt |
| `success` | Whether the attempt succeeded (1/0) |

## Task 1: Identify Devices Needing an Update

To see which machines needed patching, I queried the relevant columns from `machines` rather than pulling every field with `SELECT *` — narrowing the columns keeps the output focused on what's actually needed for the update decision.

```sql
SELECT device_id, operating_system, OS_patch_date
FROM machines;
```

**Why these columns:** `device_id` identifies the machine, `operating_system` shows what's installed, and `OS_patch_date` tells us how current (or stale) the patch level is.

## Task 2: Investigate Login Activity

### Step 1 — Check login locations
The organization only expects login activity from the **United States, Canada, or Mexico**. To check for logins outside those countries, I first pulled the location data:

```sql
SELECT event_id, country
FROM log_in_attempts;
```

Scanning the results for countries outside the expected set flags potential unauthorized access attempts — e.g., a login originating from a country the organization has no offices or remote staff in is worth escalating.

### Step 2 — Check for after-hours activity
Next, I looked at *when* logins occurred, to catch attempts made outside normal business hours:

```sql
SELECT username, login_date, login_time
FROM log_in_attempts;
```

This isolates the fields needed to cross-reference login timestamps against expected working hours — a common way to spot suspicious activity like credential misuse outside the workday.

## Task 3: Sort Results with `ORDER BY`

To make the patch data easier to triage, I sorted the machines by patch date so the most outdated (and highest-priority) devices surface first:

```sql
SELECT device_id, operating_system, OS_patch_date
FROM machines
ORDER BY OS_patch_date;
```

By default, `ORDER BY` sorts ascending — so the oldest `OS_patch_date` values (the machines most overdue for an update) appear at the top of the results, making it easy to prioritize which devices IT should patch first.

## Key Takeaways
- `SELECT column1, column2 FROM table;` retrieves only the fields needed for the task — a small habit that keeps output readable when working with large security datasets.
- Reviewing raw login data (location, timestamp) is a first step in spotting anomalies before applying more advanced filtering (`WHERE`, `AND`/`OR`).
- `ORDER BY` turns a flat table into a prioritized list — useful for triage tasks like "which machines are most overdue for patching" or "which logins happened most recently."

---

# Filter a SQL Query — 

## Overview
This lab built on basic `SELECT` statements by introducing **filters** — using `WHERE` and `LIKE` to narrow query results to exactly the records needed. As a security analyst, being able to pull a targeted subset of data (rather than scanning an entire table) is essential for tasks like patch management, compliance notices, and incident response.

The scenario: gather specific information about employees, their machines, and departments to support running updates, posting a privacy notice, and alerting an employee about a machine issue.

## Database Schema

I started by running `DESCRIBE` on both tables to confirm exact column names and types before writing any queries:

```sql
DESCRIBE machines;
DESCRIBE employees;
```

`DESCRIBE` returns each column's name and data type without pulling any rows — a quick way to avoid typos in a `SELECT` statement.

## Task 1: List All Organization Machines

First, I pulled a baseline list of every machine and its operating system:

```sql
SELECT device_id, operating_system
FROM machines;
```

**Result:** 200 rows returned — the full inventory of organization machines.

## Task 2: Retrieve Machines Running OS 2

Machines running `OS 2` needed an update, so I filtered the `machines` table down to just that operating system using a `WHERE` clause:

```sql
SELECT device_id, operating_system
FROM machines
WHERE operating_system = 'OS 2';
```

**Result:** 80 machines were running OS 2 — this is the exact list IT would use to schedule the update.

**Note on syntax:** string values (like `'OS 2'`) must be wrapped in single quotes, while column names (like `operating_system`) are never quoted. Every statement also needs a closing semicolon or the shell keeps waiting for more input.

## Task 3: List Employees in Specific Departments

A confidential-information notice needed to go out to two departments, so I queried `employees` filtered by department.

**Finance department:**
```sql
SELECT *
FROM employees
WHERE department = 'Finance';
```
The first row returned had `employee_id` **1003**.

**Sales department:**
```sql
SELECT *
FROM employees
WHERE department = 'Sales';
```
**Result:** 33 employees work in Sales.

## Task 4: Identify Employee Machines

A machine in office `South-109` was reported faulty, and I needed to identify the employee using it before escalating to the wider South building.

**Single office lookup:**
```sql
SELECT *
FROM employees
WHERE office = 'South-109';
```
**Result:** the employee using that office is **jlansky**.

**Building-wide lookup with `LIKE`:**

Since all South building offices follow the pattern `South-###`, I used `LIKE` with a wildcard (`%`) instead of an exact match to catch every office in that building:

```sql
SELECT *
FROM employees
WHERE office LIKE 'South%';
```

The `%` acts as a wildcard for any number of characters. Placement matters:
- `'South%'` — matches anything *starting* with "South" (e.g., `South-109`, `South-210`)
- `'%South'` — matches anything *ending* with "South"
- `'%South%'` — matches "South" appearing *anywhere* in the value

**Result:** the first employee listed in the South building belongs to the **Finance** department.

## Key Takeaways
- `WHERE` narrows a query to only the rows that meet a specific condition — critical for isolating actionable data (e.g., only the machines that need patching) instead of manually scanning a full table.
- String values in `WHERE` clauses must be single-quoted; column and table names are not.
- `LIKE` with `%` wildcards enables pattern matching for cases where an exact match won't work — such as grouping all offices in a building by a shared naming convention.
- Running `DESCRIBE` before querying an unfamiliar table saves time and avoids errors from mistyped column names.

---

Overview

This lab extended basic filtering by introducing comparison operators for numeric and date/time data — essential for a security analyst investigating an incident timeline. The scenario: narrow down log_in_attempts records to a specific date range and time window as part of investigating a security incident.

Operators used in this lab:

Operator	Meaning
=	equal
>	greater than
<	less than
<>	not equal to
>=	greater than or equal to
<=	less than or equal to
Task 1: Retrieve Login Attempts After a Certain Date

Strictly after a date:

```sql
SELECT *
FROM log_in_attempts
WHERE login_date > '2022-05-09';

Result: 134 login attempts were made after 2022-05-09.

On or after a date:

```sql
SELECT *
FROM log_in_attempts
WHERE login_date >= '2022-05-09';

Result: 165 login attempts were made on or after 2022-05-09 — the extra records represent attempts made on 2022-05-09 itself, which the strict > in the first query excluded.

Task 2: Retrieve Logins in a Date Range

To narrow the investigation to a tighter window, I used BETWEEN ... AND ... to bound the search between two dates:

```sql
SELECT *
FROM log_in_attempts
WHERE login_date BETWEEN '2022-05-09' AND '2022-05-11';

Result: 123 login attempts fell between 2022-05-09 and 2022-05-11.

Note: BETWEEN is inclusive on both ends — it returns records matching the start and end dates as well as everything in between.

Task 3: Investigate Logins at Certain Times

The organization's typical work hours start at 07:00:00, so login attempts before that time are worth reviewing for anomalies.

All logins before working hours:

```sql
SELECT *
FROM log_in_attempts
WHERE login_time < '07:00:00';

The fifth record returned had the username eraab.

Narrowing to a one-hour window:

```sql
SELECT *
FROM log_in_attempts
WHERE login_time BETWEEN '06:00:00' AND '07:00:00';

The earliest login attempt in that window occurred at 06:01:31.

Note: time values, like dates, are placed in single quotes since they're treated as string literals in the query.

Task 4: Investigate Logins by Event ID

Finally, I filtered on event_id — a numeric column, so values here are not wrapped in quotes.

Event IDs of 100 or higher:

sql
SELECT event_id, username, login_date
FROM log_in_attempts
WHERE event_id >= 100;

The third result returned had a login_date of 2022-05-09.

Narrowing to a specific ID range:

sql
SELECT event_id, username, login_date
FROM log_in_attempts
WHERE event_id BETWEEN 100 AND 150;

The seventh result returned had the username bisles.

Key Takeaways
Comparison operators (>, <, >=, <=) let you filter numeric and date/time data with precision — critical when reconstructing an incident timeline down to the minute.
BETWEEN ... AND ... is inclusive of both boundary values, and is more readable than chaining two separate comparisons.
Date and time values are treated as strings and need single quotes; numeric values (like event_id) do not.
Narrowing a query step by step — first a broad date filter, then a tighter time window, then specific event IDs — mirrors how a real investigation progressively focuses on the relevant data.

---

## Overview
This lab moved from single-condition filters to **combined filters** using `AND`, `OR`, and `NOT` — the operators that let a security analyst express compound investigative criteria (e.g., "failed AND after hours," "Finance OR Sales," "everyone except IT"). The scenario continued the login-activity and employee-machine investigation from the earlier labs.

## Task 1: Retrieve After-Hours Failed Login Attempts

The team needed failed logins that occurred after business hours (18:00). This required combining two conditions with `AND`:

```sql
SELECT *
FROM log_in_attempts
WHERE login_time > '18:00' AND success = 0;
```

**Result:** 19 failed login attempts occurred after 18:00.

**Note:** `success` is a Boolean column (`1` = TRUE, `0` = FALSE) stored as MySQL's native Boolean representation, so `0` is written without quotes — unlike string or date values.

## Task 2: Retrieve Login Attempts on Specific Dates

To investigate a suspicious event on `2022-05-09`, the team also wanted the day before it, `2022-05-08`. Since a single row can't match two different dates at once, this called for `OR` rather than `AND`:

```sql
SELECT *
FROM log_in_attempts
WHERE login_date = '2022-05-09' OR login_date = '2022-05-08';
```

**Result:** 75 login attempts were made across these two days.

## Task 3: Retrieve Login Attempts Outside of Mexico

The `country` column stores Mexican entries inconsistently (`'MEX'` and `'MEXICO'`), so an exact-match filter wouldn't catch both. I combined `NOT` with `LIKE` and a wildcard pattern to exclude anything starting with "MEX":

```sql
SELECT *
FROM log_in_attempts
WHERE NOT country LIKE 'MEX%';
```

**Result:** 144 login attempts originated outside of Mexico.

## Task 4: Retrieve Employees in Marketing (East Building)

For the machine-update tasks, I needed employees who satisfied **two conditions at once**: in the Marketing department *and* located in the East building. Both conditions had to hold for the same row, so this used `AND` together with `LIKE` for the office-naming pattern:

```sql
SELECT *
FROM employees
WHERE department = 'Marketing' AND office LIKE 'East%';
```

**Result:** the first employee returned had the username **elarson**.

## Task 5: Retrieve Employees in Finance or Sales

A separate update needed to reach anyone in *either* Finance or Sales. Even though both conditions check the same column, each condition has to be written out in full with `OR`:

```sql
SELECT *
FROM employees
WHERE department = 'Finance' OR department = 'Sales';
```

**Result:** the first employee in the Sales department returned was **lrodriqu**.

## Task 6: Retrieve All Employees Not in IT

The final update had already been applied to Information Technology, so the remaining rollout needed everyone *except* that department. `NOT` combined with an equality check handled the exclusion:

```sql
SELECT *
FROM employees
WHERE NOT department = 'Information Technology';
```

**Result:** 188 employees are outside the Information Technology department.

## Key Takeaways
- `AND` requires **both** conditions to be true for a row to be returned — used here for compound criteria like "failed AND after hours" or "Marketing AND East building."
- `OR` requires **either** condition to be true — necessary when checking if a single column could match one of several values (e.g., two possible dates, two possible departments). Each condition must be spelled out in full, even when checking the same column twice.
- `NOT` inverts a condition, which is often the fastest way to say "everyone except X" without listing every other possible value.
- Boolean columns (like `success`) are compared with unquoted `1`/`0`, distinguishing them from quoted string or date comparisons.
- Combining `LIKE` with `AND`/`NOT` handles real-world data inconsistencies (like `'MEX'` vs `'MEXICO'`) that an exact match would miss.

---
