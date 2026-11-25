# Weekly Plan Automation (`Wochenplan` -> Todoist)

An `n8n` workflow that listens for new weekly plan entries in Google Sheets, parses tasks and target weekdays in the `Europe/Berlin` timezone, structures and categorizes them using an LLM via the Groq API, and creates corresponding tasks in a Todoist project.

## How It Works

1. **Trigger (`Google Sheets Trigger`):** Polls a Google Sheet every minute for newly added rows in the weekly plan responses sheet[cite: 1].
2. **Input Parsing & Date Calculation (`Prepare Days`):**
   - Locates the `Wochenplan` column and splits non-empty lines[cite: 1].
   - Expects each line in the `Task - Day` format (separated by the last ` - `)[cite: 1].
   - Resolves each weekday in `Europe/Berlin` time to the next matching date (today or upcoming within 7 days)[cite: 1].
3. **LLM Structuring (`Groq - Parse Weekly Plan`):**
   - Sends the parsed tasks and submission date to `https://api.groq.com/openai/v1/chat/completions` (`openai/gpt-oss-20b`, JSON object mode)[cite: 1].
   - Generates a concise action title, a 1–3 sentence description, preserves `due_date`, and assigns one of the allowed categories: `Russisch lernen`, `TUM`, `Arbeit`, `Training`, or `Sonstiges`[cite: 1].
4. **JSON Validation (`Parse AI JSON`):** Validates the AI response structure, non-empty titles, `YYYY-MM-DD` dates, and allowed categories[cite: 1].
5. **Todoist Lookup & Mapping (`Get Projects`, `Get Sections`, `Merge`, `Validate and Map Tasks`):**
   - Fetches Todoist projects and sections for the target project (`Weekly Plan`) and merges them with the parsed tasks[cite: 1].
   - Validates calendar dates, normalizes names/categories to match Todoist sections (with a fallback to `Needs Review` if an unexpected category slips through), and prepares the task payload[cite: 1].
6. **Task Creation (`HTTP Request`):** Sends `POST` requests to `/api/v1/tasks` on the Todoist API to create each task with its content, description, due date, project ID, and section ID[cite: 1].

## Expected Input Format

In the `Wochenplan` column, enter one task per line using `<task description> - <day>`:

```text
Review lecture notes - Montag
Finish backend endpoint - Mittwoch
Gym session - Freitag
```

## Tech Stack & Integrations

- **Workflow Engine:** n8n (self-hosted, e.g., via Docker)[cite: 1]
- **Input Source:** Google Sheets Trigger (`n8n-nodes-base.googleSheetsTrigger`)[cite: 1]
- **LLM API:** Groq Chat Completions API (`openai/gpt-oss-20b`)[cite: 1]
- **Task Manager:** Todoist REST API (`/api/v1/projects`, `/api/v1/sections`, `/api/v1/tasks`)[cite: 1]
- **Scripting:** JavaScript (`n8n-nodes-base.code`)[cite: 1]

## Setup & Credentials

1. Import `Wochensplan.json` into your `n8n` instance[cite: 1].
2. Configure the required credentials:
   - **Google Sheets Trigger OAuth2 API** connected to your weekly plan spreadsheet[cite: 1].
   - **HTTP Bearer Auth (`Groq Bearer`)** with your Groq API key[cite: 1].
   - **HTTP Bearer Auth (`Todoist Bearer`)** with your Todoist API token[cite: 1].
3. Ensure your Todoist account has a `Weekly Plan` project with matching sections (`Russisch lernen`, `TUM`, `Arbeit`, `Training`, `Sonstiges`, and optionally `Needs Review`), and update the target project/section IDs as needed[cite: 1].
4. Activate the workflow in `n8n`[cite: 1].

## Repository Contents

- `Wochensplan.json` – Exported `n8n` workflow definition[cite: 1].
