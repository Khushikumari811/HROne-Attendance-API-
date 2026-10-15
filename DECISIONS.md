# Design decisions

1. **Indexes.** I created unique indexes on `employees.emp_code` and `attendance_logs.(emp_code, date)` so duplicate employee codes and duplicate daily attendance cannot race through. Department/join-date and employee/date/status indexes support department headcount, employee monthly lookups and filtered attendance. A date/employee index supports month-range analytics and attendance ordering. I considered a standalone status index, but status has low cardinality and is usually paired with a date or employee filter, so it is not a useful first choice.

2. **Punch-in race.** Both requests can pass their employee lookup, but only one insert can satisfy the unique `(emp_code, date)` index. MongoDB accepts one; the other raises `DuplicateKeyError`, which the API converts to 409.

3. **Ties.** The leaderboard uses MongoDB `$rank`, not row position. If the cutoff rank is tied, every employee sharing that rank is returned, ordered by late minutes descending and employee code ascending.

4. **Headcount.** Department summary starts from employees who joined by month-end and looks up logs. Employees with no matching logs remain in the pipeline, so they still contribute to headcount.

5. **100x data.** I would pre-aggregate daily/department facts and use incremental materialized summaries, while retaining raw records for audit and drill-down.
