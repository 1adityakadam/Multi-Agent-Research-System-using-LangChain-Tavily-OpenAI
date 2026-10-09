# Multi-Agent Research System using LangChain, Tavily & OpenAI

Enter any topic and four AI agents take it from there. One searches the live web, one scrapes the most relevant source, one writes a structured report with sources, and one critiques it with a score out of 10. Built with LangChain, Tavily and GPT-4o-mini, with a Streamlit interface that shows each stage as it runs.

## How it works

```
Topic → Search Agent → Reader Agent → Writer Chain → Critic Chain → Report + Score
         (Tavily)      (scrape URL)    (LCEL)         (LCEL)
```

| Step | Component | What it does |
|---|---|---|
| 1 | **Search Agent** | A LangChain agent with the `web_search` tool. Queries Tavily and returns titles, URLs and snippets for the top 5 results. |
| 2 | **Reader Agent** | A LangChain agent with the `scrape_url` tool. Picks the most relevant URL from the search results and pulls its text for deeper reading. |
| 3 | **Writer Chain** | An LCEL chain (`prompt \| llm \| parser`) that combines the search results and scraped content into a report with Introduction, Key Findings, Conclusion and Sources. |
| 4 | **Critic Chain** | An LCEL chain that reviews the report and returns a score out of 10, strengths, areas to improve, and a one-line verdict. |

## Tools

| Tool | Description |
|---|---|
| `web_search(query)` | Tavily search, returns up to 5 results with title, URL and a 300-character snippet |
| `scrape_url(url)` | Fetches the page with Requests, removes scripts, styles, navigation and footers with BeautifulSoup, and returns the first 3,000 characters of clean text |

`scrape_url` has an 8-second timeout and returns a readable error message if a page can't be fetched, so one bad URL doesn't stop the pipeline.

## Features

- Two tool-calling agents (search and read) and two prompt chains (write and critique)
- Live web research instead of relying only on model knowledge
- Structured report format with a sources list
- LLM-as-judge critique with a fixed output format
- Streamlit app with a status card per stage (waiting, running, done)
- Command-line version for quick runs

## Tech stack

Python · LangChain · OpenAI (`gpt-4o-mini`) · Tavily · BeautifulSoup · Requests · Streamlit · python-dotenv

## Setup

**1. Install dependencies**

```bash
pip install -r requirements.txt
pip install streamlit
```

This project uses `create_agent` from LangChain 1.x, so make sure `langchain>=1.0` is installed.

**2. Add your API keys** in a `.env` file in the project root:

```bash
OPENAI_API_KEY=your_openai_key
TAVILY_API_KEY=your_tavily_key
```

**3. Run it**

Web app:

```bash
streamlit run app.py
```

Open http://localhost:8501, enter a topic and click **Run Research Pipeline**.

Command line:

```bash
python pipeline.py
```

## Example topics

- `LLM agents 2025`
- `CRISPR gene editing`
- `Fusion energy progress`

## Example critic output

```
Score: 8/10

Strengths:
- Clear structure with well-explained key findings
- Sources are listed

Areas to Improve:
- Add more quantitative detail
- Cover more than one source

One line verdict:
A solid, readable report that would benefit from broader sourcing.
```

## Project structure

| File | Description |
|---|---|
| `app.py` | Streamlit interface with the pipeline status cards |
| `pipeline.py` | The same four steps as a command-line script |
| `agents.py` | Search and reader agents, plus the writer and critic chains |
| `tools.py` | The `web_search` and `scrape_url` tools |
| `requirements.txt` | Python dependencies |

## Limitations and next steps

- The reader scrapes a single URL and keeps the first 3,000 characters, so the report is only as deep as that source
- The critic scores the report but doesn't send it back for revision
- Next: scrape the top 3 sources, and loop the critic's feedback into a rewrite step
