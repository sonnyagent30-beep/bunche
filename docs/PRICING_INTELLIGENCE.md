# Bunche — Pricing Intelligence

**Last Updated:** 2026-06-27
**Purpose:** Buy price, sell price, exchange rate, and profit margins for all products.

---

## Exchange Rate

| Rate | Value | Source | Notes |
|------|-------|--------|-------|
| **Official NGN/USD** | ₦1,500/$1 | CBN official rate | Not realistic for business |
| **Parallel market** | ₦1,550–1,600/$1 | LocalFX / unofficial | What we actually get when buying USD |
| **Safe working rate** | **₦1,500/$1** | Assumed | Used for all margin calculations below |

**⚠️ FX Risk:** If parallel market moves to ₦1,700+/$1, residential margins go negative. Monitor monthly.

---

## Buy Price (What Bunche Pays Providers)

### Proxy-Seller (Primary — ISP & DC)

| Product | Buy Price | Notes |
|---------|----------|-------|
| ISP (UK/US/DE/FR/CA) | $1.50/IP/month | 1 IP minimum, billed monthly |
| ISP (JP/AU/BR/SG/KR) | $2.00/IP/month | Premium countries |
| Datacenter | $0.70/IP/month | Standard DC |

**Proxy-Seller API:** `https://api.proxy-salesman.com/v2/`

---

### DataImpulse (Secondary — Residential & Mobile)

| Product | Buy Price | Notes |
|---------|----------|-------|
| Residential (PAYG) | $1.00/GB | No expiry on purchased GB |
| Residential (50GB+) | $0.80/GB | Volume discount |
| Residential (1TB+) | $0.50/GB | Enterprise volume |
| Mobile | $2.00/GB | 4G/LTE carrier IPs |

**DataImpulse API:** `https://api.dataimpulse.com/`

---

### Rayobyte (DC Rotating — Future)

| Product | Buy Price | Notes |
|---------|----------|-------|
| DC Rotating | $0.45/GB | Entry, unlimited bandwidth |
| DC Rotating (10GB+) | $0.30/GB | Volume discount |

---

## Sell Price (What Customers Pay)

| Product | Sell Price (₦) | Tracking |
|---------|--------------|----------|
| ISP (UK/US/DE/FR/CA) | ₦6,500/mo | Time-based expiry |
| ISP (JP/AU/BR/SG/KR) | ₦7,500/mo | Time-based expiry |
| Datacenter | ₦3,000/mo | Time-based expiry |
| Residential 5GB | ₦9,500 | GB-based, no time expiry |
| Residential 10GB | ₦18,000 | GB-based, no time expiry |
| Mobile 4G 5GB | ₦20,000 | 30-day window + GB cap |
| Mobile 4G 10GB | ₦38,000 | 30-day window + GB cap |

---

## Profit Margins (at ₦1,500/USD)

### ISP Products

| Product | Sell | Buy (USD) | Buy (₦) | Profit (₦) | Margin |
|---------|------|----------|---------|-----------|--------|
| ISP UK/US/DE/FR/CA | ₦6,500 | $1.50 | ₦2,250 | **₦4,250** | **65.4%** |
| ISP JP/AU/BR/SG/KR | ₦7,500 | $2.00 | ₦3,000 | **₦4,500** | **60.0%** |

### Datacenter

| Product | Sell | Buy (USD) | Buy (₦) | Profit (₦) | Margin |
|---------|------|----------|---------|-----------|--------|
| DC | ₦3,000 | $0.70 | ₦1,050 | **₦1,950** | **65.0%** |

### Residential

| Product | Sell | Buy (USD) | Buy (₦) | Profit (₦) | Margin |
|---------|------|----------|---------|-----------|--------|
| Residential 5GB | ₦9,500 | $5.00 | ₦7,500 | **₦2,000** | **21.1%** |
| Residential 10GB | ₦18,000 | $10.00 | ₦15,000 | **₦3,000** | **16.7%** |

**⚠️ Note:** Residential margins are lower. The ₦9,500/5GB price compensates for DataImpulse's $1/GB rate.

### Mobile

| Product | Sell | Buy (USD) | Buy (₦) | Profit (₦) | Margin |
|---------|------|----------|---------|-----------|--------|
| Mobile 4G 5GB | ₦20,000 | $10.00 | ₦15,000 | **₦5,000** | **25.0%** |
| Mobile 4G 10GB | ₦38,000 | $20.00 | ₦30,000 | **₦8,000** | **21.1%** |

