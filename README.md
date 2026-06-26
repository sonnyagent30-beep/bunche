# Bunche — Automated WhatsApp Proxy Retail Platform

> **Bunche** = Nigerian Pidgin for "plenty" / "bundle." Sell ISP, Residential, Mobile, and Datacenter proxies to Nigerian customers — via WhatsApp, powered by n8n automation.

**Status:** Planning complete → Ready to build

---

## What This Is

A fully automated proxy resale business that runs on WhatsApp:
- Customer messages `Order ISP UK 1` on WhatsApp
- n8n generates a Flutterwave payment link and sends it back
- Customer pays via transfer, card, USSD, or QR
- n8n receives webhook → calls provider API → delivers proxy credentials
- **Entire process: under 2 minutes. Fully automated.**

---

## Quick Facts

| | |
|--|--|
| **Business Model** | Proxy resale (no inventory, no own proxy infrastructure) |
| **Market** | Nigeria + West Africa |
| **Customers** | Social media managers, e-commerce sellers, forex traders, web scrapers |
| **Interface** | WhatsApp Business API |
| **Automation** | n8n workflow engine (self-hosted on VPS) |
| **Payments** | Flutterwave (NGN — transfer, card, USSD, QR) |
| **Database** | Google Sheets (Orders, Inventory, Customers, Providers) |
| **Proxy Supply** | IPRoyal, NodeMaven, Proxy-Seller (APIs) |
| **Margin** | 100–300% on all products |

---

## Directory Structure

```
Bunche/
├── README.md
├── LICENSE
├── .github/workflows/ci.yml
├── .n8n/workflows/          ← n8n JSON exports go here
├── docs/
│   ├── BUILD_PACKAGE.md     ← Master build guide (start here)
│   ├── TOOLS_CHECKLIST.md
│   ├── GOOGLE_SHEETS_SETUP.md
│   ├── FLUTTERWAVE_WHATSAPP_SETUP.md
│   ├── PROVIDER_SETUP_GUIDE.md
│   └── OPERATIONAL_RUNBOOK.md
├── legal/
│   ├── TERMS_OF_SERVICE.md
│   ├── PRIVACY_POLICY.md
│   └── ACCEPTABLE_USE_POLICY.md
└── workflows/
    └── WORKFLOW_SPECS.md   ← Step-by-step n8n build specs
```

---

## How to Build

See `docs/BUILD_PACKAGE.md` for the complete step-by-step guide.

**TL;DR:**
1. Get a VPS + domain
2. Set up Flutterwave + WhatsApp Business API
3. Create Google Sheets (4 tabs)
4. Open IPRoyal account + fund with ~$50
5. Install n8n on VPS via Docker
6. Build the 4 n8n workflows
7. Test end-to-end
8. Launch

---

## Current Status

- ✅ Research complete
- ✅ Strategy defined
- ✅ Legal docs drafted
- ✅ Build docs complete
- ✅ n8n workflow specs complete
- ✅ GitHub repo initialized
- 🟡 VPS not yet accessed
- 🟡 Waiting for account registrations

---

*Built with Sonny (Hermes Agent) for Dannion Creative Hub*
