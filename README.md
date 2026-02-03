# Log File Analyzer

> Automated security log analysis system using n8n to detect threats, brute force attacks, SQL injection attempts, and anomalies in Apache/Nginx/syslog files

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![n8n](https://img.shields.io/badge/n8n-workflow-orange)](https://n8n.io)

![Workflow Screenshot](media/workflow-screenshot.png)

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Demo](#demo)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Threat Detection](#threat-detection)
- [Sample Data](#sample-data)
- [Troubleshooting](#troubleshooting)
- [License](#license)

---

## Overview

**Problem:** System administrators manually review thousands of log entries daily to detect security threats. Critical incidents (brute force attacks, SQL injection, privilege escalation) can be missed in the noise of normal traffic.

**Solution:** This n8n workflow automatically parses Apache/Nginx access logs and syslog files, detects suspicious patterns using regex and threat intelligence, calculates threat scores per IP, and generates actionable HTML security reports.

**Technology:**
- n8n (workflow automation)
- Regex pattern matching for threat detection
- IP-based threat scoring and aggregation
- HTML report generation
- Email/Slack alerting

**Use Cases:**
- Web server security monitoring (Apache, Nginx)
- SSH brute force detection (auth.log)
- SQL injection attempt detection
- DDoS pattern recognition
- Automated incident response

---

## Features

- **Multi-log format support** - Apache, Nginx, syslog, auth.log
- **Real-time threat detection** - 10+ attack pattern types
- **IP reputation scoring** - Aggregate threats by source IP
- **Automated reporting** - HTML reports with charts and tables
- **Rate limiting detection** - Identify brute force attempts (>100 req/min)
- **False positive filtering** - Whitelist known IPs
- **Email/Slack alerts** - Instant notifications for critical threats
- **Historical tracking** - CSV logs for trend analysis

---

## Demo

### Audio Case Study (Coming Soon)
Full walkthrough available after workflow build completion.

### Visual Demo
![Demo GIF](media/demo.gif)

---

## Prerequisites

**Required:**
- **n8n instance** (self-hosted or cloud)
- **Server log files** (Apache access.log, auth.log, or similar)

**Optional:**
- **Email/Slack** for alerts
- **Web server access** for automated log retrieval

---

## Installation

### Quick Start: Import Workflow (5 minutes)

1. **Download workflow:**
   - [Releases](https://github.com/Dessybabybaby/log-file-analyzer/releases)
   - Download `log-analyzer-workflow.json`

2. **Import to n8n:**
   - Workflows → Import from File → Select JSON

3. **Upload sample log file:**
   - Place log file in `/tmp/access.log` or configure path

4. **Execute workflow:**
   - Click "Execute Workflow"
   - Review generated report

5. **Schedule automated analysis:**
   - Replace Manual Trigger with Schedule Trigger (daily/weekly)

---

## Usage

### Analyzing Log Files

**Supported Log Formats:**

**Apache Combined Log:**
```
192.168.1.100 - - [19/Jan/2026:15:30:45 +0000] "GET /admin HTTP/1.1" 404 512 "-" "Mozilla/5.0"
```

**Nginx Access Log:**
```
192.168.1.100 - - [19/Jan/2026:15:30:45 +0000] "POST /login HTTP/1.1" 401 128
```

**Linux auth.log (SSH):**
```
Jan 19 15:30:45 server sshd[12345]: Failed password for root from 192.168.1.100 port 22 ssh2
```

### Running Analysis

**Manual:**
1. Upload log file to n8n server
2. Update file path in "Read Log File" node
3. Execute workflow
4. Download HTML report

**Automated:**
1. Replace Manual Trigger with Schedule Trigger
2. Configure log file path (e.g., `/var/log/apache2/access.log`)
3. Activate workflow
4. Reports generated daily

### Reading Reports

**HTML Report Sections:**

1. **Executive Summary**
   - Total requests analyzed
   - Threats detected (count + %)
   - Top threat types
   - Critical IPs flagged

2. **Top Threat IPs Table**
   - IP address
   - Request count
   - Threat score (0-100)
   - Attack types detected
   - Recommended action (Block/Monitor/Whitelist)

3. **Threat Timeline**
   - Hourly distribution of attacks
   - Spike detection

4. **Attack Pattern Breakdown**
   - SQL injection attempts: X
   - Path traversal: X
   - Brute force: X
   - Scanner activity: X

---

## Threat Detection

### Detection Patterns

| Attack Type | Pattern | Example | Severity |
|-------------|---------|---------|----------|
| **SQL Injection** | `union select`, `' or 1=1`, `drop table` | `/search?q=' OR 1=1--` | CRITICAL |
| **Path Traversal** | `../`, `..%2f`, `....//` | `/files/../../../etc/passwd` | HIGH |
| **XSS Attempt** | `<script>`, `javascript:`, `onerror=` | `/comment?text=<script>alert(1)</script>` | HIGH |
| **Admin Panel Probing** | `/admin`, `/phpmyadmin`, `/wp-admin` | `GET /admin HTTP/1.1` | MEDIUM |
| **Brute Force** | >100 requests/min from single IP | 150 requests in 60 seconds | HIGH |
| **Scanner Activity** | `/config.php`, `/.env`, `/backup.sql` | `GET /.env HTTP/1.1` | MEDIUM |
| **Shell Upload** | `cmd.php`, `shell.asp`, `.jsp` | `POST /uploads/cmd.php` | CRITICAL |
| **404 Scanning** | >20 sequential 404s | 25 consecutive not-found errors | LOW |
| **Auth Failures** | Failed password, Invalid user | `Failed password for root` | MEDIUM |
| **Rate Limiting** | Excessive requests (>500/min) | DDoS pattern | HIGH |

### Threat Scoring Algorithm
```
Base Score = 0

For each request:
  IF SQL Injection detected: +30
  IF Path Traversal detected: +25
  IF XSS detected: +20
  IF Admin probing: +10
  IF Scanner activity: +15
  IF Shell upload: +35
  IF Auth failure: +10
  
Rate multipliers:
  IF requests > 500/min: Score × 2
  IF requests > 100/min: Score × 1.5
  IF sequential 404s > 20: +20

Final Threat Score = Capped at 100
```

**Score Interpretation:**
- **0-20:** Low risk (normal traffic)
- **21-50:** Medium risk (suspicious, monitor)
- **51-80:** High risk (likely attacker, block recommended)
- **81-100:** Critical (active attack, immediate action)

---

## Sample Data

### Test Log Files

**`sample-data/access.log`** - Apache access log with injected threats

**`sample-data/auth.log`** - SSH brute force attempts

**`sample-data/threat-report-sample.html`** - Expected output report

### Running Test

1. Download sample log: `sample-data/access.log`
2. Place in `/tmp/access.log`
3. Execute workflow
4. Compare output with `sample-data/threat-report-sample.html`

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **File not found** | Verify log file path in "Read Log File" node; check permissions |
| **No threats detected** | Verify log format matches parser; check sample data for comparison |
| **Memory errors (large files)** | Process logs in batches (limit to 10,000 lines per execution) |
| **Invalid regex errors** | Check pattern syntax in "Detect Threats" node |
| **Report not generated** | Verify HTML template syntax; check execution logs |

**Processing Large Logs:**
```bash
# Split large log into smaller files
split -l 10000 access.log access_part_

# Process each part separately
for file in access_part_*; do
  # Update n8n "Read Log File" path to $file
  # Execute workflow
done
```

---

## License

MIT License - see [LICENSE](LICENSE)

---

## Acknowledgments

- Inspired by [UnixGuy](https://youtube.com/@UnixGuy) - Sysadmin and log analysis tutorials
- Built with [n8n.io](https://n8n.io)
- Threat patterns based on OWASP Top 10

---

## Contact

**Creator:** [Achusi Desmond]
- Portfolio: https://Dessybabybaby.github.io/portfolio-site
- GitHub: [@Dessybabybay](https://github.com/Dessybabybaby)

---

**If this tool helps secure your infrastructure, please star the repo!**
```
