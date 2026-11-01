# AI Email Triage & Auto-Reply Workflow (n8n)

An automated customer support and lead-triage pipeline built in **n8n**[cite: 1]. This workflow monitors an inbound IMAP inbox, classifies and extracts structured data from emails using **Groq LLMs**, logs tickets into an **n8n Data Table**, alerts human agents for high-priority/sensitive emails, and automatically drafts and sends replies for standard enquiries[cite: 1].

## Workflow Architecture

1. **Email Trigger (IMAP):** Listens for incoming emails in real time[cite: 1].
2. **Normalize Email (Set Node):** Extracts and cleans `from_email`, `from_name`, `subject`, plain-text `body`, `received_at` (ISO), and `message_id`[cite: 1].
3. **Skip Automated Mail (Filter):** Prevents self-loops by filtering out emails sent from the support address itself[cite: 1].
4. **Email Analyze (LangChain LLM + Groq `openai/gpt-oss-20b` + Structured Output Parser):** 
   - Detects prompt injection and spam[cite: 1].
   - Classifies emails into: `Spam`, `Refund`, `Complaint`, `Meeting`, `Existing Client`, `Quote Request`, `Enquiry`, `Job Application`, `Vendor Pitch`, or `Other`[cite: 1].
   - Determines `priority` (`Low`, `Normal`, `High`, `Urgent`), `sentiment`, `language`, and whether human intervention (`requires_human`) is needed[cite: 1].
5. **Save Ticket (n8n Data Table):** Logs the parsed email metadata and triage status (`Spam`, `Needs Human`, or `New`)[cite: 1].
6. **Routing (`Not Spam` -> `Needs Human?`):**
   - **True (`Needs Human`):** Sends an immediate summary alert via SMTP (**Notify Me**) to the team inbox[cite: 1].
   - **False (Standard Enquiry):** Passes the triage data to **Draft Reply** (`openai/gpt-oss-120b` via Groq) to generate a plain-text, helpful response, sends it via **Send Reply (SMTP)**, and marks the ticket status as `Replied` in **Update Ticket**[cite: 1].

## Prerequisites & Setup

To import and run this workflow in your own n8n instance:

1. **Import Workflow:** In n8n, go to **Workflows** -> **Import from File** and select `Email Automation workflow.json`[cite: 1].
2. **Configure Credentials:**
   - **IMAP Account:** Attach your inbound email credentials to the `Email Trigger (IMAP)` node[cite: 1].
   - **SMTP Account:** Attach your outbound email credentials to the `Notify Me` and `Send Reply` nodes[cite: 1].
   - **Groq API:** Add your Groq API key to `Groq Chat Model` and `Groq Chat Model1`[cite: 1].
3. **Set Up n8n Data Table:**
   - Create an n8n Data Table named `Save Ticket` with the following columns: `from_email`, `message_id`, `customer_name`, `subject`, `body`, `received_at`, `category`, `priority`, `sentiment`, `summary`, `customer_request`, `requires_human` (boolean), `human_reason`, `status`, `reply_text`, and `replied_at`[cite: 1].
   - Relink the table in both the `Save Ticket` and `Update Ticket` nodes[cite: 1].
