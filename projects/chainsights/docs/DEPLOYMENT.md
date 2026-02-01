# Deployment Guide - Free Tier Launch

## Prerequisites

- Database access (Postgres)
- npm installed
- Environment variables configured (.env.local)

## Step 1: Run Database Migration

The free tier requires a new database table. Run the migration:

```bash
npm run db:push
```

This will create:
- `identifier_type` enum (email, wallet)
- `free_query_log` table
- Performance indexes
- Unique constraint for quota enforcement

**Verify migration:**
```sql
SELECT * FROM free_query_log LIMIT 1;
```

Should return empty result (table exists but no data yet).

## Step 2: Deploy Frontend

```bash
npm run build
npm run start
```

Or deploy to Vercel:
```bash
npx vercel --prod
```

## Step 3: Test the Flow

1. **Homepage**: Click "Get Report" button
2. **Select DAO**: Choose a DAO that has existing reports (e.g., one you've generated before)
3. **Data Check**: System should show "Data available!"
4. **Free Tier**: Click "Get Free Score"
5. **Register**: Enter email + check consent
6. **Results**: Should see rating instantly
7. **Share**: Test Twitter share button

## Step 4: Verify Database

After testing, check that data was saved:

```sql
SELECT
  dao_id,
  rating_score,
  consent_given,
  month_key,
  created_at
FROM free_query_log
ORDER BY created_at DESC
LIMIT 10;
```

You should see your test entry!

## Step 5: Test Quota Enforcement

Try to get another free rating with the same email within the same month:
- Should see error: "You've used your free rating this month"
- Should show reset date (first day of next month)

## Step 6: Launch Marketing

Now you're ready to promote! See Maya's marketing plan in the commit messages.

### Quick Launch Checklist

- [ ] Database migration completed
- [ ] Frontend deployed
- [ ] Free tier tested end-to-end
- [ ] Quota enforcement working
- [ ] Paid tier still works (€49/€149)
- [ ] Social sharing buttons work
- [ ] Twitter handle is correct (@ChainSights_one)

## Monitoring

Watch these metrics after launch:
- Free tier signups per day
- Free → Paid conversion rate
- Social shares per free rating
- Quota exceeded rate (should be low)

## Troubleshooting

### "Database not configured" error
- Check DATABASE_URL in .env.local
- Verify database connection

### "No rating data available"
- User selected DAO with no reports yet
- Add more DAOs via admin or suggest "Request Analysis"

### Quota not working
- Check month_key format in database (should be 'YYYY-MM')
- Verify unique constraint exists: `unique_monthly_query`

### Email not showing in identifier detection
- Check that email contains '@' symbol
- Check wallet starts with '0x'

## What's Next (Optional Enhancements)

1. **Email Delivery**: Send ratings via email (Phase 7)
2. **Rankings Integration**: Use free tier data in public rankings (Phase 6)
3. **Admin Dashboard**: View free tier usage, quotas (Phase 5)
4. **Analytics**: Track conversion funnel (Phase 9)

---

**Ready to launch! 🚀**
