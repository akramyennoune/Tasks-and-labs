# Apply Filters to SQL Queries

## Project Description
As a cybersecurity analyst, I'm often asked to investigate specific security events by filtering through large volumes of log and employee data to find exactly what matters. In this project, I used SQL filtering with `WHERE`, `AND`, `OR`, `NOT`, and `LIKE` to investigate suspicious login activity in a `log_in_attempts` table and to pull targeted employee records from an `employees` table for security updates. These queries demonstrate how a SOC analyst narrows down noisy datasets to the specific rows relevant to an incident or task.

---

## Retrieve After-Hours Failed Login Attempts

**Scenario:** A potential security incident was discovered after business hours. I needed to find all failed login attempts that occurred after 18:00.

```sql
SELECT *
FROM log_in_attempts
WHERE success = 0 AND login_time > '18:00:00';
```

**How it works:** This query selects all columns from the `log_in_attempts` table. The `WHERE` clause combines two conditions with `AND`, meaning both must be true for a row to be returned: `success = 0` filters for failed login attempts, and `login_time > '18:00:00'` filters for attempts that occurred after 6:00 PM. Using `AND` ensures the results only include failed attempts that also happened after hours, not just one condition or the other.

<img width="503" height="494" alt="19 row" src="https://github.com/user-attachments/assets/e4b7fd4f-18a4-413e-b9fc-c3f4fc6129c5" />

---

## Retrieve Login Attempts on Specific Dates

**Scenario:** A suspicious event occurred on 2022-05-09. I needed to review all login attempts from that day and the day before.

```sql
SELECT *
FROM log_in_attempts
WHERE login_date = '2022-05-09' OR login_date = '2022-05-08';
```

**How it works:** This query uses `OR` to return rows that match either condition: `login_date = '2022-05-09'` or `login_date = '2022-05-08'`. Unlike `AND`, `OR` only requires one of the conditions to be true, so the result set includes every login attempt from both dates combined.

<img width="590" height="442" alt="75 row" src="https://github.com/user-attachments/assets/1dcf653d-5426-4d3a-9800-f074a6e2bc2d" />

---

## Retrieve Login Attempts Outside of Mexico

**Scenario:** Investigators determined the suspicious activity did not originate in Mexico, so I needed to find all login attempts that occurred outside of Mexico.

```sql
SELECT *
FROM log_in_attempts
WHERE NOT country LIKE 'Mex%';
```

**How it works:** The `country` column contains inconsistent values for Mexico, such as `MEX` and `MEXICO`. The `LIKE` keyword with the `%` wildcard matches any value that starts with "Mex," regardless of what follows, catching both variations in one condition. Wrapping that condition in `NOT` reverses the filter, returning every row where the country does *not* start with "Mex," meaning all login attempts that occurred outside of Mexico.

<img width="584" height="256" alt="7 rows" src="https://github.com/user-attachments/assets/250d2fda-14a8-4289-9db7-f3a9ee67e83f" />

---

## Retrieve Employees in Marketing

**Scenario:** The team needed to perform security updates on Marketing department machines located in the East building.

```sql
SELECT *
FROM employees
WHERE department = 'Marketing' AND office LIKE 'East%';
```

**How it works:** This query combines an exact match with a pattern match using `AND`. `department = 'Marketing'` filters for employees in the Marketing department, while `office LIKE 'East%'` uses the `%` wildcard to match any office value that begins with "East" (e.g., East-170, East-195), regardless of the number that follows. Since both conditions are joined with `AND`, only Marketing employees whose office is in the East building are returned.

<img width="599" height="459" alt="sql" src="https://github.com/user-attachments/assets/4ba0b84e-ad9f-4581-bb4b-b71ddac29260" />


---

## Retrieve Employees in Finance or Sales

**Scenario:** The next security update targeted employee machines in the Sales and Finance departments.

```sql
SELECT *
FROM employees
WHERE department = 'Sales' OR department = 'Finance';
```

**How it works:** This query uses `OR` to return any row where the department is either 'Sales' or 'Finance'. Because `OR` only requires one condition to be true, the result set includes employees from both departments combined, rather than requiring an employee to belong to both (which would be impossible).

![Employees in Finance or Sales query](7_rows.PNG)

---

## Retrieve All Employees Not in IT

**Scenario:** Employees in the Information Technology department had already received a security update. The final task was to identify all employees who still needed it — everyone outside of IT.

```sql
SELECT *
FROM employees
WHERE NOT department = 'Information Technology';
```

**How it works:** This query uses `NOT` to reverse the condition `department = 'Information Technology'`, returning every row where the department does *not* equal Information Technology. This effectively pulls in employees from every other department (Marketing, Human Resources, Finance, Sales, etc.) in a single query.

<img width="603" height="499" alt="sql2" src="https://github.com/user-attachments/assets/b84c805e-4d48-4866-9117-8342492c4f49" />


---

## Summary
Across these six tasks, I used SQL filtering operators — `WHERE`, `AND`, `OR`, `NOT`, and `LIKE` — to investigate a simulated security incident and support routine security operations. I filtered login attempt data by time, date, and inconsistent country values to narrow in on suspicious after-hours and out-of-country activity, and filtered employee data by department and office location to identify machines needing security updates. This project reflects a core SOC analyst skill: quickly isolating the relevant subset of data from a much larger dataset to investigate incidents and support the team's operational needs.