---

## Full Margin Table (All Products)

| Product | Sell (₦) | Cost (₦) | Profit (₦) | Margin % | Status |
|---------|---------|---------|-----------|---------|--------|
| ISP UK/US/DE/FR/CA | 6,500 | 2,250 | **4,250** | 65.4% | ✅ Healthy |
| ISP JP/AU/BR/SG/KR | 7,500 | 3,000 | **4,500** | 60.0% | ✅ Healthy |
| Datacenter | 3,000 | 1,050 | **1,950** | 65.0% | ✅ Healthy |
| Residential 5GB | 9,500 | 7,500 | **2,000** | 21.1% | ✅ Profitable |
| Residential 10GB | 18,000 | 15,000 | **3,000** | 16.7% | ✅ Profitable |
| Mobile 4G 5GB | 20,000 | 15,000 | **5,000** | 25.0% | ✅ Healthy |
| Mobile 4G 10GB | 38,000 | 30,000 | **8,000** | 21.1% | ✅ Healthy |

---

## Break-Even Analysis

### At what FX rate does each product become unprofitable?

| Product | Break-even FX | Current FX | Buffer |
|---------|--------------|------------|--------|
| ISP UK | ₦4,333/USD | ₦1,500 | +189% |
| DC | ₦2,857/USD | ₦1,500 | +90% |
| Residential 5GB | ₦1,900/USD | ₦1,500 | +27% |
| Mobile 4G 5GB | ₦2,000/USD | ₦1,500 | +33% |

**Residential is most sensitive to FX.** If NGN weakens past ₦1,900/$, residential becomes loss-making.

---

## Cost Per Order (Variable Cost)

| Product | Variable Cost | Notes |
|---------|--------------|-------|
| ISP | $1.50–2.00/order | Billed monthly regardless of usage |
| DC | $0.70/order | Billed monthly |
| Residential | $1.00/GB used | Only paid for GB customer actually uses |
| Mobile | $2.00/GB used | Only paid for GB customer actually uses |

**Key insight:** Residential and Mobile are variable-cost. You only pay what the customer uses. ISP and DC are fixed monthly costs even if customer uses 0 GB.

---

## Recommended Pricing Adjustments (if FX moves)

If parallel market FX moves to ₦1,700/$:

| Product | Current Sell | Recommended Sell | New Margin |
|---------|-------------|-----------------|------------|
| ISP UK | ₦6,500 | ₦7,500 | 64.0% |
| Residential 5GB | ₦9,500 | ₦10,500 | 19.0% |

---

## Volume Discounts from Providers

### DataImpulse Volume Tiers

| GB Purchased | Rate/GB | 5GB Cost | 10GB Cost |
|-------------|---------|---------|----------|
| PAYG | $1.00 | $5.00 | $10.00 |
| 50GB+ | $0.80 | $4.00 | $8.00 |
| 100GB+ | $0.65 | $3.25 | $6.50 |
| 500GB+ | $0.55 | $2.75 | $5.50 |
| 1TB+ | $0.50 | $2.50 | $5.00 |

### Proxy-Seller Volume Tiers

| Product | 1 IP | 10 IPs | 50 IPs |
|---------|------|--------|--------|
| ISP | $1.50/IP | $1.35/IP | $1.20/IP |
| DC | $0.70/IP | $0.63/IP | $0.55/IP |

---

## Provider Comparison (Current Stack)

| Product | Proxy-Seller | DataImpulse | Winner |
|---------|-------------|-------------|--------|
| ISP | $1.50/IP | N/A | Proxy-Seller only option |
| DC | $0.70/IP | N/A | Proxy-Seller only option |
| Residential | $2.20/GB | **$1.00/GB** | ✅ DataImpulse |
| Mobile | N/A | **$2.00/GB** | ✅ DataImpulse only option |

---

## Operational Costs (Monthly)

| Item | Cost |
|------|------|
| Hetzner CX21 VPS | €6.00 (~$9 @ ₦1,500) |
| Domain (bunche.ng) | ~$2/mo |
| **Total infrastructure** | **~$11/mo** |

At 20 customers/month: infrastructure cost = ₦550/customer (negligible).

---

## Notes

- All buy prices are retail published rates. No negotiated reseller rates confirmed.
- LunaProxy and PrivateProxy were researched but excluded (dead domains / unverified).
- Rayobyte pending integration for DC rotating product.
- FX rate is the biggest risk to margins — monitor parallel market monthly.
