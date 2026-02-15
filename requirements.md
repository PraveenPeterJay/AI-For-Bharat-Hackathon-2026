# Product Requirements Document (PRD)

## Project Name: Sahayak (The Digital Caseworker)

### Track: AI for Communities, Access & Public Impact

## 1. Problem Statement

**The "Time Tax" of Bureaucracy:**

Accessing essential government services (scholarships, ration cards, crop insurance) and banking facilities requires filling out complex, English-heavy digital forms. This creates a massive barrier for:

* **300M+ Indians** with low literacy.
* **Rural populations** with limited digital confidence ("Technophobia").
* **Visually impaired** individuals who cannot navigate visual CAPTCHAs or small UI elements.

Current solutions (screen readers, translation widgets) only change the *output* language but do not reduce the *cognitive load* of the input process.

## 2. Solution Overview

**"Sahayak"** is a Voice-First, Multilingual AI Agent that acts as a human caseworker. Instead of forcing the user to learn the UI, the AI learns the form.

* **Input:** User speaks naturally via WhatsApp in their local dialect (e.g., "I need to apply for the pre-matric scholarship").
* **Process:** The AI engages in a conversation to gather details (Name, Income, ID numbers).
* **Action:** The AI autonomously drives a headless browser to fill out the actual government/banking portal in real-time.

## 3. User Personas

* **Ramesh (The Farmer):** Needs crop insurance but cannot navigate the English web portal. He uses WhatsApp daily.
* **Sunita (The Student):** Eligible for a scholarship but is intimidated by the complex 5-page application form.
* **NGO Field Worker:** Needs to register 50 villagers for ID cards quickly without typing manually on a laptop.

## 4. Functional Requirements

### 4.1. Voice Interface

* **FR-01:** System must accept voice notes via WhatsApp.
* **FR-02:** System must support Hindi, Tamil, Telugu, and Hinglish (mixed code).
* **FR-03:** System must provide audio feedback (TTS) to confirm actions.

### 4.2. Intelligent Slot Filling

* **FR-04:** AI must extract entities (Name, DOB, Aadhar) from unstructured speech.
* **FR-05:** AI must identify *missing* information and ask clarifying questions (e.g., "You didn't mention your date of birth. Please say it.").
* **FR-06:** System must validate data format (e.g., ensure PAN card is 10 chars) before attempting form submission.

### 4.3. Autonomous Form Execution

* **FR-07:** System must map extracted data to specific HTML selectors on the target website.
* **FR-08:** System must handle basic form interactions: text inputs, dropdowns, radio buttons, and checkboxes.
* **FR-09:** System must generate a screenshot of the filled form for user verification.

## 5. Non-Functional Requirements (NFR)

* **Latency:** Voice-to-Response time should be under 5 seconds to maintain conversational flow.
* **Reliability:** The browser agent must handle "Auto-Waiting" for slow government server responses.
* **Data Privacy:** User data (PII) must be processed in ephemeral memory and deleted after the session ends.
* **Accessibility:** Must work on 2G/3G networks (low bandwidth) via the WhatsApp interface.

## 6. Constraints & Assumptions

* **Constraint:** The target website must not use advanced anti-bot measures (like Cloudflare Turnstile) for the prototype.
* **Assumption:** The user has a valid WhatsApp account.
* **Assumption:** The target form is publicly accessible without 2FA (for the demo scope).
