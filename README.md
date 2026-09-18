# sqltune — a real-time SQL Server monitor (free)

![.NET 11](https://img.shields.io/badge/.NET-11.0-512BD4)
![C# 15](https://img.shields.io/badge/C%23-15-239120)
![Blazor Hybrid](https://img.shields.io/badge/Blazor-Hybrid-5C2D91)
![SQL Server 2019+](https://img.shields.io/badge/SQL%20Server-2019%2B-CC2927)
![Windows x64](https://img.shields.io/badge/Windows-x64-0078D6)

**[Download the latest release](https://github.com/Doni-Kim/sqltune-release/releases/latest)** ·
[한국어 설명](README.ko.md) · [Manual (Korean, HTML)](sqltune.html)

sqltune is a desktop monitor for SQL Server. One window, one executable, nothing to install on the server —
everything is read from DMVs, Query Store and the system catalog. It only reads: the one thing that changes
the server is killing a session, and only when you confirm it.
Free to use, no strings attached.

## Screenshots

| Live dashboard | Top SQL (Query Store) |
|---|---|
| ![Dashboard](screenshots/dashboard.jpg) | ![Top SQL](screenshots/top-sql.jpg) |

| Lock chains | Session detail |
|---|---|
| ![Locks](screenshots/locks.jpg) | ![Session detail](screenshots/session-detail.jpg) |

| Health Check | Deadlocks |
|---|---|
| ![Health Check](screenshots/health-check.jpg) | ![Deadlocks](screenshots/deadlocks.jpg) |

| Alerts | Index diagnostics |
|---|---|
| ![Alerts](screenshots/alerts.jpg) | ![Indexes](screenshots/indexes.jpg) |

The screenshots show a throwaway test server (Docker) with the AdventureWorks sample database.

## Install

- Unzip, keep the folder together, and run `sqltune.exe` (single-file publish).
- No .NET install needed — the runtime is inside the executable.
- The only thing to set up is `sqltune.json` next to the executable: either edit it (see below), or just run
  `sqltune.exe` — when the bundled values do not connect, a connection dialog opens with those values filled in.
  Once the connection has succeeded, what you typed is saved back (the password is stored encrypted).

## What it does

- **Live dashboard** — buffer cache hit ratio, SQL Server CPU, PLE, memory, memory grants pending, tempdb, batches / transactions /
  compiles / recompiles / page splits / full scans / lock waits per second, disk throughput and latency, top waits, trend graphs, and the session list.
  Rates always cover the last sample, not the time since the server started.
  SQL Server CPU is a share of the CPUs SQL Server may use: when affinity or an edition limit gives it only some of the
  host's CPUs (say 4 of 8), 100% means all of those are busy, and the gauge also shows "4 of 8 CPUs" and the share of the whole host.
- **Lock chains that find the real root** — including sessions that left a transaction open and went idle
  (`idle-tx`), the usual root blocker that request-only views miss. `F5` shows blockers together with the
  sessions they block.
- **Session detail** (`Enter`) — full statement and batch, the live execution plan with actual row counts so far
  (or the cached plan), locks held. `Ctrl+K` kills the session (it checks the login time first, so a reused
  session number is never killed by mistake); `Ctrl+X` exports the session to Excel — plan, locks, and the size,
  indexes and statistics of the tables in its plan.
- **Panels** — Server (`I`), Connections (`C`), Locks (`A`), Disk (`D`), Waits (`W`),
  Deadlocks (`E`, from `system_health`, the latest one spelled out), Statistics (`U`, with an UPDATE STATISTICS
  script), Indexes (`X`: missing with CREATE, unused with DROP, duplicate and overlapping, fragmentation on demand).
- **Top SQL** (`T`) — from Query Store: by duration, CPU, reads, executions or a combined score, over 1 h / 24 h /
  7 days; statements with several plans; regressed queries (at least twice as slow as the week before);
  text search; plan history per query with a force-plan script (shown, never run).
- **Health Check** (`G`) — 10 categories (server configuration, performance, database settings, indexes,
  statistics, tempdb, files, backups, Agent jobs, log) with a score, a grade and what to fix first; Excel export.
  MAXDOP is judged per NUMA node, PLE by buffer pool size, backups by the server's own clock.
- **Alerts** — 12 rules: CPU (of the CPUs SQL Server may use), lock chains, blocked requests, idle in transaction, long requests, buffer cache hit,
  page life expectancy, memory grants pending, tempdb usage, disk read / write latency, deadlocks —
  with your own thresholds.
- **History** — press `L` to log every sample into a local SQLite file, then `H` to look back.
  - Ranges: 1 hour / 6 hours / 24 hours / 1 week / 1 month / all. 20 metrics.
  - Old rows are trimmed automatically (30 days of metrics, 7 days of sessions by default; configurable).
- **Excel export** — built on ClosedXML, so the `.xlsx` is written even without Excel installed.
- 12 themes (6 light, 6 dark). Reconnects by itself when the connection drops.
- `Ctrl+B` switches the database used by the database-level panels. Press `F1` for the keyboard shortcuts.

The bundled `sqltune.html` is the full manual with screenshots (in Korean).

## Requirements and limits

- **Windows only.** The UI is web-based (Blazor Hybrid), so it does not run standalone on Linux or macOS.
- **SQL Server 2019 or later** — on-premises, VM or Managed Instance. Developed and checked against SQL Server 2025.
- **Azure SQL Database is not supported** — sqltune says so in the connection dialog.
- **Top SQL needs Query Store** in the database you look at (on by default for new databases since SQL Server 2022).
  When it is off, the panel shows the statement that turns it on.
- The code is obfuscated with ConfuserEx — a free tool, so do not expect strong protection.

## A dedicated monitoring account

Checked on SQL Server 2025: with these grants every panel and the Health Check show the same as with `sa`.

```sql
USE master;
CREATE LOGIN sqltune WITH PASSWORD = '...';
GRANT VIEW SERVER STATE    TO sqltune;   -- dashboard, sessions, waits, locks, deadlocks, plans
GRANT VIEW ANY DEFINITION  TO sqltune;   -- Disk: files and volumes of every database
GRANT ALTER ANY CONNECTION TO sqltune;   -- optional: Ctrl+K (kill)

-- In each database you look at with Ctrl+B (Top SQL, Indexes, Statistics, Health Check)
USE YourDatabase;
CREATE USER sqltune FOR LOGIN sqltune;
GRANT VIEW DATABASE STATE TO sqltune;           -- Query Store, index usage, log space
GRANT VIEW DEFINITION     TO sqltune;           -- index and statistics metadata
ALTER ROLE db_datareader ADD MEMBER sqltune;    -- optional: statistics — see below

-- Health Check: backups and Agent jobs
USE msdb;
CREATE USER sqltune FOR LOGIN sqltune;
GRANT SELECT  ON dbo.backupset      TO sqltune;
GRANT SELECT  ON dbo.sysjobs        TO sqltune;
GRANT SELECT  ON dbo.sysjobhistory  TO sqltune;
GRANT EXECUTE ON dbo.agent_datetime TO sqltune;
```

- `sys.dm_db_stats_properties` only returns statistics of tables the login may read, so without
  `db_datareader` the Statistics panel and the Statistics category of the Health Check come back empty.
  Everything else works without it.
- Without `VIEW ANY DEFINITION` the Disk panel shows no files or volumes.
- `SQLAgentReaderRole` is not enough for the Agent check — it covers the Agent procedures, not the job tables.
- When a grant is missing, sqltune says which one on the panel instead of showing an empty list.

## Blank window? (WebView2)

If the window opens but stays blank, the WebView2 runtime is missing.

- **Windows 11** — built into the OS, always present.
- **Windows 10** — shipped through Windows Update since 2021, so it is there on most machines.
  A PC that has not been updated in a long time, or a special edition such as LTSC, may not have it.
- **Windows Server (2016/2019/2022)** — often not included; install it separately.

Install Microsoft's "Evergreen Standalone Installer"
(`MicrosoftEdgeWebView2RuntimeInstallerX64.exe`) from
https://developer.microsoft.com/microsoft-edge/webview2/

## If something breaks

Errors are written to `sqltune.log` next to the executable (the file only appears when something went wrong).

- **Bugs and questions** — open an [issue](https://github.com/Doni-Kim/sqltune-release/issues).
  Please do not attach the log there: it holds no passwords, but it can contain server addresses and SQL text.
- **The log file**, or anything you would rather not post in public — mail it to **doniikim@gmail.com**.

## Built with

- .NET 11.0 (x64), C# 15, Blazor Hybrid
- Microsoft.Data.SqlClient · Microsoft.Data.Sqlite · ClosedXML · Microsoft.Web.WebView2 · Microsoft.AspNetCore.Components.WebView.WindowsForms
- Copyright notices and license texts of these bundled components: [THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt) (also inside the zip)

## sqltune.json

The zip ships a small `sqltune.json` with default values (`127.0.0.1:1433`, `sa`). Edit it, or let the
connection dialog fill it in:

```json
{
  "databases": [
    {
      "userId": "sa",
      "password": "<password>",
      "server": "127.0.0.1",
      "port": "1433",
      "database": "master",
      "encrypt": "mandatory",
      "trustServerCertificate": true
    }
  ],
  "interval": 5
}
```

- Write `password` in plain text — it is encrypted on the first run and stored back as `ENC1:...`.
- A named instance goes into `server` as `HOST\\INSTANCE`, with `port` empty or `1433` (found through SQL Browser).
- `database` is where the database-level panels start; `Ctrl+B` switches it. The session list is always server-wide.
- `encrypt`: `mandatory` (default) / `optional` / `strict`. `trustServerCertificate` accepts a self-signed certificate.
- Sections such as `alerts` and `logRetention` are optional. `sqltune_sample_en.json` in the zip
  documents every setting.

## Terms

Free to use, at work or at home. Please do not redistribute the binary or reverse-engineer it.
The source is not published.

sqltune bundles third-party components, including Microsoft components (such as the SQL Server network library
Microsoft.Data.SqlClient.SNI). Each is provided under its own license terms, reproduced in full in
[THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt) (also inside the zip); by using sqltune you also accept those terms.

## Contact

DBMS Works — **doniikim@gmail.com**

Also available for Oracle → PostgreSQL / MySQL migration and database performance tuning work.
