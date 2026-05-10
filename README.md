# NetDB-IDS: Network + Database Intrusion Detection System

NetDB-IDS is a cybersecurity  project that detects attacks by correlating network telemetry with database query behavior. It is designed to show practical blue-team engineering, network security awareness, database internals knowledge, and applied machine learning in one runnable demo.

The project runs locally with free tools and Python libraries. It simulates Zeek-style flow logs, Suricata-style IDS alerts, suspicious SQL activity, anomaly scoring, cross-layer correlation, and a live SOC dashboard.

## Why This Project

Most beginner cybersecurity projects stop at a port scanner, password checker, or static dashboard. NetDB-IDS is different because it connects multiple layers of an attack:

- A network scan touches database-facing ports.
- A web or DB session sends suspicious SQL.
- The database proxy extracts query structure using SQL AST parsing.
- The correlator joins network and database events by source IP and time window.
- The alert engine classifies the activity as reconnaissance, SQL injection, privilege escalation, or data exfiltration.

This is the same kind of thinking used in SIEM, SOC, and detection engineering work.

## Features

- TCP SQL proxy that intercepts and audits every query
- SQLite target database with realistic users, products, orders, and inventory tables
- SQL AST feature extraction with `sqlglot`
- Sensitive table, wildcard, UNION, comment, DROP, ALTER, and admin-role detection
- Zeek-like network flow simulation
- Suricata-like IDS signature simulation
- Isolation Forest anomaly scoring with `scikit-learn`
- Cross-layer DB/network joint risk score
- DuckDB audit storage
- Streamlit SOC dashboard
- Automated red-team scenarios
- Automated test suite and benchmark report

## Architecture

```text
Attack Simulator
  |-- baseline traffic
  |-- recon / SQLi / privesc / exfiltration
  |
  +--> Network telemetry queue
  |      Zeek-like flows + Suricata-like IDS alerts
  |
  +--> Mock DB TCP proxy
         SQL execution + SQL AST feature extraction
         |
         +--> DuckDB audit log
         +--> DB event queue

Correlation Engine
  |-- reads DB events
  |-- reads network events
  |-- joins on source IP + timestamp window
  |-- computes DB score, network score, joint score
  |-- classifies threat stage
  |
  +--> DuckDB alert store

Streamlit Dashboard
  |-- event timeline
  |-- threat distribution
  |-- network telemetry
  |-- correlated alert feed
  |-- DB event log
```

## Tech Stack

| Layer | Tool |
|---|---|
| Language | Python |
| DB target | SQLite |
| Audit storage | DuckDB |
| SQL parsing | sqlglot |
| ML detection | scikit-learn Isolation Forest |
| Dashboard | Streamlit + Plotly |
| Network simulation | Zeek-style flows, Suricata-style signatures |
| Attack simulation | nmap/sqlmap-inspired native scripts |

## Project Structure

```text
netdb-ids-native/
  run.py                         # orchestration, tests, benchmark
  run.ps1                        # Windows-friendly launcher
  requirements.txt               # Python dependencies
  proxy/
    mock_db_server.py             # TCP SQL proxy + DB audit logger
  attack_sim/
    attack_sim_native.py          # baseline and attack traffic generator
  correlator/
    correlator_native.py          # ML scoring + DB/network correlation
    eval.py                       # precision/recall/F1 benchmark
  dashboard/
    dashboard.py                  # Streamlit SOC dashboard
  data/                           # generated logs and DuckDB files
```

## Quick Start

On Windows PowerShell:

```powershell
cd netdb-ids-native
.\run.ps1 demo
```

Open the dashboard:

```text
http://localhost:8501
```

For a one-shot verification run that starts services, runs attacks, runs tests, prints metrics, and stops:

```powershell
.\run.ps1 demo-once
```

On macOS/Linux, use Python directly:

```bash
cd netdb-ids-native
python -m pip install -r requirements.txt
python run.py demo
```

## Commands

