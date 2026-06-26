# Bunche — n8n Workflow Specifications
*Zero-inventory, fully on-demand. Ollama + LiteLLM for natural language. No account creation — ever. Admin involved ONLY when human judgment is needed.*

---

## Refund Policy

**⚠️ STRICT: No refunds after proxy is generated and delivered.**

> Once a proxy IP is generated and sent to the customer, the sale is FINAL.
> Refunds are only considered in specific exempt cases listed below.

### Exemptions — Refunds ARE Allowed

| Case | Description | Who Approves |
|------|-------------|--------------|
| **Provider API failure** | Proxy never worked from the start | Admin — automatic |
| **Wrong IP delivered** | Wrong country/spec sent | Admin |
| **Fraudulent order** | Order placed with stolen payment | Admin |
| **Duplicate charge** | Same order charged twice | Admin — automatic |
| **Admin-approved exemption** | Exceptional circumstance | Admin only |

### Non-Exemptions — NO Refund

| Case | Reason |
|------|--------|
| "I changed my mind" | Proxy already generated and usable |
| "Platform banned my IP" | Outside our control — see replacement policy |
| "I don't need it anymore" | Service was delivered |
| "Found cheaper elsewhere" | Not our concern |
| Customer claims it doesn't work | Must prove it never worked from the start |
| Account banned by platform | Platform decision — not a proxy defect |

### Refund Process

```
Customer requests refund
        ↓
n8n checks: Is proxy fulfilled?
  → NO (not yet generated): Approve refund automatically
  → YES (already delivered): 
    → WhatsApp: "Refunds are not available after
                 a proxy is delivered. If your proxy
                 has a technical issue, please send
                 a screenshot and we'll review."
    → If customer insists: [ADMIN ALERT]
      → Admin reviews → decides if exemption applies
      → If approved: Admin initiates refund via command
```

---

## System Architecture

```
Customer WhatsApp Message
        ↓
[SECURITY LAYER] — Strip links, files, jailbreak attempts
        ↓
[LLM PARSING] — Ollama via LiteLLM → structured order intent
        ↓
[CHECK: Is this an admin command?] → YES → Route to Admin Workflow
        ↓
[CHECK: Is this routine or exception?] → ROUTINE → n8n handles automatically
                                          → EXCEPTION → Route to Admin Workflow
        ↓
Provider API → Proxy credentials
        ↓
PDF receipt generated
        ↓
WhatsApp delivery to customer
```

---

## The Core Principle

**Routine = fully automated. Exception = admin review.**

| What happens | Who does it |
|-------------|------------|
| New order → payment → delivery | n8n fully automated ✅ |
| Returning customer ordering | n8n fully automated ✅ |
| Lost proxy details (known number) | n8n fully automated ✅ |
| Refund request (proxy not yet sent) | n8n approves automatically ✅ |
| Refund request (proxy already sent) | n8n declines → admin reviews ⚠️ |
| Ban claim with screenshot | Admin review needed ⚠️ |
| Deceptive customer suspected | Admin review needed ⚠️ |
| New number claiming to be returning | Admin review needed ⚠️ |
| PIN/OTP fails 3x | Admin review needed ⚠️ |
| Admin commands | Admin handles ⚠️ |

---

## Admin WhatsApp Interface

Admin is Dannion — messaging Bunche from a known admin number.

### Admin Commands

| Command | What it does |
|---------|-------------|
| `Admin` | Show all pending actions |
| `Approve ORD-XXXXX` | Approve replacement/refund (if exemption applies) |
| `Reject ORD-XXXXX [reason]` | Reject with optional reason |
| `Block [phone] [reason]` | Block customer |
| `Unblock [phone]` | Unblock customer |
| `Details ORD-XXXXX` | Get full order details |
| `Refund ORD-XXXXX` | Initiate refund (admin exemption only) |
| `Pending` | List all pending admin actions |

---

## Workflow 1: Order Handler (WhatsApp Incoming)

**Trigger:** `POST /webhook/whatsapp-incoming`

```
Webhook Trigger (WhatsApp POST)
  ↓
Edit Fields: Extract from, msg_body, msg_id, timestamp
  ↓
[SECURITY LAYER] — Code Node: Strip links, files, injection
  ↓
[CHECK: Is admin number?] → YES → Route to Admin Workflow
  ↓
[CHECK: Existing customer?] → YES → Workflow 1a
                           → NO  → Workflow 1b
```

