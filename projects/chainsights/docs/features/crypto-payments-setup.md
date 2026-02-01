# Crypto Payments Setup Guide

**Status:** ✅ Implemented
**Date:** 2024-12-20
**Feature:** USDC payment option for ChainSights reports

---

## Overview

ChainSights now supports crypto payments using USDC on Ethereum mainnet, alongside the existing Stripe credit card payments.

**What was implemented:**
- USDC payment option on pricing page
- Wallet connection (MetaMask, WalletConnect, Coinbase Wallet)
- On-chain transaction verification
- Database schema updated to track crypto payments
- API endpoint to process crypto payments

---

## Configuration Required

### 1. Environment Variables

Add these to your `.env.local` file:

```bash
# REQUIRED: Treasury wallet for receiving USDC
TREASURY_WALLET_ADDRESS=0x...
NEXT_PUBLIC_TREASURY_WALLET_ADDRESS=0x...  # Same address, but public

# RECOMMENDED: Alchemy API for on-chain verification
ALCHEMY_API_KEY=...  # Get from https://alchemy.com

# OPTIONAL: WalletConnect for better wallet support
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=...  # Get from https://cloud.walletconnect.com
```

### 2. Treasury Wallet Setup

**What you need:**
1. An Ethereum wallet address (e.g., MetaMask, Gnosis Safe)
2. This wallet will receive all USDC payments
3. Store private keys securely (NEVER commit to git)

**Steps:**
1. Create or use existing Ethereum wallet
2. Copy the wallet address (0x...)
3. Add to `.env.local` as `TREASURY_WALLET_ADDRESS`
4. Add same address as `NEXT_PUBLIC_TREASURY_WALLET_ADDRESS`

**Recommended wallet options:**
- **MetaMask** - Simple, for testing
- **Gnosis Safe** - Multi-sig, for production (recommended)
- **Ledger/Trezor** - Hardware wallet for security

### 3. Alchemy API Key (Recommended)

**Why needed:** Verifies USDC transactions on-chain to prevent fraud

**Steps:**
1. Go to https://alchemy.com
2. Sign up for free account
3. Create new app → Select "Ethereum" → "Mainnet"
4. Copy API key
5. Add to `.env.local` as `ALCHEMY_API_KEY`

**Without Alchemy:**
- Falls back to Cloudflare public RPC (slower, rate-limited)
- Still works, but verification may fail under load

### 4. WalletConnect Project ID (Optional)

**Why needed:** Better wallet connection experience (QR codes, mobile wallets)

**Steps:**
1. Go to https://cloud.walletconnect.com
2. Create account (free)
3. Create new project
4. Copy Project ID
5. Add to `.env.local` as `NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID`

**Without WalletConnect:**
- MetaMask and Coinbase Wallet still work
- Users can't scan QR codes with mobile wallets
- Acceptable for MVP

---

## Verification Checklist

Before going live with crypto payments:

### Pre-Launch:
- [ ] Treasury wallet configured in `.env.local`
- [ ] Alchemy API key configured (recommended)
- [ ] Test payment on testnet first (see Testing section)
- [ ] Verify database migration ran successfully
- [ ] Check that pricing page shows USDC buttons

### Post-Launch Monitoring:
- [ ] Monitor treasury wallet for incoming USDC
- [ ] Check database for `paymentMethod='crypto'` orders
- [ ] Verify orders are created correctly with tx hashes
- [ ] Test full flow: payment → verification → intake form

---

## Testing

### Local Development Testing:

```bash
# 1. Run database migration
npm run db:push

# 2. Start dev server
npm run dev

# 3. Open http://localhost:3000
# 4. Click "Pay with USDC" button
# 5. Connect wallet (must have USDC on Ethereum mainnet)
```

### Production Testing (Use Testnet First!):

**IMPORTANT:** Before accepting real payments, test on Sepolia testnet:

1. Update wagmi config to use Sepolia:
   ```typescript
   // src/lib/crypto/wagmi-config.ts
   import { sepolia } from 'wagmi/chains'
   chains: [sepolia]  // Instead of mainnet
   ```

2. Use Sepolia USDC contract (not mainnet!)
3. Get testnet USDC from faucet
4. Test full payment flow
5. Switch back to mainnet before production

---

## How It Works

### User Flow:
1. User clicks "Pay with USDC" on pricing page
2. Modal opens → User connects wallet (MetaMask/WalletConnect/Coinbase)
3. Modal displays treasury wallet address and amount
4. User sends USDC from their wallet
5. User enters transaction hash and email
6. Backend verifies transaction on-chain
7. Order created in database
8. User redirected to intake form