```powershell
.\run.ps1 demo       # full demo, keeps dashboard running
.\run.ps1 demo-once  # full demo + tests, then stops services
.\run.ps1 start      # start DB proxy, correlator, and dashboard
.\run.ps1 attack     # run baseline and attack scenarios
.\run.ps1 test       # run automated tests
.\run.ps1 eval       # print benchmark report
.\run.ps1 clean      # wipe generated data
```

## Attack Scenarios

| Scenario | Simulated Behavior | Detection Signals |
|---|---|---|
| Reconnaissance | Port sweep and schema enumeration | Unique destination ports, SQLite schema queries |
| SQL injection | Auth bypass, UNION SELECT, SQL comments | SQLi patterns, UNION AST node, IDS signature |
| Privilege escalation | Admin role changes, DROP, ALTER TABLE | Privilege operations, schema changes, DB admin signature |
| Data exfiltration | Bulk SELECT, sensitive columns, joins | Wildcard reads, sensitive tables, large outbound flow |

## Example Demo Output

Recent full verification:

```text
Tests passed: 23/23
DB events: 327
Network events: 84
Alerts generated: 21
Detected: SQLi, privilege escalation, exfiltration
Dashboard: http://localhost:8501
```

Example benchmark output:

```text
Type              Precision  Recall  F1
SQLi              1.000      1.000   1.000
Privilege Esc     1.000      1.000   1.000
Exfiltration      detected with correlated network + DB context
```

## Detection Logic

The project combines rule-based and ML-based detection.

Database features include:

- SQL length
- statement type
- tables accessed
- sensitive table access
- wildcard SELECT
- WHERE/LIMIT presence
- JOIN count
- UNION/subquery usage
- comments
- DROP/ALTER/admin-role operations
- SQL injection pattern count

Network features include:

- source IP
- destination port
- service
- bytes in/out
- IDS signature
- severity
- simulated tool source
- timestamp

The correlator computes:

```text
joint_score = 1 - ((1 - db_score) * (1 - network_score))
```

This means a suspicious SQL query becomes more severe when it appears near suspicious network activity from the same source.

## Dashboard

The SOC dashboard shows:

- Total DB events
- Total alerts
- Network events
- High-risk alerts
- SQLi hit count
- Event timeline
- Network destination port activity
- Threat distribution
- Correlated alert feed
- Raw DB event log

Recommended screenshot path for GitHub:

```text
docs/screenshots/dashboard.png
```

Add a screenshot later with:

```markdown
![NetDB-IDS dashboard](docs/screenshots/dashboard.png)
```

## Interview Talking Points

Use this short explanation:

> I built a cross-layer intrusion detection system that correlates network-level telemetry with database query behavior. The network layer models Zeek and Suricata outputs, while the database proxy extracts SQL AST features with sqlglot. The correlator joins events by source IP and time window, computes a joint anomaly score, and classifies attacks such as SQL injection, privilege escalation, and data exfiltration.

Good files to explain during an interview:

- `proxy/mock_db_server.py`: query interception and SQL feature extraction
- `correlator/correlator_native.py`: network/database correlation and scoring
- `attack_sim/attack_sim_native.py`: red-team simulation scenarios
- `dashboard/dashboard.py`: SOC dashboard
- `correlator/eval.py`: benchmark report

## What This Demonstrates

- Network security fundamentals
- IDS/SIEM-style event correlation
- Database auditing and query analysis
- Detection engineering
- Python backend engineering
- ML anomaly detection
- Dashboarding and security visualization
- End-to-end testing

## Ethical Use

This project is for defensive security education and local lab demonstration only. The attack payloads are simulated against the included local mock database service. Do not run security tools or attack simulations against systems you do not own or have explicit permission to test.

## Future Improvements

- Add real Zeek log ingestion
- Add real Suricata `eve.json` ingestion
- Add PostgreSQL `pg_audit` mode
- Add Docker Compose lab with DVWA and PostgreSQL
- Add MITRE ATT&CK mapping
- Add alert acknowledgement workflow
- Add PDF demo report generation

## License

MIT License. Use it, modify it, and improve it for defensive security learning.