### Workflow 1a: Returning Customer

```
[EXISTING CUSTOMER]
  ↓
LLM Request → LiteLLM
  ↓
intent == "order":
  → Google Sheets Read: Lookup price
  → HTTP Request → Flutterwave POST /payments
  → Google Sheets Append: Pending_Orders (status: awaiting_payment)
  → WhatsApp: "Payment link sent. Amount: ₦[price]"
  → Webhook Response: HTTP 200

intent == "lost proxy details":
  → Google Sheets Read: Get proxy details
  → WhatsApp: Send proxy details directly
  → Webhook Response: HTTP 200

intent == "ban reported" OR "ip blocked":
  → Check: Was order within 24hrs?
    → YES: WhatsApp: "I'm sorry. Please send a screenshot of the ban message."
      → Wait for screenshot → Save → Google Sheets Update: "ban_pending_review"
      → [ADMIN ALERT] → Webhook Response: HTTP 200
    → NO: WhatsApp: "I'm sorry, replacement is only available within 24hrs of purchase. If your IP has issues, please describe them."
      → Webhook Response: HTTP 200

intent == "refund" OR "cancel":
  → Google Sheets Read: Get order status
    → Status == "awaiting_payment":
      → HTTP Request → Flutterwave: Cancel payment link
      → Google Sheets Update: Status = "cancelled"
      → WhatsApp: "✅ Order cancelled. No charges made."
    → Status == "fulfilled":
      → WhatsApp: "⚠️ Refunds are not available after a proxy is delivered. If there's a technical issue, please send a screenshot and we'll review."
      → If customer continues: [ADMIN ALERT]
      → Webhook Response: HTTP 200

intent == "help":
  → WhatsApp: Send menu
  → Webhook Response: HTTP 200

intent == "renew":
  → Show their active orders
  → Let customer select which to renew
  → Generate payment link
  → Webhook Response: HTTP 200

Default:
  → WhatsApp: LLM reply
  → Webhook Response: HTTP 200
```

### Workflow 1b: New Customer (or New Number)

```
[NEW CUSTOMER — first purchase]
  ↓
LLM Request → LiteLLM
  ↓
intent == "order":
  → Google Sheets Read: Lookup price
  → HTTP Request → Flutterwave POST /payments
  → Google Sheets Append: Pending_Orders (status: awaiting_payment)
  → WhatsApp: "Payment link sent. Amount: ₦[price]"
  → Webhook Response: HTTP 200

intent == "lost proxy details" OR "need my proxy":
  → Google Sheets Read: Check if phone in system
    → Phone NOT found:
      → WhatsApp: "Enter your PIN or OTP"
        → PIN entered → verify against stored hash
          → Match: Send proxy details
          → No match: "Incorrect PIN. Try again."
            → 3 failures → [ADMIN ALERT]
        → OTP chosen → Send code to WhatsApp
          → Code entered → verify
            → Match within 5 mins: Send proxy details
            → Expired/wrong: "Code expired. Try again."
              → 3 failures → [ADMIN ALERT]
  → Webhook Response: HTTP 200

intent == "help":
  → WhatsApp: Send menu
  → Webhook Response: HTTP 200

Default:
  → WhatsApp: LLM reply
  → Webhook Response: HTTP 200
```

---

## Workflow 2: Payment Confirmation (Flutterwave Webhook)

**Trigger:** `POST /webhook/flutterwave`

