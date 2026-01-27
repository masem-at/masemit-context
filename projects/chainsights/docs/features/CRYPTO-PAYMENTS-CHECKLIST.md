# Crypto Payments - Quick Setup Checklist

**Mario, this is what YOU need to do to enable crypto payments:**

---

## ✅ REQUIRED (Must Do Now)

### 1. Set Treasury Wallet Address

Add to `.env.local`:

```bash
TREASURY_WALLET_ADDRESS=YOUR_ETHEREUM_ADDRESS_HERE
NEXT_PUBLIC_TREASURY_WALLET_ADDRESS=YOUR_ETHEREUM_ADDRESS_HERE
```

**What address to use:**
- Your MetaMask address (simple, for testing)
- OR a Gnosis Safe multisig (recommended for production)
- This wallet will receive all USDC payments

**Security:**
- ⚠️ Use a dedicated wallet, not your personal one
- ⚠️ For production: Use Gnosis Safe with 2-of-3 multisig
- ⚠️ NEVER commit private keys to git

---

## 🔶 RECOMMENDED (Should Do Soon)

### 2. Get Alchemy API Key

**Why:** Verifies transactions on-chain to prevent fraud

**Steps:**
1. Go to https://alchemy.com
2. Sign up (free)
3. Create app: Ethereum → Mainnet
4. Copy API key
5. Add to `.env.local`:
   ```bash
   ALCHEMY_API_KEY=your_key_here
   ```

**Without it:** Falls back to public RPC (slower, may fail under load)

---

## 📝 OPTIONAL (Nice to Have)

### 3. WalletConnect Project ID

**Why:** Better wallet support (QR codes, mobile wallets)

**Steps:**
1. Go to https://cloud.walletconnect.com
2. Create account (free)
3. Create project
4. Copy Project ID
5. Add to `.env.local`:
   ```bash
   NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_project_id
   ```

**Without it:** MetaMask still works, just no QR code scanning

---

## 🚀 Deployment Steps

### 1. Add Env Vars to Vercel

```bash
# In Vercel dashboard, add these environment variables:
TREASURY_WALLET_ADDRESS=0x...
NEXT_PUBLIC_TREASURY_WALLET_ADDRESS=0x...  # Same as above
ALCHEMY_API_KEY=...  # If you got one
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=...  # If you got one
```

### 2. Deploy

```bash
# Database migration already ran locally
# Just deploy the new code:
git add .
git commit -m "Add USDC payment option"
git push

# Or use Vercel CLI:
npx vercel --prod
```

### 3. Test

1. Go to https://chainsights.one
2. Scroll to pricing
3. Click "Pay with USDC"
4. Connect wallet
5. Send small test payment (10-20 USDC)
6. Verify it works end-to-end

---

## 💰 Receiving Payments

### Where to check:

**1. Treasury Wallet (On-chain):**
- Open https://etherscan.io
- Search for your treasury wallet address
- Click "Token" tab
- Look for USDC balance

**2. Database:**
```sql
SELECT * FROM orders
WHERE payment_method = 'crypto'
ORDER BY created_at DESC;
```

**3. Admin Panel:**
- Coming soon (or manually check database for now)

---

## ⚠️ IMPORTANT NOTES

### Exchange Rate:
- Currently hardcoded: 1 EUR = 1.09 USD
- €99 = ~109 USDC
- €199 = ~218 USDC
- Update periodically in `src/lib/crypto/verify-transaction.ts` if needed

### Supported Networks:
- ✅ Ethereum Mainnet only
- ❌ NOT Polygon, BSC, Arbitrum, etc.
- ❌ NOT USDT, DAI, or other stablecoins

### User sends wrong token/network:
- Funds are lost (user error, not your fault)
- Consider adding warning in UI
- Already have warning in payment modal

---

## 🐛 Troubleshooting

### "Payment verification failed"

**Check:**
1. Is transaction confirmed? (Need 3+ confirmations)
2. Did user send to correct wallet?
3. Is it USDC (not USDT)?
4. Is it Ethereum mainnet (not Polygon)?
5. Is amount correct?

**View transaction:**
- Get tx hash from user
- Check on https://etherscan.io
- Look for USDC transfer event

### "No USDC received"

**Check:**
1. User sent to correct wallet address?
2. User sent USDC (not ETH)?
3. User sent on Ethereum mainnet?
4. Transaction is confirmed?

---

## 📊 Summary of Changes

**What was added:**
- ✅ "Pay with USDC" buttons on pricing page
- ✅ Crypto payment modal (wallet connection + instructions)
- ✅ On-chain transaction verification
- ✅ Database schema updated (new columns in orders table)
- ✅ API endpoint `/api/crypto-checkout`
- ✅ Full documentation in `docs/features/`

**What you need to do:**
1. Set treasury wallet address (REQUIRED)
2. Get Alchemy API key (recommended)
3. Deploy to production
4. Test with small payment
5. Monitor first real transactions

---

## 📞 Need Help?

**Read full docs:** `docs/features/crypto-payments-setup.md`

**Check code:**
- Payment modal: `src/components/crypto-payment-modal.tsx`
- API endpoint: `src/app/api/crypto-checkout/route.ts`
- Verification: `src/lib/crypto/verify-transaction.ts`

**Database schema:** `src/lib/db/schema.ts` (search for "crypto")

---

*TLDR: Add treasury wallet address to .env.local, get Alchemy key, deploy. Done.*
