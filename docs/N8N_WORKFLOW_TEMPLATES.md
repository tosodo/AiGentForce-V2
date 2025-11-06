# n8n Workflow Templates

Exportable workflow templates for common AiGentForce-V2 automation scenarios. Each template can be imported directly into n8n.

## Quick Start

1. Copy the workflow JSON from the respective section
2. In n8n: **Workflows** → **Import** → Paste JSON
3. Configure credentials (API keys, database connections)
4. Customize for your environment
5. Deploy and test

---

## Template 1: Document Analysis Pipeline

**Use Case:** Ingest documents from storage, parse with LangChain, analyze with OpenAI, store results

**Prerequisites:**
- OpenAI API key configured
- LangChain integration
- MySQL database
- File storage (S3, Google Drive, etc.)

**Nodes Overview:**
```
Webhook → Download File → LangChain Parser → OpenAI Analysis → MySQL Store → Slack Notify → Error Handler
```

**Setup Instructions:**

1. **Webhook Node**: Configure to accept POST requests
   - Expected payload: `{ documentId, fileName, fileUrl }`
   - Set authentication method (API Key recommended)

2. **HTTP Download Node**: Fetch file from URL
   - Use file URL from webhook payload
   - Add timeout: 60000ms
   - Error handling: Retry 3 times with exponential backoff

3. **LangChain Node**: Parse document
   - Input: Downloaded file
   - Operations: Extract text, split into chunks, compute embeddings
   - Chunk size: 1000 characters
   - Overlap: 200 characters

4. **OpenAI Node**: Generate summary and insights
   - Model: `gpt-4-turbo`
   - Prompt: "Summarize this document and extract key insights"
   - Temperature: 0.3 (deterministic)

5. **MySQL Node**: Store results
   ```sql
   INSERT INTO documents (id, filename, summary, embeddings, status, created_at)
   VALUES (?, ?, ?, ?, 'completed', NOW())
   ```
   - Parameters: documentId, fileName, summary from OpenAI, embeddings from LangChain

6. **Slack Node**: Notify team of completion
   - Channel: #document-processing
   - Message: "Document {{documentId}} analyzed successfully"

7. **Error Handler**: Catch and log failures
   - Log to MySQL errors table
   - Send alert to #alerts channel

**Estimated Execution Time:** 30-60 seconds per document

---

## Template 2: User Onboarding Automation

**Use Case:** Automate new user setup including license provisioning, service activation, notifications, and follow-ups

**Prerequisites:**
- MySQL database for user records
- Email service (Gmail/Outlook)
- Slack workspace
- External service APIs (SSO, tools)

**Nodes Overview:**
```
Webhook → License Check → Create User Record → Provision Services → Send Welcome Email → 
→ Add to Teams → Slack Announcement → Schedule Follow-ups → Audit Log
```

**Setup Instructions:**

1. **Webhook Node**: Receive new user signup
   - Expected payload: `{ userId, email, firstName, lastName, role, department }`

2. **License Check Node**: Verify available licenses
   ```sql
   SELECT COUNT(*) as used FROM users WHERE status = 'active'
   AND total_licenses FROM org_settings WHERE id = ?
   ```
   - If licenses full: Branch to "Queue for Approval"
   - Otherwise: Continue to next step

3. **MySQL Node**: Create user record
   ```sql
   INSERT INTO users (id, email, first_name, last_name, role, status, created_at)
   VALUES (?, ?, ?, ?, ?, 'pending', NOW())
   ```

4. **HTTP Nodes** (Parallel): Provision external services
   - Call SSO provisioning API
   - Call tool subscription API
   - Call team management API
   - Each with retry logic

5. **Email Node**: Send welcome email
   - Template: Welcome email with login link
   - To: user email
   - Subject: "Welcome to AiGentForce!"

6. **Gmail/Outlook Node**: Add to team distribution list
   - Group: Engineering, Sales, etc. (based on department)

7. **Slack Node**: Post announcement
   - Channel: #new-members
   - Message: "Please welcome {{firstName}} {{lastName}} to the team!"

8. **Schedule Node**: Set follow-up reminders
   - Create scheduled workflows for:
     - 24h: Send training resources
     - 7d: Check-in survey
     - 30d: 1-on-1 feedback

9. **Audit Log Node**: Store complete audit trail
   ```sql
   INSERT INTO audit_log (event, user_id, details, timestamp)
   VALUES ('user_onboarded', ?, ?, NOW())
   ```

**Estimated Execution Time:** 10-15 seconds

---

## Template 3: Monitoring & Alerting

**Use Case:** Continuously monitor workflow health, detect anomalies, and trigger alerts

**Prerequisites:**
- MySQL database for metrics
- Slack webhook
- Email service
- (Optional) PagerDuty account

**Nodes Overview:**
```
Schedule (5min) → Fetch Metrics → Analyze Thresholds → Determine Severity → 
→ Route Alerts → Create Incident → Send Summary Dashboard
```

**Setup Instructions:**

1. **Schedule Node**: Run every 5 minutes
   - Cron: `*/5 * * * *`

2. **MySQL Node**: Query KPI metrics
   ```sql
   SELECT 
     (SELECT COUNT(*) FROM workflow_executions 
      WHERE status='success' AND created_at > DATE_SUB(NOW(), INTERVAL 1 HOUR)) / 
     (SELECT COUNT(*) FROM workflow_executions 
      WHERE created_at > DATE_SUB(NOW(), INTERVAL 1 HOUR)) * 100 as success_rate,
     
     AVG(duration) as avg_response_time,
     
     COUNT(*) as failed_count
   FROM workflow_executions 
   WHERE status='failed' AND created_at > DATE_SUB(NOW(), INTERVAL 1 HOUR)
   ```

