

#### 184. Department Highest Salary

[Problem](https://leetcode.com/problems/department-highest-salary/)

```sql
WITH max_sal  AS (
SELECT departmentId, MAX(salary) as salary
FROM employee
GROUP BY 1
)

SELECT d.name as Department, e.name Employee, e.salary
FROM Employee e
JOIN Department d ON  e.departmentId = d.id
WHERE (departmentId, salary) IN (SELECT * FROM max_sal)
```
