# Technical Design Document (TDD)

## Project Name: Sahayak

## 1. High-Level Architecture

The system follows an **Event-Driven Agentic Architecture**. It decouples the "Communication Layer" (WhatsApp) from the "Reasoning Layer" (LLM) and the "Action Layer" (Browser Automation).

### Flow Summary:

`User Voice` -> `WhatsApp Webhook` -> `ASR Service` -> `State Machine (Router)` -> `LLM (Slot Filling)` -> `Browser Agent (Playwright)` -> `Target Website`

## 2. Technology Stack (AWS & Open Source)

We leverage **AWS Cloud** for scalability and **Bhashini** for Indic-language supremacy.

| Component | Technology | Reasoning |
| :--- | :--- | :--- |
| **Interface** | WhatsApp Business API | Ubiquitous access for the target demographic (Track 3). Low bandwidth requirement. |
| **Compute** | **AWS EC2 (t3.medium)** | Hosting the FastAPI backend and Headless Browser (Playwright requires a persistent environment). |
| **LLM Inference** | **Amazon Bedrock** | Accessing **Anthropic Claude 3 Haiku** for fast, cost-effective reasoning and JSON extraction. |
| **Speech-to-Text** | **Bhashini API** / Whisper | Bhashini is the GOI standard for Indic languages. Fallback to Whisper Large-v3 on EC2. |
| **Orchestration** | **LangGraph** (Python) | Manages the conversational state (loops, branching logic) and tool calling. |
| **Browser Agent** | **Playwright** (Python) | Robust headless browser automation with "auto-wait" capabilities for slow sites. |
| **Storage** | **Amazon S3** | Temporary storage for incoming voice notes and outgoing screenshots. |
| **Database** | **Amazon DynamoDB** | Storing session state (conversation history, partial form data) with TTL (Time-to-Live). |

## 3. Detailed Component Design

### 3.1. Ingestion Layer (FastAPI + AWS)

* **Endpoint:** `/webhook/whatsapp` hosted on EC2.
* **Process:**
  1. Receives POST request from Meta.
  2. Extracts `media_id`.
  3. Downloads OGG audio to **Amazon S3**.
  4. Converts OGG to WAV using `ffmpeg` (pydub).

### 3.2. Reasoning Layer (The "Brain")

* **ASR:** The WAV file is sent to the Bhashini API.
  * *Input:* Audio
  * *Output:* Hindi/Tamil Text
* **Translation:** Text is translated to English for processing (optional, but improves LLM accuracy).
* **State Machine (LangGraph):**
  * **Node 1: Intent Classifier:** Is the user asking for a scholarship or a bank account?
  * **Node 2: Slot Filler:** Uses **Amazon Bedrock (Claude)** with `Instructor` library to force Pydantic JSON output.

```python
class UserDetails(BaseModel):
    name: str | None
    income: int | None
    village: str | None
```

  * **Node 3: Validator:** Checks if all fields in `UserDetails` are non-null. If null, generates a question to ask the user.

### 3.3. Action Layer (The "Hands")

* **Trigger:** Once `UserDetails` is complete.
* **Tool:** Playwright Script.
  * Launches Chromium instance.
  * Navigates to URL.
  * Uses **LLM Vision (Claude 3.5 Sonnet)** or Semantic Selectors to robustly find fields even if HTML IDs change.
  * `await page.fill('input[name="name"]', user_data.name)`
  * Takes a screenshot (`page.screenshot()`).

### 3.4. Response Layer

* **Visual:** Uploads screenshot to S3 -> Sends Presigned URL to WhatsApp.
* **Audio:** Uses Bhashini TTS (Text-to-Speech) to generate a voice note: "I have filled your name and income. Please check the photo."

## 4. Data Flow Diagram

```mermaid
graph TD
    User((User)) -- Voice Note --> WA[WhatsApp API]
    WA -- Webhook --> API[FastAPI on AWS EC2]
    API -- Audio --> S3[Amazon S3]
    API -- Audio --> ASR[Bhashini ASR]
    ASR -- Text --> Agent[LangGraph Agent]
    Agent -- Prompt --> Bedrock[Amazon Bedrock (Claude)]
    Bedrock -- JSON Entities --> DB[(DynamoDB State)]
    
    subgraph Action Loop
        DB -- Complete Data --> PW[Playwright Bot]
        PW -- Fill Form --> Web[Target Govt Portal]
        Web -- Screenshot --> PW
    end
    
    PW -- Image --> WA
    Agent -- Response Text --> TTS[Bhashini TTS]
    TTS -- Audio --> WA
```

## 5. Security & Privacy

* **Ephemeral Data:** Voice notes are deleted from S3 immediately after processing.
* **No PII Storage:** User personal data is only held in DynamoDB for the active session duration (TTL = 24 hours), then purged.
* **Encryption:** All data in transit (WhatsApp to Server, Server to Bedrock) is encrypted via TLS 1.3.

## 6. Scalability Strategy

* **Queueing:** For high loads, we will introduce Amazon SQS between the Webhook and the Processing Logic to handle bursts of users.
* **Serverless:** The Core Logic (ASR -> Bedrock) can be migrated to AWS Lambda to reduce costs, keeping only the Playwright worker on EC2.