```
Webhook Trigger (Flutterwave POST)
  ↓
Code Node: Verify Flutterwave-Signature (HMAC SHA256)
  ↓
IF event !== "charge.completed" OR status !== "successful":
  → Respond 200 "ignored"
  ↓
Edit Fields: Extract tx_ref, amount, customer.phone, meta
  ↓
Google Sheets Read Row: Find order by tx_ref (Pending_Orders)
  ↓
IF order not found → Log error, respond 200
  ↓
IF Status !== "awaiting_payment":
  → Respond 200 "already processed"
  ↓
Google Sheets Update Row: Status = "paid_pending_fulfillment"
  ↓
Google Sheets Read Row: Check if customer exists (by phone)
  ↓
[IF NEW CUSTOMER — First purchase]
  → WhatsApp: "✅ Payment confirmed!"
  → WhatsApp: "Before your proxy is delivered,
               set up quick security for your next visit:
               
               1️⃣ PIN — Set a 4-digit PIN
               2️⃣ OTP — Get code via WhatsApp
               
               Reply '1' or '2' 👇"
  → Wait for reply → Store recovery choice
  → If PIN chosen: Wait for 4-digit PIN → Store hash
  → WhatsApp: "Got it! What's your name?"
  → Wait for name → Store in Customers sheet
  ↓
[IF RETURNING CUSTOMER]
  → No recovery setup — skip
  ↓
HTTP Request → Provider API (POST /orders)
  ↓
IF Provider API fails → Try backup provider
  Proxy-Seller → OkeyProxy → DataImpulse
    IF All fail:
      → Initiate refund → [ADMIN ALERT] → Mark failed
  ↓
Edit Fields: Parse credentials from provider response
  ↓
Google Sheets Update Row: Status = "fulfilled", Proxy Details = creds
  ↓
Google Sheets Append/Update: Add/update Customer
  ↓
[PDF GENERATION] — Generate receipt PDF
  ↓
WhatsApp Send Message: Proxy details + PDF receipt
  ↓
[IMPORTANT] → WhatsApp: "⚠️ No refunds once proxy is delivered. Replacement only within 24hrs if banned. See 'Help' for more."
  ↓
Webhook Response: HTTP 200
```

### Recovery Setup Messages (First Purchase Only)

**1 — Choose method:**
```
✅ Payment confirmed!

Before your proxy is delivered,
set up quick security for your
future visits (takes 10 seconds):

1️⃣ PIN — Set a 4-digit PIN
2️⃣ OTP — Get code via WhatsApp

Reply "1" or "2" 👇
```

**2a — PIN chosen:**
```
Reply with your 4-digit PIN 👇
```

**2b — OTP chosen:**
```
Got it! When you need to recover
your proxy details, a code will
be sent to this WhatsApp. ✅
```

**3 — Name:**
```
What should we call you?
(Your nickname or full name) 👇
```

---

## Workflow 3: Admin Command Handler

**Trigger:** `POST /webhook/whatsapp-incoming` (from admin number)

```
Webhook Trigger (WhatsApp POST)
  ↓
Edit Fields: Extract from, msg_body
  ↓
[CHECK: Is admin number?] → NO → Route to Workflow 1
  ↓
[ADMIN COMMAND PARSER] — Parse command
  ↓
"Pending":
  → Google Sheets Read: All rows where Status = "ban_pending_review" OR "admin_action_required"
  → WhatsApp: Send summary list of all pending actions

"Approve ORD-XXXXX":
  → Google Sheets Read: Get order details + action type
  → Switch: Route by action type
    "ban_pending_review":
      → HTTP Request → Provider API: Generate replacement proxy
      → Google Sheets Update: Status = "fulfilled", new proxy details, Replacement Count +1
      → WhatsApp (customer): "✅ Replacement approved! Your new proxy:"
        + New proxy details + PDF receipt
    "refund_pending":
      → HTTP Request → Flutterwave: Initiate refund
      → Google Sheets Update: Status = "refunded"
      → WhatsApp (customer): "✅ Refund processed. ₦{amount} in 5–7 days."
  → WhatsApp (admin): "✅ Done — ORD-XXXXX approved"

"Reject ORD-XXXXX [reason]":
  → Google Sheets Update: Status = "rejected", Notes = reason
  → WhatsApp (customer): "❌ We couldn't verify your claim."
  → WhatsApp (admin): "✅ Rejected — ORD-XXXXX. Reason: [reason]"

"Block [phone] [reason]":
  → Google Sheets Update: Customers — Blocked = TRUE, Blocked Reason = reason
  → WhatsApp (admin): "✅ Blocked [phone]. Reason: [reason]"

"Unblock [phone]":
  → Google Sheets Update: Customers — Blocked = FALSE, Blocked Reason = ""
  → WhatsApp (admin): "✅ Unblocked [phone]"

"Details ORD-XXXXX":
  → Google Sheets Read: Get full order details
  → WhatsApp (admin): Send full order summary

"Refund ORD-XXXXX":
  → Google Sheets Read: Check order status
    → If Status == "fulfilled":
      → WhatsApp (admin): "⚠️ Proxy already delivered for ORD-XXXXX.
                           Refunds are not available unless there's
                           a verified technical issue.
                           Reply 'Force-Refund ORD-XXXXX' to override."
    → If Status == "awaiting_payment" or "paid_pending_fulfillment":
      → HTTP Request → Flutterwave: Cancel/refund
      → Google Sheets Update: Status = "refunded"
      → WhatsApp (admin): "✅ Refunded — ORD-XXXXX"

"Force-Refund ORD-XXXXX":
  → Only works for admin — override for exceptional cases
  → HTTP Request → Flutterwave: Initiate refund
  → Google Sheets Update: Status = "refunded", Notes = "Admin override — exemption"
  → WhatsApp (admin): "✅ Force-refunded — ORD-XXXXX (exemption)"

Default:
  → WhatsApp (admin): "Unknown command. Type 'Pending' to see actions."
```

