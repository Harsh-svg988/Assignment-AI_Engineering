# Sample Input / Output

Three worked examples showing how the workflow handles different ticket types.
Use **Example 1** as your main demo in the video — it shows the full happy path *and*
the human-escalation branch.

---

## Example 1 — Angry billing ticket → HUMAN ESCALATION

### Input (POST to the webhook)
```json
{
  "name": "Priya Sharma",
  "email": "priya@example.com",
  "message": "I have been charged TWICE this month and nobody is responding! This is the third time. If I don't get my money back today I am cancelling and disputing the charge with my bank."
}
```

### Triage Agent (AI) output
```json
{
  "category": "billing",
  "urgency": "high",
  "sentiment": "angry",
  "summary": "Customer was double-charged, demands refund today, threatening to cancel and dispute.",
  "is_common_question": false,
  "confidence": 0.95
}
```

### Routing decision
`urgency = high` → **Route by Urgency → Escalate to Human**

### Final record (logged to Sheet)
```json
{
  "action": "ESCALATED_TO_HUMAN",
  "alert": "⚠️ HIGH URGENCY ticket from Priya Sharma (priya@example.com). Sentiment: angry. Issue: Customer was double-charged... A human must review before replying.",
  "draftReply": ""
}
```
No auto-reply is sent — a human reviews first. ✅ *human-in-the-loop demonstrated.*

---

## Example 2 — Simple FAQ → AUTO-REPLY

### Input
```json
{
  "name": "Arjun Mehta",
  "email": "arjun@example.com",
  "message": "Hi, where can I download my past invoices? I just need them for my records, no rush."
}
```

### Triage Agent (AI) output
```json
{
  "category": "billing",
  "urgency": "low",
  "sentiment": "neutral",
  "summary": "Customer wants to know where to download past invoices.",
  "is_common_question": true,
  "confidence": 0.9
}
```

### Routing decision
`is_common_question = true` → **Route by Urgency → Draft Reply Agent**

### Draft Reply Agent (AI) output
```
Hi Arjun, happy to help! You can download all your past invoices anytime from
Account → Billing → Invoice History, where each one is available as a PDF. If you
don't see a particular invoice there, just reply here and our billing team will send
it across. Thanks for being a customer!
```

### Final record
```json
{ "action": "AUTO_REPLIED", "draftReply": "Hi Arjun, happy to help! ..." }
```
Email is sent automatically. ✅ *AI reasoning + tool use demonstrated.*

---

## Example 3 — Malformed ticket → FALLBACK (Manual Review)

### Input (missing message, bad email)
```json
{
  "name": "Test",
  "email": "not-an-email",
  "message": ""
}
```

### Routing decision
`2. Validate Input` sets `isValid = false` → **Is Input Valid? → Manual Review Queue**
(the AI is never even called — saves cost).

### Final record
```json
{
  "action": "MANUAL_REVIEW",
  "summary": "Missing or invalid name/email/message",
  "confidence": 0,
  "alert": "Ticket needs human triage (invalid input or low AI confidence)."
}
```
✅ *fallback / error handling demonstrated.*

---

## Quick test command (cURL)

Once your workflow is active, grab the **Test URL** from the Webhook node and run:

```bash
curl -X POST "PASTE_YOUR_WEBHOOK_TEST_URL_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Priya Sharma",
    "email": "priya@example.com",
    "message": "I have been charged TWICE this month and nobody is responding!"
  }'
```
