# n8n Troubleshooting Guide

This document covers common issues and solutions when working with n8n.

## Common Issues

### Workflow Execution Issues

#### Workflow Not Triggering

**Causes:**
- Workflow not activated
- Webhook URL incorrect
- Schedule time not reached
- Trigger misconfigured

**Solutions:**
1. Check workflow is active (green toggle)
2. Verify webhook URL is production URL (not test)
3. Check schedule trigger configuration
4. Test trigger manually first

#### Timeout Errors

**Causes:**
- Long-running operations
- Slow external APIs
- Large data processing

**Solutions:**
```bash
# Increase timeout
EXECUTIONS_TIMEOUT=3600  # seconds
EXECUTIONS_TIMEOUT_MAX=7200
```

Or in workflow settings:
- Settings → Execution Timeout → Set higher value

#### Memory Errors

**Causes:**
- Processing large datasets
- Holding too much data in memory
- Memory-intensive operations

**Solutions:**
1. Process in batches using Loop Over Items
2. Reduce data volume early in workflow
3. Increase container memory
4. Use streaming for large files

---

### Node-Specific Issues

#### HTTP Request Fails

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid credentials | Check API key/token |
| 403 Forbidden | Insufficient permissions | Verify access rights |
| 404 Not Found | Wrong endpoint | Check URL |
| 429 Too Many Requests | Rate limited | Add Wait node, reduce frequency |
| 500 Server Error | External service issue | Retry with backoff |
| CORS Error | Browser block | Use server-side request |
| SSL Error | Certificate issue | Check SSL settings |

**Debug HTTP Requests:**
```javascript
// Enable full response
Response: Include Full Response
// Shows headers, status code, body
```

#### Webhook Not Receiving Data

**Checklist:**
1. ✓ Use **production** webhook URL (not test)
2. ✓ Workflow is **active**
3. ✓ Firewall allows incoming traffic
4. ✓ Correct HTTP method (GET/POST)
5. ✓ Authentication configured (if used)

**Test Webhook:**
```bash
curl -X POST https://your-n8n.com/webhook/path \
  -H "Content-Type: application/json" \
  -d '{"test": "data"}'
```

#### Schedule Trigger Not Running

**Causes:**
- Timezone mismatch
- Cron syntax error
- Instance restarted

**Solutions:**
1. Set correct timezone:
```bash
GENERIC_TIMEZONE=Asia/Bangkok
```

2. Verify cron syntax:
```javascript
// Examples
0 9 * * *      // 9 AM daily
0 */2 * * *    // Every 2 hours
0 0 * * 1-5    // Midnight weekdays
```

3. Check execution logs for schedule issues

---

### Expression Issues

#### Expression Returns Undefined

**Causes:**
- Field doesn't exist
- Wrong path to data
- Null value

**Solutions:**
```javascript
// Use optional chaining
{{ $json.user?.email }}

// Provide default
{{ $json.name || 'Unknown' }}
{{ $json.count ?? 0 }}

// Check data exists first
{{ $json.items?.length > 0 ? $json.items[0] : null }}
```

#### Type Errors

**Causes:**
- String vs number mismatch
- Object vs string operations

**Solutions:**
```javascript
// Convert types
{{ parseInt($json.stringNumber) }}
{{ String($json.number) }}
{{ JSON.stringify($json.object) }}
{{ JSON.parse($json.jsonString) }}
```

#### Date/Time Issues

**Causes:**
- Timezone differences
- Format mismatch
- Invalid date string

**Solutions:**
```javascript
// Parse date
{{ DateTime.fromISO($json.date) }}
{{ DateTime.fromFormat($json.date, 'dd/MM/yyyy') }}

// Format date
{{ $now.toFormat('yyyy-MM-dd HH:mm:ss') }}

// Timezone conversion
{{ $now.setZone('Asia/Bangkok').toISO() }}
```

---

### Connection Issues

#### Cannot Connect to Database

