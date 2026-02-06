# Log File Analyzer - Setup Guide

## Quick Setup (10 minutes)

### Step 1: Prepare Log File

**Supported formats:**
- Apache Combined Log
- Nginx Access Log
- Exel Log  
- Linux auth.log (SSH)

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

---

### Step 3: Configure File Path

1. Open "Read Log File" node
2. Update path: `/tmp/access.log` (or your log location)
3. Save

---

### Step 4: Execute Analysis

1. Click "Execute Workflow"
2. Wait for completion (~30 seconds for 1000 lines)
3. Report generated at: `/tmp/threat-report.html`

---

### Step 5: View Report
```bash
# Download report:
docker cp n8n:/tmp/report_file.html ./report.html

# Open in browser:
open report_file.html
```

---

## Automation Setup

### Daily Scheduled Analysis

1. Replace "Manual Trigger" with "Schedule Trigger"
2. Configure:
   - Interval: Days
   - Time: 2:00 AM
   - Timezone: Your timezone
3. Activate workflow

### Email Reports

1. Add "Email Send" node as "Save Report"
2. Configure:
   - Attach: `/tmp/report_report.html`
   - Recipients: security-team@company.com
3. Test and activate

---

## Customization

### Adjust Threat Patterns

Edit "Detect Threats" node:
```javascript
const threatPatterns = [
  {
    name: 'Your Custom Pattern',
    regex: /your-regex/i,
    severity: 'HIGH',
    score: 25
  },
  // Add more...
];
```

### Change Scoring Thresholds

Edit "Aggregate Threats by IP" node:
```javascript
// Modify risk levels:
if (finalScore > 70) {  // was 80
  riskLevel = 'CRITICAL';
}
```

### Whitelist IPs

Add to "Aggregate by IP" node:
```javascript
const whitelist = ['192.168.1.100', '10.0.0.50'];

if (whitelist.includes(ip)) {
  return null;  // Skip whitelisted IPs
}
```

---

## Troubleshooting

**Large files causing timeouts:**
```bash
# Process in chunks:
head -n 10000 access.log > access_part1.log
# Analyze part1, then part2, etc.
```

**Memory errors:**
- Limit to 10,000 lines per execution
- Use streaming for very large logs

**No threats detected:**
- Verify log format matches parser regex
- Check sample data for comparison
- Review threat patterns

---

## Performance

| Log Size | Processing Time | Memory Usage |
|----------|----------------|--------------|
| 1,000 lines | ~5 seconds | <50MB |
| 10,000 lines | ~30 seconds | ~200MB |
| 100,000 lines | ~5 minutes | ~1GB |

**Recommendation:** Process logs in 10K line batches for optimal performance.
