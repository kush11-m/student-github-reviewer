# Student GitHub Reviewer

An AI-powered GitHub portfolio reviewer for students.

This project analyzes a GitHub username, fetches recent repository metadata from the GitHub API, and generates mentor-style feedback using a Groq-hosted LLM through LangGraph.

## Features

- FastAPI backend with a simple review endpoint
- LangGraph workflow with two nodes:
  - GitHub data extraction
  - AI mentor feedback generation
- Streamlit UI for entering a GitHub username and viewing results
- Optional GitHub token support for better API rate limits

## Project Structure

```text
student-github-reviewer/
├── main.py               # FastAPI app and /review endpoint
├── requirements.txt      # Python dependencies
├── agent/
│   ├── graph.py          # LangGraph flow definition
│   ├── nodes.py          # Data extractor + mentor reviewer nodes
│   └── state.py          # Shared state schema
└── ui/
    └── app.py            # Streamlit frontend
```

## How It Works

1. The UI (or API client) sends a GitHub username to `POST /review`.
2. LangGraph starts with `data_extractor`:
   - Calls GitHub user and repo APIs
   - Extracts recent repositories, primary languages, and public repo count
3. LangGraph moves to `code_mentor`:
   - Sends extracted data to Groq LLM (`llama-3.1-8b-instant`)
   - Returns concise, actionable mentor feedback
4. Backend responds with extracted data and feedback.

## Requirements

- Python 3.10+
- A Groq API key
- Optional: GitHub personal access token (for higher rate limits)

## Installation

```bash
# from project root
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## Environment Variables

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key_here
GITHUB_TOKEN=your_github_token_here
```

Notes:
- `GROQ_API_KEY` is required for AI feedback generation.
- `GITHUB_TOKEN` is optional but recommended.

## Run Locally

### 1) Start backend (FastAPI)

```bash
uvicorn main:app --reload
```

Backend will run at `http://127.0.0.1:8000`.

### 2) Start frontend (Streamlit)

In a new terminal (same virtual environment):

```bash
streamlit run ui/app.py
```

## API Endpoints

### `GET /`
Health check endpoint.

Response:

```json
{
  "message": "GitHub Reviewer backend is running perfectly!"
}
```

### `POST /review?username=<github_username>`
Runs the full analysis pipeline.

Example:

```bash
curl -X POST "http://127.0.0.1:8000/review?username=torvalds"
```

Example response shape:

```json
{
  "username": "torvalds",
  "extracted_data": {
    "recent_repos": ["repo-a", "repo-b"],
    "primary_languages": ["C", "Python"],
    "public_repos_count": 8
  },
  "mentor_feedback": "Strong systems background..."
}
```

## Important Note About Current UI Backend URL

The Streamlit app currently points to a deployed backend URL:

- `https://github-reviewer-api-wrtv.onrender.com/review`

If you want to test fully local development, update `ui/app.py` to call:

- `http://127.0.0.1:8000/review`

## Troubleshooting

- `Could not connect to the backend` in Streamlit:
  - Ensure FastAPI is running
  - Ensure UI is calling the correct backend URL
- GitHub API errors:
  - Check username exists
  - Add `GITHUB_TOKEN` to avoid low unauthenticated rate limits
- Groq/LLM errors:
  - Verify `GROQ_API_KEY` is set correctly in `.env`

## Next Improvements

- Add unit tests for agent nodes and API routes
- Add request/response models for stronger FastAPI validation
- Add loading and error-state improvements in Streamlit
- Allow selecting model and number of repos from UI
