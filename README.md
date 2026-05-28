# PortSwigger Web Security Academy — SQL Injection Labs (All 18 Completed)

> Personal notes from completing every SQL Injection lab on [PortSwigger Web Security Academy](https://portswigger.net/web-security/sql-injection).  
> Documented for learning purposes as part of my web application security journey.

📝 **XSS Write-up:** (https://medium.com/@sukansh.07/sql-injection-masterclass-solving-all-18-portswigger-labs-with-payloads-ee5969ae5986)](https://medium.com/@sukansh.07/sql-injection-masterclass-solving-all-18-portswigger-labs-with-payloads-ee5969ae5986)*  
🔗 **LinkedIn:** [Connect with me](#) *(https://www.linkedin.com/in/sukanshh/)*  

---

## What's in This Repo

Each lab has its own note file covering:
- The vulnerable parameter and attack surface
- The exact payload(s) used, step by step
- The reasoning behind each step
- Database-specific syntax where relevant

Labs are grouped by technique below — much more useful than going through them in order.

---

## Lab Index

### 🔴 Classic SQLi — In-Band (Labs 1–2)

| # | Lab | Key Technique |
|---|-----|---------------|
| 1 | [SQLi in WHERE clause — hidden data](labs/lab01-sqli-where-hidden-data.md) | `' OR 1=1--` to return all rows including unreleased |
| 2 | [SQLi login bypass](labs/lab02-sqli-login-bypass.md) | `administrator'--` to comment out the password check |

---

### 🟠 UNION Attacks (Labs 3–6)

| # | Lab | Key Technique |
|---|-----|---------------|
| 3 | [UNION — determining number of columns](labs/lab03-union-column-count.md) | NULL method and ORDER BY clause to find column count |
| 4 | [UNION — finding a column with text](labs/lab04-union-find-text-column.md) | Swap NULLs for strings to identify text-compatible columns |
| 5 | [UNION — retrieving data from other tables](labs/lab05-union-retrieve-data.md) | `UNION SELECT username, password FROM users--` |
| 6 | [UNION — retrieving multiple values in one column](labs/lab06-union-concatenation.md) | String concatenation (`\|\|`) to combine columns into one |

---

### 🟡 Database Enumeration (Labs 7–10)

| # | Lab | Key Technique |
|---|-----|---------------|
| 7 | [Query DB version — Oracle](labs/lab07-db-version-oracle.md) | `SELECT banner FROM v$version`, requires `FROM DUAL` |
| 8 | [Query DB version — MySQL & Microsoft](labs/lab08-db-version-mysql.md) | `SELECT @@version`, `#` as comment character |
| 9 | [List DB contents — non-Oracle](labs/lab09-list-contents-non-oracle.md) | `information_schema.tables` + `information_schema.columns` |
| 10 | [List DB contents — Oracle](labs/lab10-list-contents-oracle.md) | `all_tables` + `all_tab_columns` (Oracle-specific) |

---

### 🟢 Blind SQLi — Conditional Responses (Labs 11–12)

| # | Lab | Key Technique |
|---|-----|---------------|
| 11 | [Blind SQLi — conditional responses](labs/lab11-blind-conditional-responses.md) | "Welcome back" message as true/false oracle, enumerate password with Burp Intruder cluster bomb |
| 12 | [Blind SQLi — conditional errors](labs/lab12-blind-conditional-errors.md) | `CASE WHEN ... THEN TO_CHAR(1/0)` — trigger error on true, silence on false |

---

### 🔵 Blind SQLi — Time-Based (Labs 13–14)

| # | Lab | Key Technique |
|---|-----|---------------|
| 13 | [Blind SQLi — time delays](labs/lab13-blind-time-delays.md) | `pg_sleep(10)` to prove vulnerability — no output needed |
| 14 | [Blind SQLi — time delays + data retrieval](labs/lab14-blind-time-delays-data.md) | `CASE WHEN ... THEN pg_sleep(10)` to enumerate password character by character |

---

### 🟣 Blind SQLi — Out-of-Band (Labs 15–16)

| # | Lab | Key Technique |
|---|-----|---------------|
| 15 | [Blind SQLi — out-of-band interaction](labs/lab15-blind-oob-interaction.md) | Oracle XXE via `EXTRACTVALUE` + `SYSTEM` to trigger DNS lookup to Collaborator |
| 16 | [Blind SQLi — out-of-band data exfiltration](labs/lab16-blind-oob-exfiltration.md) | Embed SQL query inside DNS hostname — password exfiltrated via Collaborator DNS lookup |

---

### ⚫ Advanced (Labs 17–18)

| # | Lab | Key Technique |
|---|-----|---------------|
| 17 | [SQLi via XML encoding — WAF bypass](labs/lab17-sqli-xml-encoding.md) | Hackvertor extension encodes payload as hex entities to bypass WAF |
| 18 | [Visible error-based SQLi](labs/lab18-error-based-sqli.md) | `CAST((SELECT ...) AS int)` — database error message leaks data directly |

---

## Key Takeaways

**1. Always determine column count and data types first.** Every UNION attack starts here — `ORDER BY` to count columns, then swap NULLs for strings to find which columns accept text.

**2. Each database has its own syntax.** Oracle requires `FROM DUAL` in every SELECT. Comments differ: `--` for PostgreSQL/Oracle, `#` for MySQL. String concatenation is `||` in Oracle/PostgreSQL, `+` in Microsoft, `CONCAT()` in MySQL. Always check the cheat sheet.

**3. Blind SQLi is slower but just as powerful.** Without visible output, you turn every query into a yes/no question — through response differences, errors, or time delays. Burp Intruder's cluster bomb automates the character-by-character enumeration.

**4. Time-based blind SQLi needs a response metric.** Sort Intruder results by response time, not status code. Requests that took ~10 seconds are your `TRUE` results.

**5. Out-of-band is the last resort — and very effective.** When there's no visible output and time delays are unreliable, DNS exfiltration via Burp Collaborator gets the data out through a completely different channel.

**6. WAFs can be bypassed at the encoding layer.** Lab 17 showed that encoding the payload as XML hex entities fools the WAF without affecting how the database interprets the query.

**7. Error messages are goldmines.** Lab 18 used `CAST()` type errors to leak actual data values — the database helpfully told us what it was trying to convert.

**8. `information_schema` is your map.** On non-Oracle databases, `information_schema.tables` and `information_schema.columns` give you the full structure of every table in the database. On Oracle, use `all_tables` and `all_tab_columns`.

---

## SQLi Cheat Sheet (Quick Reference)

### Comment Characters
| Database | Comment |
|----------|---------|
| Oracle | `--` |
| PostgreSQL | `--` |
| MySQL | `#` or `-- ` (note space) |
| Microsoft | `--` |

### String Concatenation
| Database | Syntax |
|----------|--------|
| Oracle / PostgreSQL | `'foo'\|\|'bar'` |
| Microsoft | `'foo'+'bar'` |
| MySQL | `CONCAT('foo','bar')` |

### Version Query
| Database | Query |
|----------|-------|
| Oracle | `SELECT banner FROM v$version` |
| PostgreSQL | `SELECT version()` |
| MySQL / Microsoft | `SELECT @@version` |

### List Tables
| Database | Query |
|----------|-------|
| Oracle | `SELECT table_name FROM all_tables` |
| Non-Oracle | `SELECT table_name FROM information_schema.tables` |

### List Columns
| Database | Query |
|----------|-------|
| Oracle | `SELECT column_name FROM all_tab_columns WHERE table_name='X'` |
| Non-Oracle | `SELECT column_name FROM information_schema.columns WHERE table_name='X'` |

### Time Delays
| Database | Query |
|----------|-------|
| PostgreSQL | `SELECT pg_sleep(10)` |
| MySQL | `SELECT sleep(10)` |
| Oracle | `SELECT 1 FROM dual WHERE 1=1 AND ROWNUM=1` *(use DNS OOB instead)* |
| Microsoft | `WAITFOR DELAY '0:0:10'` |

---

## Structure

```
portswigger-sqli-labs/
├── README.md
└── labs/
    ├── lab01-sqli-where-hidden-data.md
    ├── lab02-sqli-login-bypass.md
    └── ... (one file per lab)
```

---

## Disclaimer

These notes are for **educational purposes only**, documenting completed exercises on PortSwigger Web Security Academy — a legal, sandboxed training platform designed for learning web security.

All techniques described here should only ever be applied to systems you own or have explicit written permission to test.

---

## Resources

- [PortSwigger Web Security Academy — SQL Injection](https://portswigger.net/web-security/sql-injection)
- [PortSwigger SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
- [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)

---

*If this helped you, consider giving it a ⭐ — and feel free to open an issue if you spot anything wrong.*