**Checklist:**
1. ✓ Database server is running
2. ✓ Credentials are correct
3. ✓ Network/firewall allows connection
4. ✓ SSL configuration matches
5. ✓ Database exists

**PostgreSQL Connection:**
```bash
# Test connection
psql -h hostname -U username -d database

# Common issues
- "connection refused" → Check host/port
- "authentication failed" → Check credentials
- "database does not exist" → Create database
```

#### API Connection Timeout

**Causes:**
- Slow API response
- Network issues
- Firewall blocking

**Solutions:**
1. Increase timeout in HTTP Request node
2. Check network connectivity
3. Verify API is accessible from server

---

### Credential Issues

#### OAuth2 Token Expired

**Symptoms:**
- 401 errors after initial success
- Workflow worked before, now fails

**Solutions:**
1. Re-authenticate the credential
2. Check token refresh is configured
3. Verify OAuth app hasn't been revoked

#### Credentials Not Found

**Causes:**
- Credential deleted
- Sharing permissions changed
- Import without credentials

**Solutions:**
1. Recreate the credential
2. Check credential sharing settings
3. Import credentials separately

---

### Docker/Hosting Issues

#### Container Won't Start

**Check logs:**
```bash
docker logs n8n
```

**Common causes:**
- Port already in use
- Volume permissions
- Missing environment variables

**Solutions:**
```bash
# Check ports
lsof -i :5678

# Fix permissions
chown -R 1000:1000 /path/to/n8n/data

# Required env vars
N8N_ENCRYPTION_KEY=your-32-char-key
```

#### Data Not Persisting

**Cause:** Volume not properly mounted

**Solution:**
```yaml
# docker-compose.yml
volumes:
  - ./n8n-data:/home/node/.n8n
```

```bash
# Verify permissions
chmod 755 ./n8n-data
```

#### Webhooks Not Accessible

**Causes:**
- Wrong WEBHOOK_URL
- Reverse proxy misconfigured
- SSL issues

**Solutions:**
```bash
# Set correct URL
WEBHOOK_URL=https://n8n.yourdomain.com/

# Check reverse proxy headers
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

---

## Debugging Techniques

### View Execution Data

1. Open workflow
2. Click "Executions" tab
3. Click failed execution
4. Inspect data at each node

### Add Debug Nodes

```
Input → Set Node (capture data) → Problem Node → Continue
```

**Set Node Configuration:**
- Keep Only Set: false
- Add fields to inspect data

### Enable Debug Logging

```bash
N8N_LOG_LEVEL=debug
N8N_LOG_OUTPUT=console
```

### Test Expressions

Use the expression editor:
1. Click expression field
2. Open expression editor
3. See live preview of result

### Check Pinned Data

Pinned data might be outdated:
1. Check for pin icons on nodes
2. Unpin to use live data
3. Re-pin after testing

---

## Performance Optimization

### Slow Workflow Execution

**Causes:**
- Too many items processed
- Sequential instead of parallel
- Unnecessary nodes

**Solutions:**
1. Filter data early
2. Process in batches
3. Use parallel execution where possible
4. Remove unused nodes

### High Memory Usage

**Solutions:**
1. Process large files with streaming
2. Use pagination for API calls
3. Clear unnecessary data between nodes
4. Split into sub-workflows

### API Rate Limiting

**Solutions:**
```
Items → Loop Over Items → Wait (1s) → API Call
           (batch: 5)
```

---

## Getting Help

### Resources

1. **n8n Community**: https://community.n8n.io/
2. **GitHub Issues**: https://github.com/n8n-io/n8n/issues
3. **Discord**: https://discord.gg/n8n
4. **Documentation**: https://docs.n8n.io/

### When Reporting Issues

Include:
- n8n version
- Node types involved
- Error message (full text)
- Steps to reproduce
- Expected vs actual behavior
- Relevant logs

### Log Location

```bash
# Docker
docker logs n8n

# npm install
Check terminal output

# File logging
cat /path/to/n8n/logs/n8n.log
```
