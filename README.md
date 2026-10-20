# Customer Onboarding Orchestrator

An AI-powered customer onboarding workflow built in n8n. A supervisor agent classifies each new customer and routes them to the right flow: individual, small business, or enterprise. Enterprise requests need human approval on Telegram before anything is saved.

## Architecture

```
Webhook → Supervisor Agent → Parse Route → Route Customer (Switch)
   ├── Individual     → Welcome Agent → Parse → Gmail → Supabase → Telegram → Response
   ├── Small Business → Business Welcome Agent → Parse → Gmail → Supabase → Telegram → Response
   └── Enterprise     → Telegram Approval → Check Decision
                           ├── Approved → Supabase → Response
                           └── Rejected → Telegram Notify → Response
```

## Features

- **Supervisor agent** classifies customers into three routes using OpenAI, with structured JSON output
- **Specialist agents** write personalized welcome emails for each customer type
- **Human-in-the-loop approval** for enterprise leads via Telegram "Send and Wait"
- **Supabase** stores every onboarded customer with route, reason, and email subject
- **Telegram notifications** alert the team on each new customer
- **Webhook API** returns a JSON response to the caller

## Tech Stack

- n8n (workflow automation)
- OpenAI `gpt-5-mini` (supervisor and email agents)
- Gmail API (welcome emails)
- Supabase (customer database)
- Telegram Bot API (team alerts and approvals)

## Setup

1. Import the workflow JSON into n8n.
2. Add credentials: OpenAI, Gmail OAuth2, Supabase, and Telegram.
3. Create the `customers` table in Supabase:

```sql
create table customers (
  id bigint generated always as identity primary key,
  name text,
  email text,
  phone text,
  company text,
  route text,
  reason text,
  welcome_subject text,
  created_at timestamptz default now()
);
```

4. Replace the Telegram `chatId` with your own chat ID.
5. Activate the workflow and send a POST request to the webhook.

## Sample Request

```json
{
  "name": "Bilal Traders",
  "email": "customer@example.com",
  "phone": "+923009876543",
  "company": "Bilal Traders Pvt Ltd",
  "customer_type": "small_business",
  "message": "We run 3 shops and need automated invoice reminders and customer follow-ups."
}
```

## Project Notes

- Built as part of my AI Automation portfolio.
- Route values use `small_business` with underscores. Telegram messages must set Parse Mode to None, or the underscore breaks formatting.