---

## Workflow 4: Ban Claim with Screenshot

```
Customer: "My IP was banned"
        ↓
Check: Was order within 24hrs?
  → NO: WhatsApp: "I'm sorry, replacement is only
               available within 24hrs of purchase.
               If your proxy has technical issues,
               please send a screenshot."
    → If provides screenshot: [ADMIN ALERT] → "Customer claiming technical issue — ORD-XXXXX"
    → Webhook Response: HTTP 200
  ↓
YES: Within 24hrs
  ↓
WhatsApp: "I'm sorry. Please send a screenshot
           of the ban/block message."
        ↓
Customer sends screenshot
        ↓
n8n saves screenshot → cloud storage
        ↓
Google Sheets Update: Status = "ban_pending_review"
                      Screenshot URL = [link]
                      Created At = now
        ↓
[ADMIN ALERT] → WhatsApp (admin):
"📋 New Ban Claim — ORD-XXXXX
 Customer: [name] — [phone]
 Order: [product] — [date of purchase]
 Screenshot: [view link]

 Reply: 'Approve ORD-XXXXX'
        or 'Reject ORD-XXXXX reason'"
        ↓
[Continues via Admin Command Handler above]
```

---

## Workflow 5: Provider APIs

### Proxy-Seller API (Primary for ISP + DC)

```
POST https://api.proxy-seller.com/v1/orders
Headers: Authorization: Bearer *** $credentials.proxyseller-api }}
Body: {"type": "isp", "country": "gb", "quantity": 1, "period": 30}

Response:
{
  "order_id": "PS-12345",
  "status": "active",
  "proxies": [{"ip": "203.0.113.42", "port": 8080, "username": "user", "password": "pass", "expires_at": "2026-07-26"}]
}
```

### OkeyProxy API (Residential)

```
POST https://api.okeyproxy.com/v1/order
Headers: Authorization: Bearer *** $credentials.okeyproxy-api }}
Body: {"type": "residential", "country": "global", "traffic": "5GB"}
```

### DataImpulse API (Mobile)

```
POST https://api.dataimpulse.com/v1/order
Headers: Authorization: Bearer *** $credentials.dataimpulse-api }}
Body: {"type": "mobile", "country": "us", "traffic": "5GB"}
```

### Fallback Chain

```
Proxy-Seller → Fails → OkeyProxy → Fails → DataImpulse → Fails
→ Initiate refund (proxy never delivered) → [ADMIN ALERT] → Mark failed
```

---

## Workflow 6: Refund Handler

**Trigger:** Flutterwave webhook (refund events)

```
Webhook Trigger
  ↓
Code Node: Verify signature
  ↓
IF event !== "refund.initiated" AND event !== "refund.completed":
  → Respond 200 "ignored"
  ↓
Edit Fields: Extract tx_ref, amount
  ↓
Google Sheets Read Row: Find order
  ↓
IF Status == "fulfilled":
  → HTTP Request → Provider API: revoke/cancel order
  ↓
Google Sheets Update Row: Status = "refunded"
  ↓
WhatsApp Send Message:
  "✅ Refund Processed. ₦{amount} for ORD-XXXXX in 5–7 business days."
```

---

## Error Workflow: Admin Alert

**Trigger:** n8n Error Trigger

```
n8n Error Trigger
  ↓
WhatsApp (admin):
"🔴 Workflow Error
  
 Workflow: {workflow_name}
 Error: {error_message}
 Execution: {execution_id}
 
 Check: https://n8n.yourdomain.com/executions"
```