3. **Switch Node**: Determine alert severity
   - Condition 1: `success_rate < 95%` → Severity: HIGH
   - Condition 2: `avg_response_time > 5000` → Severity: MEDIUM
   - Default: INFO (log only)

4. **Conditional Branches**:
   - **HIGH**: Email + Slack + PagerDuty incident
   - **MEDIUM**: Slack alert
   - **INFO**: Log to database

5. **Slack Node** (HIGH severity): Send alert
   ```
   🚨 CRITICAL ALERT
   Success Rate: {{success_rate}}%
   Response Time: {{avg_response_time}}ms
   Failed Workflows: {{failed_count}}
   Action Required: Review failed executions
   ```

6. **PagerDuty Node** (HIGH severity): Create incident
   - Severity: Critical
   - Service: AiGentForce Workflows
   - Description: Workflow health degradation

7. **Email Node** (End of workflow): Send daily summary
   - Schedule: 9 AM daily
   - Recipients: ops-team@company.com
   - Content: KPI dashboard with trends

**Metrics to Track:**
- Workflow success rate (target: >99%)
- Average response time (target: <5s)
- Failed workflows (target: 0)
- Queue depth
- API error rates by service

---

## Template 4: Scheduled Data Sync

**Use Case:** Periodically sync data between AiGentForce and external systems

**Prerequisites:**
- MySQL database
- External API endpoints
- Credentials for external services

**Nodes Overview:**
```
Schedule → Fetch Modified Records → Batch Processor → Parallel Transform & Sync → 
→ Aggregate Results → Notify Team → Error Handler
```

**Setup Instructions:**

1. **Schedule Node**: Run daily at 2 AM
   - Cron: `0 2 * * *`
   - Timezone: UTC

2. **MySQL Node**: Fetch unsynced documents
   ```sql
   SELECT * FROM documents 
   WHERE modified_at > DATE_SUB(NOW(), INTERVAL 1 DAY)
   AND sync_status IS NULL
   LIMIT 1000
   ```

3. **Batch Node**: Group records into chunks
   - Batch size: 50 records
   - Process in parallel (max 5 concurrent)

4. **Transform Node** (per batch):
   - Format data for external API
   - Validate required fields
   - Add timestamps

5. **HTTP Node** (per batch): POST to external system
   - URL: `https://api.external.com/v1/documents`
   - Method: POST
   - Headers: Authorization, Content-Type
   - Retry: 3 attempts with exponential backoff
   - Timeout: 60 seconds

6. **Error Handler** (per batch):
   - Log failed records
   - Mark for manual review
   - Store error details

7. **Aggregate Node**: Collect all results
   - Count successful syncs
   - List failed record IDs
   - Calculate execution time

8. **Notification Nodes**:
   - **Success**: Log summary, update sync_status to 'completed'
   - **Failure**: Alert admin via Slack and email

9. **Schedule Retry**: If failures exceed threshold
   - Retry failed batch in 1 hour
   - Max retries: 3

**Performance Targets:**
- Throughput: 1000 documents/execution
- Success rate: >99%
- Execution time: <10 minutes

---

## Common Error Handling Patterns

### Exponential Backoff Retry

```n8n
Node Settings:
  Max Retries: 3
  Wait Between Retries: Exponential
  Initial Delay: 1000ms
  Max Delay: 60000ms
```

### Circuit Breaker

```n8n
IF failed_attempts > 5 in last 5 minutes
  → Set circuit breaker state: OPEN
  → Return 429 Too Many Requests
  → Schedule retry in 5 minutes
ELSE
  → Execute workflow normally
```

### Graceful Degradation

```n8n
Try:
  → Call primary API
Catch:
  → Try fallback API
Catch:
  → Use cached data
Catch:
  → Return default response
```

---

## Credentials & Secrets Management

**Best Practices:**

1. Store credentials in n8n Credentials Manager, NOT in workflow code
2. Use environment variables for sensitive data
3. Rotate API keys quarterly
4. Audit credential access
5. Use separate credentials for dev/staging/prod

**Example Configuration:**
```bash
# .env.local
N8N_OPENAI_API_KEY=sk-xxx...
N8N_MYSQL_HOST=db.internal
N8N_MYSQL_USER=n8n_user
N8N_MYSQL_PASSWORD=secure-password
N8N_SLACK_WEBHOOK=https://hooks.slack.com/...
```

---

## Testing & Validation

1. **Unit Tests**: Test individual nodes with mock data
2. **Integration Tests**: Test workflows end-to-end
3. **Load Tests**: Verify performance under stress
4. **Monitoring**: Track execution metrics post-deployment

**Recommended Test Data:**
- Small batch (10 records)
- Medium batch (100 records)
- Large batch (1000 records)
- Edge cases (malformed data, timeouts)

---

## Deployment Checklist

- [ ] All credentials configured
- [ ] Workflows tested in staging
- [ ] Error handlers configured
- [ ] Monitoring and alerts enabled
- [ ] Backups configured
- [ ] Team trained on operations
- [ ] Documentation updated
- [ ] Runbook created for common issues

---

## Additional Resources

- [WORKFLOW_AUTOMATION_OPTIMIZATION.md](./WORKFLOW_AUTOMATION_OPTIMIZATION.md) - Complete optimization guide
- [n8n Documentation](https://docs.n8n.io/)
- [n8n Community Forum](https://community.n8n.io/)
- [LangChain Docs](https://docs.langchain.com/)
- [OpenAI API Reference](https://platform.openai.com/docs/)
