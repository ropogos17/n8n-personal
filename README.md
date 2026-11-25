# Weekly Plan Automation (`Wochenplan` -> Todoist)

A self-hosted `n8n` workflow running behind a reverse proxy in Docker that listens for new weekly plan entries in Google Sheets, parses tasks and target weekdays in the `Europe/Berlin` timezone, structures and categorizes them using an LLM via the Groq API, and creates corresponding tasks in a Todoist project.

## Architecture & Self-Hosting

The workflow is designed to run on a self-hosted `n8n` instance deployed via Docker (`n8nio/n8n`) with persistent storage (`n8n_data` volume mounted to `/home/node/.n8n`) behind an HTTPS reverse proxy (`N8N_PROXY_HOPS=1`, `N8N_PROTOCOL=https`, `N8N_EDITOR_BASE_URL`, `WEBHOOK_URL`).

## How It Works

1. **Trigger (`Google Sheets Trigger`):** Polls a Google Sheet every minute for newly added rows in the weekly plan responses sheet.
2. **Input Parsing & Date Calculation (`Prepare Days`):**
   - Locates the `Wochenplan` column and splits non-empty lines.
   - Expects each line in the `Task - Day` format (separated by the last ` - `).
   - Resolves each weekday in `Europe/Berlin` time to the next matching date (today or upcoming within 7 days).
3. **LLM Structuring (`Groq - Parse Weekly Plan`):**
   - Sends the parsed tasks and submission date to `https://api.groq.com/openai/v1/chat/completions` (`openai/gpt-oss-20b`, JSON object mode).
   - Generates a concise action title, a 1–3 sentence description, preserves `due_date`, and assigns one of the allowed categories: `Russisch lernen`, `TUM`, `Arbeit`, `Training`, or `Sonstiges`.
4. **JSON Validation (`Parse AI JSON`):** Validates the AI response structure, non-empty titles, `YYYY-MM-DD` dates, and allowed categories.
5. **Todoist Lookup & Mapping (`Get Projects`, `Get Sections`, `Merge`, `Validate and Map Tasks`):**
   - Fetches Todoist projects and sections for the target project (`Weekly Plan`) and merges them with the parsed tasks.
   - Validates calendar dates, normalizes names/categories to match Todoist sections (with a fallback to `Needs Review` if an unexpected category slips through), and prepares the task payload.
6. **Task Creation (`HTTP Request`):** Sends `POST` requests to `/api/v1/tasks` on the Todoist API to create each task with its content, description, due date, project ID, and section ID.

## Expected Input Format

In the `Wochenplan` column, enter one task per line using `<task description> - <day>`:

```text
Review lecture notes - Montag
Finish backend endpoint - Mittwoch
Gym session - Freitag
```

## Tech Stack & Integrations

- **Workflow Engine:** Self-hosted `n8n` in Docker behind a reverse proxy
- **Input Source:** Google Sheets Trigger (`n8n-nodes-base.googleSheetsTrigger`)
- **LLM API:** Groq Chat Completions API (`openai/gpt-oss-20b`)
- **Task Manager:** Todoist REST API (`/api/v1/projects`, `/api/v1/sections`, `/api/v1/tasks`)
- **Scripting:** JavaScript (`n8n-nodes-base.code`)

## Setup & Credentials

1. Copy `.env.example` to `.env`, fill in your domain/environment settings, and start the stack with `docker compose up -d`.
2. Import `Wochensplan.json` into your `n8n` instance.
3. Configure the required credentials:
   - **Google Sheets Trigger OAuth2 API** connected to your weekly plan spreadsheet.
   - **HTTP Bearer Auth (`Groq Bearer`)** with your Groq API key.
   - **HTTP Bearer Auth (`Todoist Bearer`)** with your Todoist API token.
4. Ensure your Todoist account has a `Weekly Plan` project with matching sections (`Russisch lernen`, `TUM`, `Arbeit`, `Training`, `Sonstiges`, and optionally `Needs Review`), and update the target project/section IDs as needed.
5. Activate the workflow in `n8n`.

## Repository Contents

- `Wochensplan.json` – Exported `n8n` workflow definition
- `docker-compose.yml` – Example Docker Compose configuration for self-hosting `n8n`
