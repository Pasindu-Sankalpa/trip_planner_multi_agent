# Trip planner multi agent

A trip planner built with [crewAI](https://docs.crewai.com) using JSON-first configuration.
You give a destination and how many days you'll stay, and five agents work in sequence to
plan the trip:

1. **Destination Expert**: lists the attractions worth visiting (about 3 per day).
2. **Lodging Planner**: finds one cheap, well-rated place to stay near the attractions and
   notes whether breakfast is included.
3. **Route Planner**: groups the attractions into days and plans how to travel between them
   (walk, bus, metro, taxi, etc.), starting and ending each day at the hotel.
4. **Dining Planner**: suggests local dishes and plans each meal near the day's route
   (skipping breakfast if the hotel includes it).
5. **Itinerary Planner**: combines everything into a timed, day-by-day schedule.

Each agent builds on earlier agents' output:

| Agent | Uses output from |
|---|---|
| Lodging Planner | Destination Expert |
| Route Planner | Destination Expert, Lodging Planner |
| Dining Planner | Lodging Planner, Route Planner |
| Itinerary Planner | All four agents |

Each step's result is saved to `trip_output/` (`1_attractions.md` through
`5_full_itinerary.md`). The final file is the complete itinerary.

All agents use the model set by `MODEL` in `.env` (for example
`MODEL=gemini/gemini-3.8-flash`), along with that provider's credentials.

## Create Environment

CrewAI requires `Python >=3.10 and <3.14`. Here's how to check your version:

```bash
python3 --version
```

Install uv:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Install CrewAI:

```bash
uv tool install crewai
```

To verify that `crewai` is installed, run:

```bash
uv tool list
```

You should see something like:

```
crewai v0.102.0
- crewai
```

### Connect to Google Cloud (Vertex AI)

The agents call Gemini through Vertex AI (`GOOGLE_GENAI_USE_VERTEXAI=true` in `.env`), so
this machine must be signed in to Google Cloud. Without this, `crewai run` fails with
"Your default credentials were not found".

Install the Google Cloud CLI:
Download the Windows installer from [cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install) and run it. Afterwards, open a new terminal window so that the `gcloud` command is recognized.

Sign in and connect to the project:
Run these commands one at a time, replacing `YOUR_PROJECT_ID`. Each `login` command opens a browser window; sign in with your organization account.

```bash
gcloud auth login
gcloud config set project YOUR_PROJECT_ID
gcloud auth application-default login
```

Use the same project ID for `GOOGLE_CLOUD_PROJECT` in `.env`.

## Running

```bash
crewai run
```

`crewai run` asks for the `destination` (for example `Kyoto, Japan`) and the number of
`days`.

## Project Structure

- `agents/` - Agent definitions (JSONC)
- `crew.jsonc` - Crew definition with tasks and configuration
- `tools/` - Custom tools (Python)
- `knowledge/` - Knowledge files for agents
- `trip_output/` - Generated trip plans (created on first run, not committed)

> **Note:** `custom:<name>` tool references execute `tools/<name>.py` as local
> Python code when the crew loads. Only run crew projects from sources you
> trust.
