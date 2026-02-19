# n8n Automation Workflows

**Automation & Agentic Tools - 2024–Present**

[Back to Portfolio](../)

---

## Overview

Two workflows built for real-world use—one for a small business client, one as a personal tool. Both run on self-hosted n8n via Docker and use AI alongside Google Sheets as a lightweight database. The goal in both cases: eliminate repetitive manual work entirely, with enough intelligence built in to handle edge cases without human intervention.

---

## Invoice Logging Automation

*Client project — local optometry practice*

<img src="/assets/img/invoice_workflow.png" alt="Invoice logging n8n workflow" style="width:80%;">

*n8n workflow — automated invoice parsing, routing, and logging pipeline*

The practice was manually entering supplier invoices into spreadsheets daily—slow, error-prone, and eating into staff time. This workflow watches a Google Drive folder for new PDFs, extracts invoice data using an AI vision model, routes it by supplier, deduplicates entries, and logs everything to the correct sheet automatically.

**The goal**: Staff should only ever see invoices that genuinely need attention. Everything else is handled silently in the background.

---

### How It Works

**1. Trigger on new upload**  
A scheduled trigger polls a monitored Google Drive folder. When a new PDF invoice appears, the workflow kicks off and downloads the file.

**2. AI document extraction**  
The PDF is passed to an AI vision model which extracts structured data—line items, totals, dates, and vendor identity. A confidence score gates the output: low-confidence results are flagged immediately rather than silently logged with bad data.

**3. Route by vendor**  
The workflow identifies the supplier and routes the extracted data to the correct Google Sheet tab. If the sheet doesn't exist yet for that supplier, it's created automatically.

**4. Deduplication check**  
Before appending, existing entries are checked to prevent double-logging the same invoice across multiple runs.

**5. File management & alerts**  
Successfully processed invoices are moved to an archived folder. Any file that requires human review triggers an email alert with context on why it was flagged—so nothing falls through the cracks silently.

---

### Why It Matters

Turned a daily manual data-entry task into a fully automated pipeline. The AI extraction layer means it's not just moving files around—it's actually reading and understanding the documents. The confidence gating and alert system mean the failure mode is "someone gets an email" rather than "bad data enters the spreadsheet unnoticed."

**Tech:** n8n · Google Drive API · Google Sheets API · AI Document Parsing (vision model) · Docker

---

## Gift Sale Tracker

*Personal project — hands-off price monitoring with SMS alerts*

<img src="/assets/img/gift_tracker_workflow.png" alt="Gift sale tracker n8n workflow" style="width:80%;">

*n8n workflow — AI product identification, multi-store price checking, and SMS alert pipeline*

Built to solve a specific annoyance: wanting to buy something as a gift but not wanting to pay full price, which means either forgetting about it or compulsively checking store pages. You add an item to a tracking sheet—even a vague description—and the workflow handles the rest. It figures out what you actually mean, finds it across multiple stores, and texts you when the price drops.

**The goal**: Add something once, forget about it, get a text when it's worth buying.

---

### How It Works

**1. Scheduled scan**  
The workflow runs on a schedule, reading a Google Sheet that acts as a simple wishlist—product descriptions, target prices, and tracking status.

**2. AI product identification**  
Gemini interprets the item description, handling vague or informal names, and identifies the most likely specific product. A second AI pass confirms the match before the workflow proceeds—reducing false positives on ambiguous searches.

**3. Multi-store price lookup**  
The identified product is searched across multiple retailers. Current prices are fetched and compared against the tracked target price stored in the sheet.

**4. State management**  
A backend sheet tracks what's already been alerted, preventing duplicate notifications. New products are registered on first run; subsequent runs only flag genuine price changes or new drops.

**5. SMS alert**  
When a tracked item hits or drops below the target price, a text message is sent with the product name, current price, and store. No app to check, no email to miss—just a text when it matters.

---

### Why It Matters

The interesting part isn't the price checking—it's the AI identification layer. Product names in the real world are messy: different retailers list the same item differently, and what you write in a tracking sheet rarely matches an exact product title. Using Gemini to interpret intent and confirm the match before doing anything else means the workflow works on real-world input, not just clean, perfectly formatted queries.

**Tech:** n8n · Google Gemini · Google Sheets API · SMS (Twilio) · Docker

---

[Back to Portfolio](../)