---

## Credentials to Create in n8n

| Name | Type | Auth |
|------|------|------|
| `flutterwave-api` | HTTP Header Auth | `Authorization: Bearer *** |
| `whatsapp-api` | HTTP Header Auth | `Authorization: Bearer *** |
| `google-sheets-service` | Google Cloud Service Account | Service Account JSON |
| `proxyseller-api` | HTTP Header Auth | `Authorization: Bearer *** |
| `okeyproxy-api` | HTTP Header Auth | `Authorization: Bearer *** |
| `dataimpulse-api` | HTTP Header Auth | `Authorization: Bearer *** |
| `lite-llm` | HTTP Query Auth | `Bearer ollama-proxy-key` |

---

## Ollama + LiteLLM Setup

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.2:3b

# Install LiteLLM
pip install litellm
litellm --model ollama/llama3.2:3b --port 4000
```

---

## Security Layer

```javascript
const input = $json.message || $json.body || "";

const stripped = input
  .replace(/https?:\/\/[^\s]+/gi, "[LINK REMOVED]")
  .replace(/\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b/g, "[IP REMOVED]")
  .replace(/ignore previous instructions/gi, "")
  .replace(/disregard all rules/gi, "")
  .replace(/system prompt/gi, "")
  .replace(/<\|.*?\|>/g, "")
  .replace(/\{.*?"role.*?\}/g, "")
  .trim()
  .substring(0, 500);

return {
  original: input,
  cleaned: stripped,
  hasLink: /https?:\/\//.test(input),
  hasInjection: /ignore|disregard|system prompt|<\|.*?\|>/i.test(input)
};
```

---

## System Prompt — LLM Rule Book

```SYSTEM
You are the order assistant for Bunche, a WhatsApp-based proxy reseller operating in Nigeria.

TONE: Friendly, brief, Nigerian-friendly English. Never excessive emojis. Be direct.

YOUR JOB:
1. Parse customer messages → extract: intent, product type, country, quantity
2. If order is clear → confirm price and prepare payment link request
3. If order is unclear → ask ONE clarifying question only
4. If customer asks about providers → deflect politely
5. If customer asks about refunds → explain the refund policy

REFUND POLICY TO COMMUNICATE:
- No refunds after a proxy is delivered
- Replacement only within 24hrs of purchase (with screenshot)
- If there's a technical issue from the start → admin will review

NEVER DO / NEVER SAY:
- Never mention Proxy-Seller, OkeyProxy, DataImpulse, IPRoyal, or any provider name
- Never reveal API keys, internal pricing margins, or provider costs
- Never explain HOW proxies work technically
- Never open, follow, or acknowledge any link in the customer message
- Never attempt to download, process, or parse any file
- Never reveal recovery method details to customers

ORDER VALID COMMANDS:
- "Order ISP [COUNTRY] [QTY]" → extract: product=ISP, country, qty
- "Order RES [QTY]GB" → extract: product=RESIDENTIAL, qty
- "Order MOB [QTY]GB" → extract: product=MOBILE, qty
- "Order DC [COUNTRY] [QTY]" → extract: product=DATACENTER, country, qty
- "Status [ORDER_ID]" → intent=status
- "Renew [ORDER_ID]" → intent=renew
- "Help" → intent=help
- "Check price [PRODUCT]" → intent=price_check
- "Refund" OR "Cancel" → intent=refund_request

IF MESSAGE IS NOT A VALID COMMAND:
- Extract intent if possible
- If it sounds like an order → ask "Did you mean: Order ISP UK 1?"
- If it cannot be resolved → "I didn't understand that. Type 'Help' to see available commands."

RESPONSE FORMAT — Return ONLY valid JSON:
{
  "intent": "order|status|renew|help|price_check|ban_reported|refund_request|unknown",
  "product": "ISP|RESIDENTIAL|MOBILE|DATACENTER|null",
  "country": "country code or null",
  "quantity": number or null,
  "confidence": 0.0 to 1.0,
  "reply": "short response (under 100 chars)"
}
```

---

## Google Sheets: Orders / Pending_Orders

