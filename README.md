# Research Agent

An automated research system that implements the [Zettelkasten](https://en.wikipedia.org/wiki/Zettelkasten) methodology.

Given a topic, it searches curated web sources, synthesizes the information using an LLM, discovers related notes in your Obsidian vault, and writes a permanent note with proper connections — then commits everything to Git.

Supports both a **CLI** and a **Telegram bot** interface.

## Features

- **Automated research** — searches a curated list of high-quality domains (Wikipedia, Investopedia, Psychology Today, etc.)
- **LLM-generated notes** — synthesizes multiple sources into a 400–600 word permanent note in Traditional Chinese with frontmatter
- **Knowledge linking** — finds and links related notes already in your vault
- **Smart organization** — categorizes notes by domain (health, finance, psychology, technology, etc.) under `30-Permanent/{domain}/`
- **MOC updates** — automatically updates the Map of Contents for each domain
- **Git sync** — commits and pushes the vault after every note is written
- **Dual interfaces** — CLI for scripting, Telegram bot for mobile/remote access
- **Swappable LLMs** — switch between Claude and Gemini via a single environment variable

## Architecture

```
main.py
  └─ ResearchAgent.run(topic)
      ├─ TavilySearchHandler.search()      # web search + full-text enrichment
      ├─ ResultHandler.process()           # LLM: classify domain, find related notes
      ├─ NoteBuilder.build()               # LLM: generate Markdown note
      ├─ VaultWriter.write()               # write file, update MOC
      └─ GitSyncer.sync()                  # git add / commit / push
```

## Requirements

- Python 3.12+
- [uv](https://github.com/astral-sh/uv)
- An [Obsidian](https://obsidian.md/) vault (can be empty, should be a Git repo for sync)
- API keys: [Anthropic](https://console.anthropic.com/) or [Google Gemini](https://aistudio.google.com/), and [Tavily](https://tavily.com/)
- (Optional) A [Telegram bot token](https://core.telegram.org/bots#botfather) for the bot interface

## Installation

```bash
git clone <repo-url>
cd research-agent

cp .env.example .env
# Edit .env with your API keys and vault path

uv sync
```

## Configuration

All settings are read from `.env`:

| Variable | Required | Default | Description |
|---|---|---|---|
| `ANTHROPIC_KEY` | if Claude | — | Anthropic API key |
| `GEMINI_KEY` | if Gemini | — | Google Gemini API key |
| `LLM_PROVIDER` | no | `claude` | `claude` or `gemini` |
| `LLM_MODEL` | no | `claude-sonnet-4-6` | Model name |
| `TAVILY_KEY` | yes | — | Tavily search API key |
| `VAULT_PATH` | yes | `~/obsidian-vault` | Path to your Obsidian vault |
| `TELEGRAM_TOKEN` | if bot mode | — | Telegram bot token |
| `ALLOWED_USERS` | no | `0` | Comma-separated Telegram user IDs |

## Usage

**CLI — research a single topic:**

```bash
python main.py --topic "複利效應"
```

Output: creates `30-Permanent/{domain}/{topic}.md` in your vault, updates the domain MOC, and pushes to Git.

**Telegram bot — long-running service:**

```bash
python main.py --bot
```

Send any topic as a message to get real-time progress updates and the final result.

## Project Structure

```
research-agent/
├── main.py                  # Entry point & composition root
├── config.py                # Environment-based configuration
├── agent/
│   └── research_agent.py    # Main orchestrator (Facade)
├── llm/
│   ├── base.py              # Abstract LLMClient
│   ├── claude.py            # Anthropic Claude implementation
│   └── gemini.py            # Google Gemini implementation
├── search/
│   ├── tavily_handler.py    # Tavily search + full-text enrichment
│   └── fetcher.py           # Web page content extractor
├── processing/
│   ├── result_handler.py    # Domain classification & related-note discovery
│   └── note_builder.py      # LLM-powered Markdown note generation
├── storage/
│   ├── vault_writer.py      # Obsidian vault file writer & MOC updater
│   └── git_syncer.py        # Git commit & push automation
└── interfaces/
    ├── cli.py               # Command-line interface
    └── telegram_bot.py      # Telegram bot interface
```
