# WhatsApp Lead Sentiment and Psychological Intent Analysis Workflow

**Automate WhatsApp lead sentiment and intent analysis with n8n, OpenAI, and Google Sheets**

---

## Overview

This project automates the analysis of WhatsApp leads by leveraging an AI-driven workflow built in n8n.  
When a WhatsApp message arrives via webhook, the system automatically detects the psychological intent, emotional tone, and buying readiness of the lead.  
Results are scored and logged into a Google Sheet, allowing sales and CRM teams to prioritize high-value leads in real time.

---

## Architecture

### **Input**

- WhatsApp conversation text (received via webhook)
- Lead ID, phone number, or session data

### **Processing**

- Webhook node captures WhatsApp message payload
- OpenAI node analyzes tone, sentiment, and buying intent
- JavaScript node formats analysis and assigns a lead score
- Google Sheets node logs or updates the lead record

### **Output**

Each lead entry includes:
- Incoming message
- Detected psychological intent
- Sentiment (positive, neutral, negative)
- Lead readiness score (0–100)
- Timestamp

---

## Workflow Diagram

<img width="1819" height="753" alt="Screenshot 2025-11-22 055452" src="https://github.com/user-attachments/assets/36d4d184-baa2-4b5e-bb58-447e30c74237" />


```
Webhook → AI Agent → JavaScript (Lead Scoring) → Google Sheet (Append/Update)
```

---

## Detailed Workflow Description

### 1. Webhook Node
- **Trigger:** Receives new WhatsApp messages via POST request
- **Payload Example:**
  ```json
  {
    "from": "+15551234567",
    "message": "Hey, I saw your product demo. Looks interesting but I need a bit more info.",
    "timestamp": "2025-11-22T10:30:00Z"
  }
  ```

### 2. AI Agent Node
- Uses an OpenAI Chat Model (e.g., GPT-4) with a contextual prompt for intent, sentiment, and score extraction.
- **Prompt Example:**
  ```
  Analyze the following WhatsApp message and classify:
  - Sentiment: Positive / Neutral / Negative
  - Intent Type: Interested / Curious / Price-Sensitive / Uncertain / Disinterested
  - Confidence (0–1)
  - Suggested Lead Score (0–100)
  - Reason for the score
  Return JSON only.
  ```
- **Expected AI Output:**
  ```json
  {
    "sentiment": "Positive",
    "intent": "Interested",
    "confidence": 0.88,
    "score": 78,
    "rationale": "User expressed curiosity and partial readiness to buy."
  }
  ```

### 3. JavaScript Node
- Merges webhook input with AI output:
  ```javascript
  const lead = items[0].json;
  const analysis = items[1].json;
  return [
    {
      json: {
        phone: lead.from,
        message: lead.message,
        sentiment: analysis.sentiment,
        intent: analysis.intent,
        score: analysis.score,
        rationale: analysis.rationale,
        timestamp: lead.timestamp
      }
    }
  ];
  ```

### 4. Google Sheets Node
- **Operation:** Append (or update) row
- **Sheet:** `Leads_Intent_Analysis`
- **Columns:** Timestamp, Phone, Message, Sentiment, Intent, Score, Rationale

---

## Example Output in Google Sheets

| Timestamp           | Phone         | Message                                                                      | Sentiment | Intent     | Score | Rationale                                   |
|---------------------|--------------|------------------------------------------------------------------------------|-----------|------------|-------|---------------------------------------------|
| 2025-11-22 10:30    | +15551234567 | Hey, I saw your product demo. Looks interesting but I need a bit more info.  | Positive  | Interested | 78    | User expressed curiosity and partial readiness to buy. |

---

## Business Use Cases

- Lead prioritization for sales teams
- Psychological profiling for targeted follow-ups
- Sentiment tracking over time
- Automated CRM enrichment

---

## Technology Stack

- n8n — Workflow automation
- OpenAI Chat Model (GPT-4 or Azure OpenAI) — Sentiment and intent analysis
- Simple Memory (optional) — Conversation context storage
- Google Sheets API — Lead tracking
- WhatsApp Business API/Cloud API — Message webhook source

