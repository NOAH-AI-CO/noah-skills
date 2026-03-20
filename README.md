## Noah AI Skills

A collection of domain-specific skills for querying Noah medical data APIs from medical and biotech AI agents or other automation. Each skill lives in its own folder under `skills/` with a `SKILL.md` description and a small CLI wrapper.

### Repository structure

- `AGENTS.md` — overview of all available skills and how agents should select and call them.
- `skills/clinical_trial` — **clinical-trial-search** skill for querying clinical trial databases.
- `skills/drug_pipeline` — **drug-search** skill for querying drug pipeline / development data.
- `skills/medical_conference` — **medical-conference-search** skill for querying conferences and presentations.

Querying skill folder typically contains:

- `SKILL.md` — formal skill specification (when to trigger, parameters, examples, dependencies).
- `scripts/search.py` — CLI entrypoint that takes JSON parameters and calls the Noah backend API.

### Requirements

- Python 3.8+
- `requests` (`pip install requests`)
- Environment variable `NOAH_API_TOKEN` set to a valid API token from `noah.bio`

#### Setting `NOAH_API_TOKEN`

#### How to get your API token
- Sign up or log in at [noah.bio](https://www.noah.bio).
- In the bottom-left corner of the page, click **API Key**.
- Create a new API key and copy the generated value into `NOAH_API_TOKEN`.
- If you are using **Openclaw**, you can simply ask it to set the variable for you, for example: *“Hi Openclaw, please set the environment variable NOAH_API_TOKEN=\\"your_token_here\\".”*

macOS / Linux (bash/zsh):
```bash
export NOAH_API_TOKEN="your_token_here"
```

### Running a skill from the CLI

All skills use a similar pattern: construct a JSON object with the query parameters described in that skill’s `SKILL.md`, then call the corresponding `scripts/search.py`.

Example (clinical trial search with inline JSON):

```bash
python skills/clinical_trial/scripts/search.py --params '{
  "indication": ["lung cancer"],
  "phase": ["Phase 3"],
  "page_num": 0,
  "page_size": 5
}'
```

Or with a params file:

```bash
echo '{
  "indication": ["lung cancer"],
  "phase": ["Phase 3"],
  "page_num": 0,
  "page_size": 5
}' > /tmp/query.json

python skills/clinical_trial/scripts/search.py --params-file /tmp/query.json
```

See each `skills/<name>/SKILL.md` for the full schema, examples, and any additional flags (e.g. conference-specific parameters).