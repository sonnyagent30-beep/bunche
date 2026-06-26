# Bunche 🤝

**WhatsApp proxy reseller for the Nigerian market. Fully automated.**

Zero inventory. Zero upfront cost. Customer pays first → you buy proxy → deliver.

---

## How It Works

```
Customer messages "Order ISP UK 1" on WhatsApp
        ↓
Bunche sends Flutterwave payment link
        ↓
Customer pays ₦6,500
        ↓
Flutterwave webhook → Bunche calls provider API
        ↓
Proxy credentials delivered on WhatsApp (< 2 min)
```

---

## Products

| Product | Price | Provider | Tracking |
|---------|-------|----------|----------|
| ISP (UK/US/DE/FR/CA) | ₦6,500/mo | Proxy-Seller | Expires on date |
| ISP (JP/AU/BR/SG) | ₦7,500/mo | Proxy-Seller | Expires on date |
| Residential 5GB | ₦5,000 | DataImpulse | No time expiry — lasts until GB used |
| Residential 10GB | ₦9,000 | DataImpulse | No time expiry |
| Mobile 4G 5GB | ₦20,000 | DataImpulse | 30-day window to use GB |
| Mobile 4G 10GB | ₦35,000 | DataImpulse | 30-day window to use GB |
| Datacenter | ₦2,500/mo | Proxy-Seller | Expires on date |

---

## Architecture

```
WhatsApp → Cloudflare → Nginx → n8n (Docker on VPS)
                                      ↓
                               PostgreSQL + Redis
                                      ↓
                               MiniMax M2 (LLM)
                                      ↓
                          Proxy-Seller / DataImpulse APIs
```

- **n8n**: Workflow engine (15 workflows documented)
- **PostgreSQL**: Customers, orders, audit logs
- **Redis**: Caching, sessions, rate limiting
- **MiniMax M2**: LLM for intent parsing and responses
- **Flutterwave**: Payment processing
- **Cloudflare R2**: File storage (screenshots, receipts)

---

## Docs

| File | What it covers |
|------|---------------|
| `docs/DEPLOYMENT.md` | Full VPS deployment guide |
| `docs/ARCHITECTURE_PLAN.md` | System architecture |
| `docs/DATABASE_SCHEMA.md` | PostgreSQL schema |
| `docs/SECURITY_PLAN.md` | Security implementation |
| `docs/FLUTTERWAVE_WHATSAPP_SETUP.md` | Payment + messaging setup |
| `docs/REFERRAL_SYSTEM.md` | Referral system spec |
| `workflows/WORKFLOW_SPECS.md` | 15 workflows documented |
| `AGENTS.md` | Agent system documentation |
| `docs/adr/` | Architecture Decision Records |

---

## Workflow Templates

Actual n8n JSON workflows in `.n8n/workflows/`:

| Workflow | File |
|----------|------|
| Order Handler | `order-handler.json` |
| Payment Confirmation | `payment-confirmation.json` |
| Data Alert Escalation | `data-alert.json` |
| Referral Credit Processor | `referral-credit.json` |
| Daily Summary | `daily-summary.json` |
| Error Alert | `error-alert.json` |

---

## Legal

`legal/TERMS_OF_SERVICE.md` · `legal/PRIVACY_POLICY.md` · `legal/ACCEPTABLE_USE_POLICY.md`

---

## Archive

Obsolete docs (Google Sheets era, old providers): `archive/`