---

## Security and Compliance

- End-to-end HTTPS webhook connection
- No message content stored outside n8n or Google Sheets
- Configurable phone number anonymization or hashing
- Access control managed via Google Workspace permissions

---

## Setup Instructions

### Prerequisites

1. **n8n Account** — Self-hosted or cloud instance
2. **OpenAI API Key** — For GPT-4 or Azure OpenAI access
3. **Google Sheets API** — Credentials for append/update operations
4. **WhatsApp Business Account** — With access to Cloud API or Business API webhooks

### Step-by-Step Implementation

#### 1. Create n8n Workflow

- Open n8n and create a new workflow
- Add a **Webhook** node (trigger)
- Configure POST endpoint for WhatsApp messages

#### 2. Configure Webhook Node

- Set method to POST
- Generate webhook URL
- Share URL with WhatsApp API integration

#### 3. Add OpenAI Chat Model Node

- Connect to Webhook output
- Set Model: `gpt-4` or `gpt-3.5-turbo`
- Input prompt (see Prompt Example above)
- Add system message for context

#### 4. Add JavaScript Node

- Connect to AI output
- Paste the JavaScript code provided above
- Format fields for Google Sheets

#### 5. Configure Google Sheets Node

- Create sheet `Leads_Intent_Analysis` in Google Drive
- Add column headers: Timestamp, Phone, Message, Sentiment, Intent, Score, Rationale
- Connect Google Sheets credentials in n8n
- Set operation to "Append Row"
- Map fields from JavaScript node output

#### 6. Test Workflow

- Send a test WhatsApp message
- Verify webhook receives payload
- Check Google Sheet for new row
- Adjust AI prompt if needed

---

## Configuration Variables

### Environment Variables (Optional)

```
OPENAI_API_KEY=your_openai_key
OPENAI_MODEL=gpt-4
GOOGLE_SHEETS_ID=your_sheet_id
WHATSAPP_PHONE_NUMBER=+1234567890
```

### AI Prompt Customization

Modify the AI prompt to adjust intent detection:

```
Intent Types:
- Interested: Strong buying signals, ready to proceed
- Curious: Questions asked, exploring options
- Price-Sensitive: Concerns about cost, comparing alternatives
- Uncertain: Hesitant, needs more information
- Disinterested: Not engaged, unlikely to convert
```

---

## Monitoring and Optimization

### Best Practices

- Review lead scores weekly to calibrate AI performance
- Test AI model accuracy against historical conversions
- Track sentiment trends over time
- Adjust score thresholds based on sales team feedback

### Troubleshooting

| Issue | Solution |
|-------|----------|
| Webhook not receiving messages | Verify WhatsApp API configuration and n8n URL is public |
| AI model rate limits | Implement exponential backoff in n8n error handler |
| Google Sheets append fails | Check API credentials and sheet permissions |
| Inaccurate sentiment detection | Refine AI prompt with business-specific language examples |

---

## Advanced Features

### Extend Workflow With

- **Sentiment Trend Analysis** — Compare sentiment over multiple messages
- **Lead Scoring Refinement** — Weight scores by industry, product type, or buyer persona
- **CRM Integration** — Sync leads directly to HubSpot, Salesforce, or Pipedrive
- **Automated Follow-up** — Trigger personalized WhatsApp responses based on intent
- **Conversation History Storage** — Archive full conversations in Supabase or Firebase

---

## Compliance and Privacy

- **GDPR Compliance** — Implement data anonymization for EU leads
- **Phone Number Masking** — Hash or encrypt phone numbers in Sheets
- **Message Retention** — Set auto-delete policy (e.g., 30 days)
- **Audit Logs** — Track who accessed lead data

---

## Support and Resources

- n8n Documentation: https://docs.n8n.io
- OpenAI API Reference: https://platform.openai.com/docs
- Google Sheets API: https://developers.google.com/sheets/api
- WhatsApp Business API: https://developers.facebook.com/docs/whatsapp

**Last Updated:** November 22, 2025
