# AI Scheduling Assistant

An intelligent calendar management system that uses LLMs to schedule meetings based on natural language requests and calendar availability.

## Overview

The AI Scheduling Assistant is a sophisticated system that can:

1. Parse natural language meeting requests from emails
2. Extract key information like meeting duration, date ranges, and time preferences
3. Check calendar availability across multiple attendees
4. Find optimal meeting slots or propose intelligent conflict resolutions
5. Generate responses with scheduling recommendations

The system leverages LangChain, LangGraph, and local LLMs (specifically Llama 3.1) to create an agentic workflow that handles the entire scheduling process.

## 🛠️ Core Architecture

This solution was built using a modern, professional AI engineering stack designed for creating robust and scalable agents.

*   **Agentic Framework - LangGraph:** The heart of the solution is a stateful agent built with LangGraph. This allows for complex, cyclical reasoning—if a conflict is found, the agent can loop, find alternatives, and re-evaluate, perfectly mimicking a human thought process.

*   **High-Performance NLU - Llama 3.1 & vLLM:** Natural Language Understanding is powered by Meta's Llama 3.1-8B-Instruct model, served via the high-throughput vLLM inference server on the MI300 GPU. This ensures fast and accurate extraction of user intent.

*   **Data Integrity - Pydantic:** All data structures, especially the agent's state, are defined and validated using Pydantic. This eliminates an entire class of runtime errors and ensures data consistency throughout the workflow.

*   **Advanced Capability - Multi-Timezone Intelligence:** The agent's `find_available_slots` tool is designed to solve the difficult problem of scheduling for global teams. It performs the following logic:
    1.  Identifies the local timezone of each attendee (e.g., India, UK, US).
    2.  Calculates the standard working hours (9 AM - 6 PM) for each person in their local time.
    3.  Converts all local work-hour windows to a universal standard (UTC).
    4.  Calculates the **intersection** of all these UTC windows to find the "Golden Window" when everyone is at their desk.
    5.  Searches for an open meeting slot *only within this Golden Window*, ensuring a proposed time is reasonable for all participants.

---

## ⚙️ Setup and Execution Guide

Follow these steps to run the AI Scheduling Assistant.

### 1. Prerequisites
- Ensure you have access to the MI300 GPU Instance.
- Clone the project repository. Ensure the `Keys/` directory is present and contains valid Google Calendar tokens for `userone.amd@gmail.com`, `usertwo.amd@gmail.com`, and `userthree.amd@gmail.com`.

### 2. Start the vLLM Server
Open a new terminal in the Jupyter Lab environment and launch the Llama 3.1 model. Wait for the `Application startup complete.` message before proceeding.

HIP_VISIBLE_DEVICES=0 vllm serve Models/meta-llama/Meta-Llama-3.1-8B-Instruct \
        --gpu-memory-utilization 0.9 \
        --swap-space 16 \
        --disable-log-requests \
        --dtype float16 \
        --max-model-len 2048 \
        --tensor-parallel-size 1 \
        --host 0.0.0.0 \
        --port 4000 \
        --distributed-executor-backend "mp"



3. Run the Submission Notebook
Open the Submission.ipynb notebook.
Run all cells from top to bottom.
Cell 0 will install any necessary Python dependencies. You must restart the kernel after this cell completes.
Cells 1-7 define the agent's architecture, tools, and the Flask API.
Cell 8 starts the Flask server in the background. The server will be listening for requests on port 5000.
The subsequent cells will run a comprehensive suite of tests, proving the agent's functionality.


📡 API Endpoint for Testing
The agent is exposed via a POST request to the /receive endpoint.
Method: POST
URL: http://<YOUR_INSTANCE_IP>:5000/receive
(Find your instance's public IP address in the startup log of the Flask server in Cell 8)
Headers: Content-Type: application/json
Body (raw JSON): Provide the input JSON in the specified format.



Sample Request Body:

{
    "Request_id": "api-test-001",
    "Datetime": "2025-07-21T10:00:00Z",
    "Location": "Virtual / Cross-Timezone",
    "From": "userone.amd@gmail.com",
    "Attendees": [
        {"email": "usertwo.amd@gmail.com"},
        {"email": "userthree.amd@gmail.com"}
    ],
    "Subject": "Global Sync for Q3 Planning",
    "EmailContent": "Hi team, let's connect for our Q3 planning session. Can we meet for an hour sometime next week?"
}

### Response Format

The API returns a structured JSON response with:

- Meeting details (subject, duration, attendee emails)
- Proposed time slot with start and end times (if available)
- Attendee calendar information
- Conflict resolution details (if applicable)
- Metadata about the scheduling status

## System Architecture

The assistant follows a graph-based workflow:

1. **Initialize State**: Gather user timezone information
2. **Extract Details**: Parse the email with LLM to understand meeting requirements
3. **Search for Slots**: Find available time slots across the requested date range
4. **Conflict Resolution**: If no slots are available, analyze conflicts and suggest alternatives
5. **Format Response**: Generate the final structured output

## Advanced Features

### Conflict Resolution

When scheduling conflicts arise, the system:

1. Classifies conflicting meetings as "reschedulable," "negotiable," or "immovable"
2. Calculates a reschedulability score for each conflict
3. Determines the priority of the requested meeting
4. Generates negotiation strategies and alternative suggestions
5. Creates professionally-worded negotiation messages

### Multi-day Search

The system efficiently searches across multiple days by:

1. Extracting date ranges from natural language (e.g., "next week")
2. Iterating through business days in the range
3. Finding the earliest available slot that works for all attendees
4. Falling back to conflict resolution when needed