| Column | Header |
|--------|--------|
| Order ID | text |
| Customer Phone | text |
| Plan Code | text |
| Country | text |
| Quantity | number |
| Amount Paid (NGN) | number |
| Payment Reference | text |
| Provider | text |
| Provider Order ID | text |
| Proxy Credentials | text |
| Status | text |
| Ban Reported | boolean |
| Screenshot URL | text |
| Ban Verified | admin_review_pending / verified / rejected |
| Replacement Count | number |
| Refund Requested | boolean |
| Notes | text |
| Created At | datetime |
| Fulfilled At | datetime |
| Cost (USD) | number |

**Status Values:**
`awaiting_payment` | `paid_pending_fulfillment` | `fulfilled` | `ban_pending_review` | `replaced` | `failed` | `refund_pending` | `refunded` | `rejected` | `cancelled`

---

## Google Sheets: Customers

| Column | Header |
|--------|--------|
| Phone | text (primary key) |
| Name | text |
| Recovery Method | PIN or OTP |
| PIN Hash | text (bcrypt) |
| Total Orders | number |
| Lifetime Value (NGN) | number |
| Replacement Count | number |
| Last Order At | datetime |
| Support Notes | text |
| Blocked | boolean |
| Blocked Reason | text |
| Created At | datetime |

---

## Security Checklist

| Rule | Enforced Where |
|------|---------------|
| No URLs in customer messages | Security Stripper |
| No links opened by LLM | System prompt + n8n pre-check |
| No file downloads | System prompt + n8n pre-check |
| No provider names revealed | System prompt (LLM) |
| No injection prompts processed | Security Stripper + system prompt |
| LLM output validated as JSON | n8n validation node |
| LLM cannot execute actions | n8n acts on structured output only |
| Message length limited to 500 chars | Security Stripper |
| PIN stored hashed | bcrypt hash in Google Sheets |
| OTP expires in 5 minutes | Timestamp checked |
| Max 3 verification attempts | Counted before admin escalation |
| Admin only on exception | Admin Workflow triggered only on exception |
| No refund after delivery | Workflow enforces — admin override only |

---

## Workflow Activation Checklist

| Workflow | Trigger | When |
|----------|---------|------|
| Order Handler | WhatsApp Webhook | Always |
| Payment Confirmation | Flutterwave Webhook | On payment |
| Admin Command Handler | WhatsApp Webhook (admin number) | On admin message |
| Ban Claim | WhatsApp Webhook (within Order Handler) | On ban claim |
| Refund Handler | Flutterwave Webhook | On refund event |
| Error Alert | n8n Error Trigger | On any error |

---

## Testing

```bash
# Scenario 1: Returning customer orders
curl -X POST https://n8n.yourdomain.com/webhook/whatsapp-incoming \
  -H "Content-Type: application/json" \
  -d '{"entry":[{"changes":[{"value":{"messages":[{"id":"t1","from":"2349000000001","timestamp":"123","text":{"body":"Order ISP UK 1"}}]}}]}]}'

# Scenario 2: New customer, first purchase
curl -X POST https://n8n.yourdomain.com/webhook/whatsapp-incoming \
  -H "Content-Type: application/json" \
  -d '{"entry":[{"changes":[{"value":{"messages":[{"id":"t2","from":"2349000000002","timestamp":"123","text":{"body":"Order ISP UK 1"}}]}}]}]}'

# Scenario 3: Admin checks pending
curl -X POST https://n8n.yourdomain.com/webhook/whatsapp-incoming \
  -H "Content-Type: application/json" \
  -d '{"entry":[{"changes":[{"value":{"messages":[{"id":"t3","from":"2347032981049","timestamp":"123","text":{"body":"Pending"}}]}}]}]}'

# Scenario 4: Admin force-refund (exemption)
curl -X POST https://n8n.yourdomain.com/webhook/whatsapp-incoming \
  -H "Content-Type: application/json" \
  -d '{"entry":[{"changes":[{"value":{"messages":[{"id":"t4","from":"2347032981049","timestamp":"123","text":{"body":"Force-Refund ORD-2026-XXXXX"}}]}}]}]}'

# Scenario 5: Customer requests refund after delivery
curl -X POST https://n8n.yourdomain.com/webhook/whatsapp-incoming \
  -H "Content-Type: application/json" \
  -d '{"entry":[{"changes":[{"value":{"messages":[{"id":"t5","from":"2349000000001","timestamp":"123","text":{"body":"I want a refund"}}]}}]}]}'
```
