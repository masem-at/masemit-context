# Disaster Recovery Runbook

**Last Updated:** 2025-12-23
**Document Owner:** Mario (masemIT)
**Review Cycle:** Monthly

---

## Overview

This runbook provides step-by-step procedures for recovering ChainSights from various failure scenarios. Follow these procedures in order of relevance to your incident.

### Recovery Objectives

| Metric | Target | Description |
|--------|--------|-------------|
| **RTO (Recovery Time Objective)** | < 1 hour | Maximum time from incident detection to service restoration |
| **RPO (Recovery Point Objective)** | < 24 hours (ideally < 1 hour) | Maximum acceptable data loss |

### Quick Reference - Which Procedure to Use

| Incident Type | Primary Recovery | Backup Option |
|---------------|------------------|---------------|
| Bad code deployment | [Vercel Rollback](#3-deployment-rollback-vercel) | [Git Revert](#2-code-rollback-git) |
| Database corruption | [Neon PITR](#1-database-recovery-neon) | Daily backup restore |
| Data loss/deletion | [Neon PITR](#1-database-recovery-neon) | N/A |
| API key compromised | [Regenerate Keys](#4-environment-variables) | Rotate all keys |
| Complete outage | [Incident Response](#incident-severity-matrix) | Contact support |

---

## 1. Database Recovery (Neon)

### 1.1 Neon Automatic Backups

**Configuration (NFR-REL-15):**
- **Backup Frequency:** Automatic daily backups
- **Retention Period:** 30 days
- **Storage:** Neon-managed, geographically distributed

**Verification Steps:**
1. Log into Neon Console: https://console.neon.tech/
2. Navigate to your project → **Backups** tab
3. Verify "Automatic backups" is enabled
4. Confirm retention shows "30 days"

### 1.2 Point-in-Time Recovery (PITR)

Neon supports PITR to restore your database to any point within the retention window.

**When to Use:**
- Accidental data deletion
- Data corruption discovered
- Need to recover to a specific point before an incident

**Recovery Procedure:**

```bash
# Step 1: Identify the target recovery time
# Use UTC timestamps (e.g., 2025-12-23T10:30:00Z)

# Step 2: In Neon Console
1. Go to https://console.neon.tech/
2. Select your project (chainsights)
3. Click "Branches" in the left sidebar
4. Click "Create branch"
5. Select "Point in time" recovery type
6. Enter the target timestamp (before the incident)
7. Name the branch (e.g., "recovery-2025-12-23")
8. Click "Create branch"

# Step 3: Verify recovered data
1. Connect to the new branch using its connection string
2. Verify data integrity
3. Test critical queries

# Step 4: Promote recovery branch (if verified)
# Option A: Update DATABASE_URL in Vercel to point to recovery branch
# Option B: Copy data from recovery branch to main branch
```

**Post-Recovery Checklist:**
- [ ] Verify all tables have expected data
- [ ] Check most recent records match expected state
- [ ] Test critical application functions
- [ ] Update any cached data if applicable
- [ ] Document incident and recovery in Change Log

### 1.3 Full Database Restore

For complete database restoration from daily backup:

```bash
# Step 1: In Neon Console
1. Navigate to project → Backups
2. Select the backup date to restore from
3. Click "Restore" and follow prompts

# Step 2: Update application
1. Update DATABASE_URL if connection string changed
2. Redeploy application if needed
3. Verify application connectivity
```

---

## 2. Code Rollback (Git)

### 2.1 Understanding Git Rollback Options

**NFR-REL-16:** Git history allows rollback to any previous commit.

| Command | Use Case | Effect | Safe? |
|---------|----------|--------|-------|
| `git revert <commit>` | Undo specific commit | Creates new commit that undoes changes | Yes - preserves history |
| `git reset --soft HEAD~1` | Undo last commit locally | Keeps changes staged | Safe locally |
| `git reset --hard <commit>` | Reset to specific commit | Discards all changes after that commit | DESTRUCTIVE |

### 2.2 Safe Rollback Procedure (Recommended)

**Use `git revert` for production rollbacks:**

```bash
# Step 1: Identify the bad commit
git log --oneline -10

# Step 2: Create a revert commit
git revert <bad-commit-hash>

# Example: Revert the last commit
git revert HEAD

# Example: Revert a specific commit
git revert abc123f

# Step 3: Push the revert
git push origin master

# Step 4: Vercel will auto-deploy the reverted code
```

### 2.3 Emergency Hard Reset (Use with Caution)

**Only use if absolutely necessary and no one else is working on the branch:**

```bash
# Step 1: Identify last known good commit
git log --oneline -20

# Step 2: Reset to that commit (DESTRUCTIVE)
git reset --hard <good-commit-hash>

# Step 3: Force push (DANGEROUS - requires force push permission)
git push --force origin master
```

### 2.4 Post-Rollback Verification

```bash
# Verify current commit
git log --oneline -1

# Verify no unintended changes
git status

# Run build to confirm working state
npm run build

# Run tests
npm run test:unit
```

---

## 3. Deployment Rollback (Vercel)

### 3.1 One-Click Rollback (NFR-REL-17)

Vercel provides instant rollback to any previous deployment.

**When to Use:**
- Bad deployment causes errors
- Need immediate rollback while investigating
- Faster than git revert for urgent issues

**Rollback Procedure:**

```
Step 1: Access Vercel Dashboard
   - Go to https://vercel.com/dashboard
   - Select the chainsights project

Step 2: Navigate to Deployments
   - Click "Deployments" in the left sidebar
   - View list of all deployments with timestamps

Step 3: Identify Last Known Good Deployment
   - Look for deployments with green "Ready" status
   - Check commit message/hash to identify the right one
   - Hover over deployment to see preview URL (test if needed)

Step 4: Rollback
   - Click the "..." menu on the target deployment
   - Select "Promote to Production"
   - Confirm the rollback

Step 5: Verify
   - Visit https://chainsights.one
   - Check critical pages load correctly
   - Test key functionality
```

### 3.2 Identifying the Right Deployment

Each deployment shows:
- **Commit hash:** Links to GitHub commit
- **Commit message:** Description of changes
- **Timestamp:** When it was deployed
- **Status:** Ready (green), Error (red), Building (yellow)
- **URL:** Unique preview URL for testing

**Tip:** Test a deployment's preview URL before promoting to production.

### 3.3 Post-Rollback Actions

After successful rollback:
1. **Notify team:** Inform relevant parties of the rollback
2. **Investigate:** Determine root cause of the issue
3. **Fix forward:** Create proper fix in new commit
4. **Document:** Add incident to Change Log

---

## 4. Environment Variables

### 4.1 Critical Variables List

**Production Environment Variables (Vercel):**

| Variable | Purpose | Criticality | Regeneration |
|----------|---------|-------------|--------------|
| `DATABASE_URL` | Neon Postgres connection | CRITICAL | Neon Console → Connection Details |
| `STRIPE_SECRET_KEY` | Payment processing | CRITICAL | Stripe Dashboard → API Keys |
| `STRIPE_WEBHOOK_SECRET` | Webhook verification | CRITICAL | Stripe Dashboard → Webhooks |
| `ANTHROPIC_API_KEY` | AI analysis | HIGH | Anthropic Console |
| `RESEND_API_KEY` | Email delivery | HIGH | Resend Dashboard |
| `SENTRY_DSN` | Error tracking | MEDIUM | Sentry → Project Settings |
| `SENTRY_AUTH_TOKEN` | Sentry source maps | MEDIUM | Sentry → Auth Tokens |
| `ADMIN_PASSWORD` | Admin dashboard | HIGH | Generate new secure password |
| `SLACK_WEBHOOK_URL` | Alert notifications | LOW | Slack App → Webhooks |

### 4.2 Backup Location

All production environment variables should be backed up in:
- **Primary:** Password manager (1Password, Bitwarden, etc.)
- **Secondary:** Secure encrypted document (offline backup)

**Backup Format:**
```
VARIABLE_NAME=value
# Last updated: YYYY-MM-DD
# Rotated: YYYY-MM-DD (if applicable)
```

### 4.3 Key Regeneration Procedures

**If API Key is Compromised:**

1. **Stripe Keys:**
   ```
   1. Go to https://dashboard.stripe.com/apikeys
   2. Click "Roll key" on the compromised key
   3. Update STRIPE_SECRET_KEY in Vercel
   4. For webhooks: Go to Webhooks → Reveal signing secret
   5. Update STRIPE_WEBHOOK_SECRET in Vercel
   6. Redeploy application
   ```

2. **Database URL:**
   ```
   1. Go to Neon Console → Project → Connection Details
   2. Click "Reset password" if password compromised
   3. Copy new connection string
   4. Update DATABASE_URL in Vercel
   5. Redeploy application
   ```

3. **Anthropic API Key:**
   ```
   1. Go to https://console.anthropic.com/
   2. Navigate to API Keys
   3. Generate new key
   4. Update ANTHROPIC_API_KEY in Vercel
   5. Delete old key in Anthropic Console
   6. Redeploy application
   ```

4. **Resend API Key:**
   ```
   1. Go to https://resend.com/api-keys
   2. Create new API key
   3. Update RESEND_API_KEY in Vercel
   4. Delete old key
   5. Redeploy application
   ```

---

## 5. Emergency Contacts

### 5.1 Escalation Matrix

| Severity | Response Time | Who to Contact |
|----------|--------------|----------------|
| **P1 - Critical** (site down, data breach) | Immediate | Mario → Vercel/Neon Support |
| **P2 - High** (major feature broken) | < 1 hour | Mario |
| **P3 - Medium** (degraded performance) | < 4 hours | Mario |
| **P4 - Low** (minor issue) | < 24 hours | Log ticket |

### 5.2 Contact Information

**Internal:**
| Role | Name | Contact |
|------|------|---------|
| Founder/Operator | Mario | hello@chainsights.one |

**External Support:**

| Service | Support URL | Notes |
|---------|-------------|-------|
| **Vercel** | https://vercel.com/support | Pro plan includes priority support |
| **Neon** | https://neon.tech/docs/introduction/support | Free tier: Community support |
| **Stripe** | https://support.stripe.com/ | 24/7 for payment issues |
| **Anthropic** | https://support.anthropic.com/ | API issues |
| **Resend** | https://resend.com/support | Email delivery issues |
| **Sentry** | https://help.sentry.io/ | Error tracking issues |

### 5.3 When to Escalate to External Support

**Contact Vercel Support:**
- Deployment system failures
- DNS/routing issues
- Platform outages
- Serverless function errors not in our code

**Contact Neon Support:**
- Database connectivity issues
- PITR not working
- Performance degradation
- Data corruption suspected

**Contact Stripe Support:**
- Payment failures
- Webhook delivery issues
- Account/security concerns

---

## 6. Monthly Verification

### 6.1 Monthly Backup Verification Checklist

**Schedule:** First Monday of each month
**Duration:** ~30 minutes
**Owner:** Mario

#### Checklist

**Database Backups (Neon):**
- [ ] Log into Neon Console
- [ ] Verify automatic backups are enabled
- [ ] Confirm 30-day retention is active
- [ ] Check most recent backup timestamp (should be within 24 hours)
- [ ] Verify PITR is available for current date

**Git Repository:**
- [ ] Verify recent commits are visible on GitHub
- [ ] Confirm branch protection rules are active
- [ ] Test `git log` shows complete history

**Vercel Deployments:**
- [ ] Log into Vercel dashboard
- [ ] Verify deployment history shows recent deploys
- [ ] Confirm rollback option is available
- [ ] Check environment variables are present

**Environment Variables:**
- [ ] Verify all critical env vars are in Vercel
- [ ] Confirm backup copy exists in password manager
- [ ] Check last rotation date for API keys (rotate if > 90 days)

**Documentation:**
- [ ] Review this runbook for accuracy
- [ ] Update any changed procedures
- [ ] Verify external links still work

### 6.2 Test Restore Procedure (Quarterly)

**Schedule:** First month of each quarter
**Duration:** ~1 hour

**Steps:**
1. Create a test branch from Neon PITR (point 1 hour ago)
2. Verify data integrity in test branch
3. Delete test branch after verification
4. Document test results in Change Log below

### 6.3 Verification Log

| Date | Verified By | All Checks Passed | Notes |
|------|-------------|-------------------|-------|
| 2025-12-23 | Dev Agent | Initial creation | Runbook created |
| | | | |

---

## Incident Severity Matrix

### Severity Definitions

| Level | Definition | Examples | Response |
|-------|------------|----------|----------|
| **P1 - Critical** | Complete outage or data breach | Site unreachable, database down, security breach | Immediate action, all hands |
| **P2 - High** | Major functionality broken | Payments failing, rankings not loading, auth broken | < 1 hour response |
| **P3 - Medium** | Degraded experience | Slow performance, minor feature broken | < 4 hours response |
| **P4 - Low** | Minor issue | UI glitch, non-critical error | Next business day |

### Incident Response Flow

```
1. DETECT
   ↓
2. ASSESS SEVERITY (P1-P4)
   ↓
3. NOTIFY (based on severity matrix)
   ↓
4. INVESTIGATE
   - Check Sentry for errors
   - Check Vercel deployment status
   - Check Neon database status
   ↓
5. MITIGATE
   - If code issue: Vercel rollback
   - If data issue: Neon PITR
   - If env issue: Check/rotate keys
   ↓
6. RESOLVE
   - Fix root cause
   - Deploy fix
   - Verify resolution
   ↓
7. DOCUMENT
   - Update Change Log
   - Create post-mortem for P1/P2
```

---

## Change Log

| Date | Author | Change |
|------|--------|--------|
| 2025-12-23 | Dev Agent | Initial runbook creation (Story 8.5) |

---

## Appendix

### A. Quick Commands Reference

```bash
# Check recent git commits
git log --oneline -10

# Revert last commit
git revert HEAD

# Check current deployment status
# (Use Vercel dashboard - no CLI for instant rollback)

# Test database connection
npx drizzle-kit studio

# Run build to verify code health
npm run build

# Run tests
npm run test:unit
```

### B. Related Documentation

- [Neon PITR Documentation](https://neon.tech/docs/introduction/point-in-time-restore)
- [Vercel Rollbacks](https://vercel.com/docs/deployments/rollbacks)
- [Git Revert vs Reset](https://www.atlassian.com/git/tutorials/undoing-changes)
- [Stripe API Key Rotation](https://stripe.com/docs/keys#rolling-keys)

### C. Dashboard Quick Links

- **Neon Console:** https://console.neon.tech/
- **Vercel Dashboard:** https://vercel.com/dashboard
- **Stripe Dashboard:** https://dashboard.stripe.com/
- **Sentry Dashboard:** https://sentry.io/
- **GitHub Repository:** https://github.com/masem1899/chainsights
