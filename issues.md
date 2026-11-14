# Lab 5: Static Analysis Issues – `society_dues.py`

Baseline (before): Pylint 5.81/10, Bandit 3 issues (1 Low, 2 Medium), Flake8 5 issues.
After: Pylint 10.00/10, Bandit 0 issues, Flake8 0 issues.

| Issue | Type | Line(s) | Description | Fix Approach |
|-------|------|---------|-------------|--------------|
| SQL injection via f-string (Bandit B608, Medium) | Security | 9 | `addDue` builds the INSERT by pasting `flat`, `month` and `amount` into the SQL string, so crafted input such as `A-1'); DROP TABLE dues;--` changes the query. | Use a parameterised query: `INSERT INTO dues VALUES (?, ?, ?, 0)` with a tuple of values (as `collect` already does). |
| Use of `eval` on user-supplied text (Bandit B307, Medium; Pylint W0123) | Security | 21 | `penalty(rule)` runs arbitrary Python from the `rule` string, enabling code execution. | Parse with `ast.parse(..., mode="eval")` and evaluate only numbers and `+ - * /` using a small whitelist-based evaluator; reject everything else. |
| Hardcoded password (Bandit B105, Low) | Security | 5 | `TREASURER_PASSWORD = "flat#101"` commits a secret to source control. | Read it from the environment: `os.environ.get("TREASURER_PASSWORD")`. |
| Mutable default argument (Pylint W0102) | Bug | 8 | `history=[]` is created once and shared across every call, so history leaks between calls. | Default to `None` and create a new list inside the function. |
| Overly broad `except Exception` (Pylint W0718) | Bug / Robustness | 22 | Swallows every error, including programming mistakes, and hides real failures. | Catch only the expected errors: `ValueError, SyntaxError, TypeError, ZeroDivisionError, RecursionError`. |
| Inconsistent return statements (Pylint R1710) | Bug | 12-16 | `collect` returns an amount when the row exists but implicitly returns `None` otherwise. | Add an explicit `return None` at the end. |
| `!= None` comparison (Pylint C0121 / Flake8 E711) | Style | 29 | Should use identity for `None`. The check is also dead code, because `fetchall()` always returns a list. | Removed the check and return the list comprehension directly. |
| Function name `addDue` not snake_case (Pylint C0103) | Style | 8 | Violates PEP 8 naming. | Renamed to `add_due` and updated the caller in `main`. |
| Missing module and function docstrings (Pylint C0114 / C0116) | Style | 1, 8, 12, 19, 27, 34 | No documentation of what each function does. | Added a module docstring and a one-line docstring per function. |
| Lines too long (Pylint C0301 / Flake8 E501) | Style | 4, 13, 15 | Lines exceed 100 (Pylint) and 79 (Flake8) characters. | Wrapped long SQL calls across lines. |
| Missing blank lines between functions (Flake8 E302) | Style | 12 | `collect` follows `addDue` without two blank lines. | Added two blank lines between top-level functions. |
| `main()` runs on import | Design | 42 | Importing the module executes the demo and prints output. | Wrapped in `if __name__ == "__main__":`. |
