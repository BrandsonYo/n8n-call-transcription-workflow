# n8n Call Transcription & Analysis Workflow

Automatically transcribes recorded phone calls and extracts structured insights using AssemblyAI and Claude. Designed for sales teams or consultants who want searchable, analysed call records without manual note-taking.

---

## What it does

```
Call recording (AWB/MP3) uploaded to Google Drive
  → n8n Google Drive trigger fires
  → match caller phone number to lead database
  → send recording to AssemblyAI (transcription + speaker diarization)
  → poll until transcript is ready
  → send transcript to Claude for structured analysis
  → log results to Google Sheets (summary, pain points, interest level, next steps)
  → send SMS/email summary notification
  → move recording to processed/ folder
```

Produces structured data per call: summary, pain points, tools mentioned, interest level, suggested next steps, and objections. All logged to a Google Sheet for review.

---

## Stack

- **n8n** — automation platform
- **AssemblyAI** — speech-to-text with speaker diarization
- **Claude (Anthropic)** — call analysis and structured extraction
- **Google Drive** — recording storage and trigger
- **Google Sheets** — call log and CRM

---

## Setup

### 1. Prerequisites

- n8n instance
- AssemblyAI account (free tier works for low volume)
- Anthropic API key
- Google account with Drive and Sheets access

### 2. Google Drive folder structure

Create two folders in Google Drive:
- `Call Recordings/incoming/` — drop recordings here to trigger processing
- `Call Recordings/processed/` — workflow moves files here after completion

Note the folder IDs from the URL (`/folders/FOLDER_ID`).

### 3. Recording format

The workflow accepts AWB (AMR Wideband), MP3, M4A, and WAV. AssemblyAI handles conversion natively — no ffmpeg required.

Filename convention used by the workflow: `+PHONENUMBER-YYYYMMDDHHMM.awb`

Example: `+6421XXXXXXX-2603111200.awb`

The phone number in the filename is matched against your lead database to identify the caller.

### 4. Credentials (set up in n8n UI)

| Credential name | Type |
|---|---|
| Google Drive account | Google Drive OAuth2 |
| Google Sheets account | Google Sheets OAuth2 |

### 5. Environment variables

```bash
cp .env.example .env
```

| Variable | Where to find it |
|---|---|
| `ASSEMBLYAI_API_KEY` | assemblyai.com/dashboard |
| `ANTHROPIC_API_KEY` | console.anthropic.com |
| `GOOGLE_DRIVE_INCOMING_FOLDER_ID` | Google Drive URL |
| `GOOGLE_DRIVE_PROCESSED_FOLDER_ID` | Google Drive URL |
| `GOOGLE_SHEETS_SPREADSHEET_ID` | Google Sheets URL |

### 6. Google Sheets structure

The workflow writes to a `Discovery Calls` tab with these columns:

| Date | Lead Name | Phone | Summary | Pain Points | Tools/CRM | Agent Type | Interest Level | Suggested Automations | Next Steps | Objections | Transcript |

Create this tab manually or modify the workflow's Sheets nodes to match your existing schema.

### 7. Import and configure

1. In n8n: **Workflows → Import from file → `workflow.json`**
2. Update credential IDs in Drive and Sheets nodes
3. Update folder IDs in the Google Drive trigger and Move File nodes
4. Update the spreadsheet ID in all Sheets nodes
5. Activate the workflow

### 8. Test

Drop a short test recording into the `incoming/` folder with a filename containing a phone number from your Leads sheet. Wait ~60 seconds for the Drive trigger to fire and check the execution log.

---

## Claude analysis fields

The analysis prompt extracts:

- **Summary** — 2-3 sentence overview of the call
- **Pain points** — what the caller is struggling with
- **Tools/CRM** — software they currently use
- **Interest level** — Hot / Warm / Cold with reasoning
- **Suggested automations** — specific to the caller's situation
- **Next steps** — concrete actions to take
- **Objections** — concerns raised during the call

Modify the analysis prompt in the Claude node to match your industry or use case.

---

## Customising the analysis prompt

The Claude prompt in the workflow is written for real estate sales calls. Edit it to match your niche — the structured JSON output format works for any industry.

---

## License

MIT
