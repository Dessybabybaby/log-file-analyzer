# Log File Analyzer - Setup Guide

## Quick Setup (10 minutes)

### Step 1: Prepare Log File

**Supported formats:**
- Apache Combined Log
- Nginx Access Log
- Linux `auth.log` (SSH)
- Text-based firewall logs

**Upload to n8n server:**
```bash
# Docker:
docker cp /path/to/access.log n8n:/tmp/access.log

# Local:
cp /var/log/apache2/access.log /tmp/access.log
```

---

### Step 2: Import Workflow

1. Download: [log-analyzer-workflow.json](../workflows/log-analyzer-workflow.json)
2. n8n → Import from File
3. Configure credentials:
- AbuseIPDB → API Key
- GitHub → Personal Access Token

---

### Step 3: Configure File Path

1. Open "Read Log File" node
2. URL: paste GitHub Raw URL OR local file path
3. Gmail node: change recipient to SOC team email
4. Save

---

### Step 4: Execute Analysis

1. Parses log lines
2. Aggregates activity by IP
3. Applies behavioral kill-chain scoring
4. Checks 24-hour cache to avoid duplicate alerts

---

### Step 5: View Report
1. Email: “SOC Alert”
2. Attachment: SOC_Action_Report.html (summary + actions)
3. Tickets: auto-created issues in soc-incident repository

---

## Automation Setup

### Daily Scheduled Analysis

1. Replace "Manual Trigger" with "Schedule Trigger"
2. Configure:
   - Interval: Hour
   - Time: 2:00 AM
   - Timezone: Your timezone
3. Activate workflow

### Alert Deduplication (Fatigue Control)

1. Default TTL: 24 hours
2.Reset for testing:
   ```javascript
   const DEBUG_MODE = true;
   ```

---

## Customization

### Behavioral Scoring

Edit "Aggregate Threats by IP" node:
```javascript
// Behavioral weights
if (entry.behaviors.has('recon')) score += 10;
if (entry.behaviors.has('discovery')) score += 15;
if (entry.behaviors.has('exploit')) score += 40;
if (entry.behaviors.has('compromise')) score += 60;

// Multi-stage bonus
if (entry.behaviors.size >= 3) score += 20;
```

### Whitelist Internal Traffic

```javascript
const whitelist = ['127.0.0.1', '192.168.1.50'];
if (whitelist.includes(ip)) return;
```

---

## Troubleshooting

**Large files causing timeouts:**
```bash
split -l 20000 access.log access_part_
```

**Increase n8n memory:**
```bash
NODE_OPTIONS=--max-old-space-size=4096
```

**Incident tickets not created:**
- Confirm Decision Engine reaches `socAction === 'INVESTIGATE'`
- Verify GitHub token repository permissions

**No threats detected:**
- Ensure log format matches regex in Detect Threats node
- Test with:
  UNION SELECT
  ../../etc/passwd

---

## Performance

| Log Size | Processing Time | Strategy |
|----------|----------------|--------------|
| 1,000 lines | ~3 seconds | Real-time behavioral |
| 10,000 lines | ~20 seconds | Batch behavioral |
| 100,000 lines | ~3 minutes | Segmented batch |

**Recommendation:** In high-volume environments, run every 30 minutes to keep batches small and fast.