### Technical Flow:
1. **Frontend:** `src/components/pricing.tsx` → Opens `CryptoPaymentModal`
2. **Wallet Connection:** Uses wagmi to connect wallet
3. **User Sends:** Manual USDC transfer to treasury wallet
4. **Verification:** User submits tx hash to `/api/crypto-checkout`
5. **Backend:** `src/app/api/crypto-checkout/route.ts` calls verification
6. **On-Chain Check:** `src/lib/crypto/verify-transaction.ts` verifies:
   - Transaction exists and succeeded
   - Sent to correct treasury wallet
   - Amount matches expected USDC (±1% tolerance)
   - Has minimum 3 confirmations
7. **Database:** Order created with `paymentMethod='crypto'`
8. **Success:** Returns order ID for intake form

---

## Monitoring & Support

### Check Crypto Orders:

```sql
-- View all crypto payments
SELECT
  id,
  customer_email,
  tier,
  amount_cents,
  crypto_tx_hash,
  crypto_wallet_address,
  created_at
FROM orders
WHERE payment_method = 'crypto'
ORDER BY created_at DESC;
```

### Troubleshooting:

**Issue:** User says payment failed verification

**Check:**
1. Transaction hash format (must start with 0x, 66 chars)
2. Transaction is confirmed (at least 3 blocks)
3. Sent to correct treasury wallet
4. Amount is correct (€49 = ~54 USDC, €149 = ~163 USDC)
5. Is USDC token (not ETH, USDT, or other token)

**Common issues:**
- User sent wrong token (USDT instead of USDC)
- User sent on wrong network (Polygon, BSC instead of Ethereum)
- User sent wrong amount
- Transaction still pending (not enough confirmations)

**Resolution:**
- If wrong token/network: Funds are lost (user error)
- If wrong amount: Create manual order or refund
- If pending: Ask user to wait for confirmations

---

## Security Considerations

### ⚠️ CRITICAL SECURITY:

1. **On-Chain Verification:**
   - Never trust client-provided data
   - Always verify transaction on-chain
   - Check recipient, amount, and token contract
   - Current implementation does this via `verify-transaction.ts`

2. **Double-Spend Prevention:**
   - Each tx hash can only be used once
   - Database enforces unique constraint on `crypto_tx_hash`
   - Prevents same transaction being claimed twice

3. **Private Key Security:**
   - NEVER store treasury private keys in code
   - Use hardware wallet or multisig for production
   - Consider using separate treasury wallet (not personal)

4. **Amount Verification:**
   - Allow 1% tolerance for exchange rate fluctuations
   - EUR → USDC rate hardcoded at 1.09 (update as needed)
   - Consider using live exchange rate API

---

## Exchange Rate Management

**Current:** 1 EUR = 1.09 USD (hardcoded)

**Location:** `src/lib/crypto/verify-transaction.ts`

```typescript
export function eurToUSDC(eurAmount: number): number {
  const EUR_TO_USD = 1.09 // Update this periodically
  return Math.ceil(eurAmount * EUR_TO_USD)
}
```

**TODO for production:**
- Integrate real-time exchange rate API (CoinGecko, CoinMarketCap)
- Update rate daily or per-transaction
- Display current rate on payment modal

---

## Files Modified/Created

### New Files:
- `src/app/api/crypto-checkout/route.ts` - Crypto payment API endpoint
- `src/lib/crypto/verify-transaction.ts` - On-chain verification
- `src/lib/crypto/wagmi-config.ts` - Wallet connection config
- `src/components/crypto-payment-modal.tsx` - Payment modal UI
- `src/components/providers/web3-provider.tsx` - Web3 provider wrapper
- `scripts/migrate-crypto-payments.ts` - Database migration script
- `drizzle/0001_add_crypto_payments.sql` - SQL migration

### Modified Files:
- `src/lib/db/schema.ts` - Added crypto payment fields to orders table
- `src/components/pricing.tsx` - Added USDC payment buttons
- `src/app/layout.tsx` - Wrapped app with Web3Provider
- `.env.example` - Added crypto payment env vars
- `package.json` - Added wagmi, viem, @tanstack/react-query

---

## Next Steps

### Immediate (Before Launch):
1. ✅ Set treasury wallet address
2. ✅ Get Alchemy API key
3. ⏳ Test payment on testnet
4. ⏳ Test payment with real USDC (small amount)
5. ⏳ Monitor first real transaction

### Future Enhancements:
- [ ] Automated transaction monitoring (webhook instead of user input)
- [ ] Support for other stablecoins (USDT, DAI)
- [ ] Multi-chain support (Polygon, Arbitrum, Base)
- [ ] Real-time exchange rate API integration
- [ ] Automated refunds for failed verifications
- [ ] Transaction status tracking page

---

## Support & Questions

**Questions about setup?**
- Check this documentation first
- Review code comments in key files
- Test on Sepolia testnet before mainnet

**Production issues?**
- Check database for order records
- Verify transaction on Etherscan
- Check treasury wallet balance
- Review API logs for verification failures

---

*Last Updated: 2024-12-20*
*Author: Barry (Quick Flow Solo Dev)*
