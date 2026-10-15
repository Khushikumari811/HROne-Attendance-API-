# Starter code review

| # | Where | What is wrong | How to notice | Fix |
|---|---|---|---|---|
| 1 | `compute_late_minutes` | Comparing floored minutes to `> 10` misses instants strictly after the 10-minute grace boundary until the floored value reaches 11. | A 09:40:01 punch for a 09:30 shift should be 10 late minutes. | Compare elapsed seconds to 600, then floor whole minutes. |
| 2 | `compute_late_minutes` | It assumes the punch-in and shift-start use the same timezone and does not account for the record's IST attendance date/overnight shift. | Run with UTC server settings or a 22:00–06:00 shift. | Convert stored UTC instants to IST at the boundary and derive shift instants from the record date. |
| 3 | `compute_work_hours` | Python `round` uses ties-to-even rather than the contract's half-up rounding. | Test a duration exactly halfway between hundredths. | Use `Decimal` with `ROUND_HALF_UP`. |
| 4 | `compute_overtime` | It compares a UTC/naive punch-out with a shift end created in IST, and anchors overnight shift end to the wrong day. | Punch out after midnight on an overnight shift. | Build shift end in IST on the correct calendar day, then compare UTC instants. |
| 5 | `compute_overtime` | It counts any positive overtime instead of requiring at least 30 whole minutes. | Punch out 29:59 after shift end. | Floor elapsed seconds to minutes and return zero below 30. |
| 6 | `create_employee` | A check-then-insert duplicate check is not safe under concurrent requests. | Send two concurrent creates for the same `emp_code`. | Enforce uniqueness with a MongoDB unique index and translate duplicate-key errors to 409. |
| 7 | `create_employee` / `EmployeeIn` | Input validation is incomplete: employee-code format, email, dates, shift formats, lengths and equal shift times are not checked. | Submit malformed values or identical shift times. | Add strict Pydantic validation. |
| 8 | `create_employee` | `created_at` is returned as an ISO datetime instead of epoch milliseconds. | Inspect the response against the OpenAPI schema. | Convert stored UTC datetime to epoch milliseconds. |
| 9 | `list_employees` | `total` ignores the department filter. | Query one department and compare `total` with filtered records. | Count with the same query used for the page. |
| 10 | `list_employees` | `skip = page * page_size` skips the first page and pagination parameters are not bounded. | Request page 1 with page size 20. | Use `(page - 1) * page_size`, validate page and cap page size at 100. |
| 11 | `list_employees` | Results are not sorted by `emp_code`, as required. | Insert codes out of order and list them. | Sort ascending in MongoDB. |
| 12 | `punch_in` | It does not return 404 for an unknown employee and can crash with a `KeyError`. | Punch in with an unknown code. | Check the employee first and return 404. |
| 13 | `punch_in` | `punched_at` uses local machine time and truthiness; invalid zero-like values can bypass the intended default behavior. | Set a non-IST server timezone or send `punched_at: 0`. | Validate epoch milliseconds strictly and default only when the field is omitted. |
| 14 | `punch_in` | It does not truncate instants to seconds or calculate the IST attendance date, including overnight shifts. | Use milliseconds with a fractional second or an overnight shift. | Truncate at the boundary and calculate the date in IST. |
| 15 | `punch_in` | Checking for an existing record before inserting is a race condition. | Send simultaneous punches for one employee/date. | Use a unique `(emp_code, date)` index; the losing insert gets 409. |
| 16 | `punch_in` | The status accepts arbitrary values and does not validate the allowed presence statuses. | Submit `status: "ABSENT"` or an unknown status. | Restrict the request model to PRESENT, WFH and ON_DUTY. |
| 17 | `list_attendance` | It fetches the entire matching collection into application memory, then sorts and paginates in Python. | Profile a query against 100,000 records. | Count separately and apply indexed sort, skip and limit in MongoDB. |
| 18 | `list_attendance` | It returns MongoDB `_id` as an `id`, even though the contract says records are addressed by `(emp_code, date)` and `_id` is never returned. | Inspect an attendance response. | Omit `_id` and do not synthesize an `id`. |
| 19 | `list_attendance` | Date formats and `date_from <= date_to` are not validated. | Send an impossible date or reversed range. | Validate dates and return 422 for invalid ranges. |
| 20 | Application startup / all endpoints | No indexes are created, so duplicate protection and expected query performance are not guaranteed. | Inspect indexes or run explain against a large dataset. | Create required unique and query indexes idempotently at startup. |

**Reviewed and not considered defects:** `compute_work_hours` correctly returns `None` when either time is missing; the natural key for attendance is `(emp_code, date)`, not MongoDB `_id`; and `history` is intended for manual corrections only, so punch-in/punch-out should not append audit entries.
