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
- **Cloudflare R2**: File storage (screenshots, receipts, backups)
- **Bitlock.ai**: Free trial human verification
- **UptimeRobot**: Uptime monitoring (5-min checks)

---

## Docs

### Setup + Build

| File | What it covers |
|------|---------------|
| `docs/DEPLOYMENT.md` | Full VPS deployment guide (steps 1–11) |
| `docs/ARCHITECTURE_PLAN.md` | System architecture |
| `docs/DATABASE_SCHEMA.md` | PostgreSQL schema |
| `.env.example` | Every environment variable documented |

### Operational

| File | What it covers |
|------|---------------|
| `docs/MONITORING.md` | UptimeRobot setup, alert webhook |
| `docs/SECURITY_RUNBOOK.md` | Secrets rotation, API monitoring, NDPR, incident response |
| `docs/PERFORMANCE_SCALING.md` | pgBouncer, Redis caching, 4-phase cost trajectory |
| `docs/SECRET_ROTATION_LOG.md` | Rotation tracker |
| `docs/FLUTTERWAVE_WHATSAPP_SETUP.md` | Payment + messaging setup |

### Features

| File | What it covers |
|------|---------------|
| `docs/REFERRAL_SYSTEM.md` | Referral system spec (name = code, 5% credit) |
| `workflows/WORKFLOW_SPECS.md` | 15 workflows documented (orders, payments, alerts, referrals) |

### Architecture Decisions (ADRs)

| File | Decision |
|------|----------|
| `docs/adr/ADR-001-postgresql-primary-database.md` | PostgreSQL over Google Sheets |
| `docs/adr/ADR-002-minimax-m2-llm.md` | MiniMax M2 cloud over Ollama local |
| `docs/adr/ADR-003-name-as-referral-code.md` | Customer name = referral code |
| `docs/adr/ADR-004-secrets-management.md` | .env → Doppler → Vault phased approach |
| `docs/adr/ADR-005-backup-strategy.md` | Daily R2 backup + age encryption + 90-day retention |

### Legal

`legal/TERMS_OF_SERVICE.md` · `legal/PRIVACY_POLICY.md` · `legal/ACCEPTABLE_USE_POLICY.md`

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

## Backup Scripts

Operational scripts in `scripts/`:

| Script | Purpose |
|--------|---------|
| `scripts/backup-bunche.sh` | Daily pg_dump → age encrypt → rclone to R2 |
| `scripts/backup-monthly-archive.sh` | First-of-month → 1-year retention |
| `scripts/backup.conf.example` | Config template |

---

## Total Cost of Operation (Phase 1)

| Component | Cost |
|-----------|------|
| Hetzner CX21 VPS | €7/mo (~$7.50) |
| Domain | ~$1/mo amortized |
| Provider credits (Proxy-Seller + DataImpulse) | ~$50 one-time |
| Flutterwave fees | 1.5% of revenue (pass-through) |
| Cloudflare (free tier) | $0 |
| Cloudflare R2 (free tier + backups) | <$1/mo |
| UptimeRobot free | $0 |
| **Total fixed** | **~$8.50/mo** |
| **Cost per customer at 1,000 users** | **<$0.01** |

---

## Archive

Obsolete docs (Google Sheets era, old providers): `archive/`

---

## Status

- ✅ Research complete
- ✅ Strategy defined
- ✅ Architecture decided (PostgreSQL + MiniMax + Bitlock)
- ✅ Legal docs drafted
- ✅ 15 workflows spec'd + 6 JSON templates ready
- ✅ ADRs for all major decisions (5 ADRs)
- ✅ Deployment guide + monitoring + backup scripts
- 🟡 VPS not yet provisioned
- 🟡 Flutterwave account setup
- 🟡 WhatsApp Business API setup
- 🟡 Provider accounts + API keys
- 🟡 Custom domain registered