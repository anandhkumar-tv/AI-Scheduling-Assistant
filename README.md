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

## Features

- **Natural Language Understanding**: Interprets meeting requests from email content
- **Multi-day Search**: Searches for available slots across date ranges (e.g., "next week")
- **Time Zone Aware**: Handles attendees in different time zones
- **Conflict Resolution**: Analyzes calendar conflicts and proposes rescheduling options
- **Priority-based Scheduling**: Evaluates meeting importance and suggests priority overrides when needed
- **Google Calendar Integration**: Fetches real calendar data from attendees

## Installation

### Prerequisites

- Python 3.9+
- Access to a vLLM server running Llama 3.1
- Google Calendar API credentials

### Setup

1. Install the required dependencies:

```bash
pip install -q --upgrade \
    langchain \
    langchain-core \
    langchain-community \
    langgraph \
    langsmith \
    langchain-openai \
    dateparser \
    pytz \
    google-api-python-client \
    google-auth-oauthlib \
    requests \
    flask
```

2. Place Google Calendar API credentials in the `IISc_Google_Calendar_Keys/` directory

3. Configure the LLM server URL and model path in the notebook (Cell 2)

## Usage

### Starting the Server

1. Run the Jupyter notebook cells sequentially
2. The Flask server will start in a background thread on port 5000

### API Endpoint

Send POST requests to `/receive` with JSON payloads in this format:

```json
{
    "Request_id": "unique-request-id",
    "Datetime": "2025-07-18T14:00:00Z",
    "Location": "Virtual / Global Call",
    "From": "organizer@example.com",
    "Attendees": [
        {"email": "attendee1@example.com"},
        {"email": "attendee2@example.com"}
    ],
    "Subject": "Meeting Subject",
    "EmailContent": "Natural language meeting request goes here."
}
```

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

## Testing

The notebook includes a comprehensive test suite with scenarios like:
- All attendees available
- Specific attendees busy
- All attendees busy
- Priority-based scheduling decisions

## License

[Insert appropriate license information]

## Contributing

[Insert contribution guidelines if applicable]
