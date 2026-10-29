# Instagram Payment Lead Automation (ManyChat)

Goal: when a lead comments or DMs "payment" on Instagram, collect their property/price and down payment, email Joseph the details, follow up on timeline after a delay, then hand off to the existing homebuyer guide automation.

No AI interpretation step. Joseph reviews every lead's numbers and sends the real payment himself, so the bot only needs to capture raw answers, not parse them. This is what replaced the original ManyChat AI-builder version, which was unreliable.

Flowchart: https://claude.ai/artifact/AAk6GVinGM4kmPk7Rm2QEK

## Entry points

Both lead into the same flow below.

- Comments Growth Tool, keyword "payment", on whichever posts/reels this should run on
- Keyword Rule, DM contains "payment"

## Flow

Guard condition, first block in the flow:
- Condition: does contact have tag `payment_flow_active`?
  - Yes -> send "Still putting that together for you, hang tight!" and end
  - No -> continue

1. **Opener** (Content block)
   Text: "Thanks for reaching out! I'll put together your monthly payment, just need a couple details. Tap below to get started."
   Button: "Show me my payment"
   Action on entry: add tag `payment_flow_active`

2. **Question: price/property**
   Text: "What's the purchase price, or the property you've got in mind?"
   Save reply to custom field `lead_property_price` (text)

3. **Question: down payment**
   Text: "Got it. And what are you thinking for a down payment, percent or dollar amount works."
   Save reply to custom field `lead_down_payment` (text)

4. **Confirmation** (static copy, no AI call)
   Text: "Perfect, I've got what I need. I'll get your numbers together and send it over soon."
   Actions: remove tag `payment_flow_active`, add tag `payment_info_received`

5. **Send Email** action
   To: Joseph
   Subject: `New payment lead: {{lead_property_price}}`
   Body: contact name/IG handle, `{{lead_property_price}}`, `{{lead_down_payment}}`, timestamp

6. **Smart Delay**: 1 hour

7. **Question: timeline**
   Text: "By the way, are you looking to get into something soon, or just exploring for now?"
   Save reply to custom field `lead_timeline`

8. **Handoff**
   Fires the existing guide automation off tag `payment_info_received` or the `lead_timeline` field being set, whichever it already listens for.

## Cleanup automation (separate, scheduled)

Strip `payment_flow_active` off any contact who's had it 24+ hours without finishing, so a lead who ghosts mid-flow isn't permanently locked out of retriggering by commenting or DMing again.

## Fields and tags

| Name | Kind | Holds |
|---|---|---|
| `payment_flow_active` | tag | Set while a lead is mid-flow. Blocks re-entry, cleared on finish or 24h stale. |
| `payment_info_received` | tag | Set once both fields below are captured. The guide automation and the stale-check key off this. |
| `lead_property_price` | field, text | Raw reply to the price/property question, unparsed. |
| `lead_down_payment` | field, text | Raw reply to the down payment question, unparsed. |
| `lead_timeline` | field, text | Raw reply to the timeline question, asked 1 hour later. |

## Open items

- Delay before the timeline question: currently 1 hour (inside Instagram's 24h messaging window)
- Retry cap: n/a now that there's no AI clarifying loop, each question is asked once
- Guide delivery: handled by an existing external trigger, not part of this build
