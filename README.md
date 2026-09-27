# at-food-co-ai-response-workflow
AI-powered inbound lead response &amp; sentiment classification workflow for AT Food Co. using n8n, OpenAI gpt-4o-mini, conditional router nodes, and Slack human-in-the-loop review cards.
# 🐶 AT Food Co. — AI Lead Response Classifier & Automated Workflow
> **Inbound Lead Automation: Real-time Sentiment Analysis, Decision Routing, and Human-in-the-Loop Email Drafting**

[![Domain](https://img.shields.io/badge/Domain-AI%20Automation%20%7C%20Inbound%20Ops-orange)](#)
[![Stack](https://img.shields.io/badge/Stack-n8n%20%7C%20OpenAI%20%7C%20Slack%20API-blue)](#)
[![Methodology](https://img.shields.io/badge/Methodology-Human--In--The--Loop-green)](#)

---

## 📌 Executive Summary & Business Objective

* **The Challenge:** Following **AT Food Co.'s** B2B corporate wellness outreach, inbound replies from HR heads range from enthusiasm to price inquiries and outright rejections. Manually triaging replies and drafting personalized responses creates massive sales latency.
* **The Solution:** An intelligent **n8n workflow** that ingests incoming lead replies, uses `gpt-4o-mini` to classify sentiment (*Positive*, *Neutral*, *Negative*), routes decision logic, and prepares human-in-the-loop draft responses in Slack.
* **Core Impact:** Reduces lead response time from **24 hours to <5 minutes**, ensuring high-intent corporate prospects receive meeting links instantly.

---

## 🏗️ Technical Workflow Architecture

    [Inbound Reply Webhook] ──> [OpenAI Sentiment Node] ──> [Router / Switch Node]
                                 (Classifies Intent &         ├── POSITIVE  ─> Draft Meeting Email
                                  Drafts JSON Response)       ├── NEUTRAL   ─> Draft Info/Nurture Email
                                                              └── NEGATIVE  ─> Flag "Not Interested"
                                                                      │
                                                                      ▼
                                                          [Slack Approval Card]
                                                          (Human Review & Dispatch)

---

## 🔌 Node-by-Node Pipeline Specification

### Node 1: Inbound Webhook (`n8n-nodes-base.webhook`)
* **Function:** Listens for incoming email response payloads from prospective B2B clients.
* **Variables Ingested:** `Lead_Name`, `Company_Name`, `Email_Address`, `Reply_Message`.

### Node 2: OpenAI Sentiment & Intent Classifier (`n8n-nodes-base.openAi`)
* **Model Engine:** `gpt-4o-mini`
* **Execution Logic:** Injects the reply text into a structured JSON prompt template to determine sentiment and generate an appropriate draft response.

### Node 3: Router Node (`n8n-nodes-base.switch`)
* **Branch A (Positive):** Routes to draft a calendar booking link email.
* **Branch B (Neutral):** Routes to draft a nurture email or attach pricing collateral.
* **Branch C (Negative):** Suppresses email drafting and updates CRM lead status to `Not Interested`.

### Node 4: Slack Approval Stage (`n8n-nodes-base.slack`)
* **Function:** Formats the AI classification and draft email into a Slack Block Kit card with `[Approve & Send]` and `[Edit Draft]` buttons.

---

## 🤖 System Classification Prompt Template

> You are an AI Lead Qualification Assistant for AT Food Co. Corporate Wellness. Analyze the incoming lead reply and classify it.
>
> Lead Context:
> Name: {{Name}}
> Company: {{Company}}
> Incoming Reply: "{{Reply_Message}}"
>
> Tasks:
> 1. Classify Sentiment as exactly one of: "Positive", "Negative", or "Neutral".
> 2. Based on the classification, execute the following decision logic:
>    - POSITIVE: Draft a friendly meeting confirmation / booking link response.
>    - NEUTRAL: Draft an informative follow-up providing requested materials (case study/pricing) or agreeing to check back at their specified timeframe.
>    - NEGATIVE: Mark status as "Not Interested" and set "Do Not Contact" flag to True. Do not draft a sales pitch.
>
> Output JSON Format:
> {
>   "classification": "Positive | Negative | Neutral",
>   "suggested_action": "Short explanation of next step",
>   "drafted_response": "Drafted text for human review (or 'N/A' if Negative)"
> }

---

## 📊 Inbound Response Execution Table

| Lead Name | Classification | Suggested Next Action | AI Drafted Output (Awaiting Human Review) |
| :--- | :--- | :--- | :--- |
| **Rajesh Kumar** | Positive | Schedule Demo | *"Hi Rajesh, glad to hear that! You can pick a 15-min slot that works best for you here: [Calendly Link]. Looking forward to showing you AT Food Co.'s corporate perk program."* |
| **Priya Sharma** | Negative | Update CRM Status | Status marked as "Not Interested - Tech stack locked". No email drafted. |
| **Amit Patel** | Neutral | Send Collateral | *"Hi Amit, absolutely! I've attached our 1-page pricing sheet and a case study showing how a peer tech company boosted office culture with fresh pet perks."* |
| **Sneha Verma** | Neutral | Nurture in Q3 | *"Hi Sneha, completely understand. I've set a reminder on my end to touch base with you at the start of Q3. Best of luck with this quarter!"* |
| **Vikram Malhotra** | Positive | Schedule Meeting | *"Hi Vikram, Thursday at 2 PM works great on my end! I've sent over a calendar invite with the meeting link."* |

---

## 💬 Sample Slack Review Output Card

    📥 Inbound Lead Reply Analyzed!

    • Lead: Rajesh Kumar (InnovateX Systems)
    • AI Classification: POSITIVE (Confidence: 98%)
    • Suggested Action: Schedule Demo

    • AI Drafted Reply:
      "Hi Rajesh, glad to hear that! You can pick a 15-min slot that works best for you 
       here: [Calendly Link]. Looking forward to showing you AT Food Co.'s corporate perk program."

    • Action Required: [Approve & Send Email] | [Edit Draft in Slack]

---

## ⚙️ How to Import & Run This n8n Workflow

1. Download the `lead_response_workflow.json` file from this repository.
2. Open your **n8n instance** and click **Import from File**.
3. Configure your `OpenAI API Key` and `Slack OAuth Credentials`.
4. Send a test HTTP payload to the Webhook URL to verify classification routing.
