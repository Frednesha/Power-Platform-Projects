# Weekly Identity Audit — Teams Notification via Adaptive Card (Power Automate)

## Overview

This Power Automate flow sends a weekly Adaptive Card notification to a Microsoft Teams channel listing all new user accounts created in the last 7 days. The card is designed to support identity governance by prompting reviewers to verify that new accounts correspond to approved HR onboarding tickets.

---

## Use Case

In environments where user provisioning happens across multiple systems or teams, it can be easy for untracked accounts to slip through. This flow provides a lightweight, recurring audit mechanism by surfacing new accounts directly in Teams — where reviewers already work — without requiring access to the admin portal.

---

## Flow Summary

| Step | Action |
|------|--------|
| 1 | Scheduled trigger (weekly) |
| 2 | Query user accounts created in the last 7 days |
| 3 | Filter array to isolate new accounts |
| 4 | Join account list into a readable string |
| 5 | Post Adaptive Card to Teams channel |

---

## Adaptive Card JSON

Paste the following into the **Adaptive Card** field of the **"Post card in a chat or channel"** Teams action in Power Automate.

```json
{
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.4",
  "body": [
    {
      "type": "TextBlock",
      "size": "Medium",
      "weight": "Bolder",
      "text": "Weekly Identity Audit: New Accounts"
    },
    {
      "type": "TextBlock",
      "text": "The following accounts were created in the last 7 days. Please verify these match approved HR onboarding tickets.",
      "wrap": true
    },
    {
      "type": "FactSet",
      "facts": [
        {
          "title": "Total New Accounts:",
          "value": "@{length(body('Filter_array'))}"
        }
      ]
    },
    {
      "type": "TextBlock",
      "text": "@{outputs('Join')}",
      "wrap": true,
      "spacing": "Medium",
      "color": "Accent"
    }
  ]
}
```

### Dynamic Expressions Used

| Expression | Purpose |
|------------|---------|
| `@{length(body('Filter_array'))}` | Returns the count of new accounts from the Filter Array step |
| `@{outputs('Join')}` | Returns the formatted list of new account names/details from the Join step |

---

## Common Error & Fix

**Error:** `Action 'Post_card_in_a_chat_or_channel' failed: The specified Teams flowbot message's message body is invalid JSON.`

**Cause:** This error typically has two sources:

1. **Wrong JSON structure** — Wrapping the card in an `attachments` envelope when using the dedicated Adaptive Card field. The Teams connector in Power Automate handles the envelope automatically; adding it manually breaks the payload.

2. **Dynamic expressions resolving incorrectly** — If a referenced action name has changed or the expression output is null/unexpected, it can corrupt the JSON structure at runtime.

**Fix:**
- Use the plain card JSON (no `attachments` wrapper) in the Adaptive Card field.
- To isolate the issue, temporarily replace dynamic expressions with static strings and test. If the card posts successfully with static values, debug the expressions individually.

---

## Notes

- This card was built and tested using **Adaptive Card schema version 1.4**.
- The card is read-only (no actions/buttons). Action buttons for approve/flag workflows can be added using the `actions` array if needed in a future iteration.
- To preview and test card layouts before deploying, use the [Adaptive Cards Designer](https://adaptivecards.io/designer/) — note that Power Automate dynamic expressions (`@{...}`) will need to be replaced with static values for the designer to render correctly.

---

## Technologies Used

- Microsoft Power Automate
- Microsoft Teams (Post card in a chat or channel)
- Adaptive Cards v1.4
- Microsoft 365