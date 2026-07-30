# Data Analyst Setup Fundamentals
*Lessons every analyst should internalize — learned hands-on while setting up this project.*

---

## Project Case Study: Fixing a Real Encoding Mismatch (DataCo Supply Chain Dataset)

While loading the DataCo Smart Supply Chain CSV (~180,519 rows) into PostgreSQL on Windows, `\copy` failed with:

```
ERROR: character with byte sequence 0x81 in encoding "WIN1252" has no equivalent in encoding "UTF8"
```

**Diagnosis:** Byte `0x81` is undefined in Windows-1252 — meaning the file wasn't actually Windows-1252 at all, but likely ISO-8859-1 (Latin-1), a related encoding where every byte value 0–255 is defined.

**Fix, applied in PowerShell**, using .NET's encoding classes directly (PowerShell's built-in `-Encoding` flag doesn't expose Latin-1 by name):
```powershell
$content = [System.IO.File]::ReadAllText(
    "path\to\DataCoSupplyChainDataset.csv",
    [System.Text.Encoding]::GetEncoding(28591)   # 28591 = ISO-8859-1
)
[System.IO.File]::WriteAllText(
    "path\to\DataCo_utf8.csv",
    $content,
    [System.Text.UTF8Encoding]::new($false)      # false = no BOM
)
```

**Still failed after this fix — same byte, same line.** This revealed a *second*, independent issue: PostgreSQL's `client_encoding` session setting was still defaulting to Windows-1252, silently misreading the now-correctly-encoded file. Fixed with:
```sql
SET client_encoding TO 'UTF8';
```

**Result:** `\copy` succeeded, loading all 180,519 rows on the next attempt.

**Why this is worth documenting:** the same error message appeared twice, from two genuinely different root causes (file encoding, then client encoding) — a reminder that an unchanged error after a fix doesn't always mean the fix failed; it can mean there's a second, independent problem in a different layer of the stack. Methodically isolating *which* layer (file → terminal → client) was still wrong, rather than re-guessing at the file itself, is what resolved it.

---

## 1. Encoding: the invisible bug that breaks real-world data

**The concept:** A file is just a sequence of bytes. "Encoding" is the rulebook that turns those bytes into readable characters. If two systems disagree on the rulebook, you get garbled text or hard errors — even though nothing is visibly "wrong" with the file.

**Three separate places encoding can live — don't confuse them:**
| Layer | What it controls | How to check/set it |
|---|---|---|
| **File encoding** | How the file's bytes were saved to disk | `Get-Content -Encoding Default` (Windows-1252) vs Latin-1 vs UTF-8 |
| **Terminal display encoding** | How your terminal *displays* characters | `chcp` (Windows codepage) |
| **Client encoding** | How your SQL client (psql) *interprets* incoming bytes | `SET client_encoding TO 'UTF8';` |

**Why this matters for your career:** Real company data — especially exports from older systems, Excel, or Windows-based tools — is very often NOT clean UTF-8. Knowing how to diagnose "which layer is wrong" (file vs. terminal vs. client) instead of guessing is a genuinely senior-level troubleshooting skill.

**Rule of thumb:** If you see mangled characters OR a "byte sequence has no equivalent" error, don't panic — identify which of the three layers above is misconfigured, fix that one, and retest.

---

## 2. PATH: why your terminal "can't find" a program you know is installed

**The concept:** Your terminal only knows about programs sitting in specific folders it's told to check — this list of folders is called the `PATH`. If a program (like `psql`) isn't in one of those folders, the terminal says "not recognized" even though the program is very much installed.

**Two ways to fix it:**
1. **Quick/temporary:** call the program using its full file path (`& "C:\Program Files\PostgreSQL\18\bin\psql.exe"`)
2. **Permanent:** add the program's folder to your PATH once, via System Environment Variables — then it works from any terminal, forever

**Why this matters:** You'll hit "command not found" constantly across different tools (Python, git, node, database CLIs). It's never actually broken — it's just a missing PATH entry, 95% of the time.

---

## 3. GUI tools vs. command-line tools aren't interchangeable

**The concept:** Tools like SQLTools (in VS Code) or pgAdmin talk to Postgres using the standard SQL protocol. The `psql` command-line tool has *extra shortcuts* (called meta-commands, always starting with `\`) that only it understands — like `\d tablename` (describe table) or `\copy` (bulk-load a local file).

**Rule of thumb:** If a command starts with `\`, it only works inside `psql` in a terminal — never paste it into a GUI query editor. If you need the same info in a GUI tool, use real SQL instead (e.g., query `information_schema.columns` instead of `\d`).

---

## 4. Verify before you load — every time

**The habit:** Before loading any dataset, confirm:
- The table exists and is empty (`SELECT COUNT(*) FROM table;` → expect 0)
- The table structure matches your source file (`\d table` or `information_schema.columns`)
- You know the *expected* row count of your source (so you can catch a partial or failed load immediately)

**Why:** This 30-second habit is what separates "I ran it and hoped for the best" from "I confirmed my data is correct" — which matters enormously once you're making business decisions off these numbers.

---

## 5. `\copy` vs `COPY` — client-side vs server-side

- `\copy` (psql-only) reads a file from **your own computer** — use this for local files.
- `COPY` (plain SQL) reads a file from **the Postgres server's own disk** — only works if the file already lives on the server itself.

**Why it matters:** In real jobs, your database server is often remote (cloud-hosted). Knowing this distinction stops you from wasting time trying to `COPY` a file that Postgres literally cannot see.

---

## 6. `TRUNCATE` before reloading failed imports

If a bulk load fails partway through, some rows may have already been inserted. Always `TRUNCATE` (clear the table) before retrying — otherwise you risk silent duplicate rows that quietly corrupt your analysis later.

---

## 7. Treat PII columns with discipline, even in practice datasets

If a dataset has anything that *looks* like personal data (emails, names, passwords, addresses) — even synthetic/fake data on Kaggle — build the habit now of never using `SELECT *` in anything you'll screenshot, paste into a README, or push publicly. Always select named, relevant columns. This habit is what real-world data handling requires, and demonstrating it unprompted is a strong signal in a portfolio project.

---

## 8. Read error messages as clues, not roadblocks

Every error we hit today (`command not found`, `relation already exists`, `byte sequence has no equivalent`) told us *exactly* what was wrong once read carefully — the line number, the byte, the missing path. Confident analysts read the full error message before searching for a fix, rather than pattern-matching to a guess.
