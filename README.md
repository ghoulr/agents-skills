# Skills

AI agent skills collection brewed by hand and love. Each skill is a self-contained module with a `SKILL.md` entry point.

## Skills

| Skill | Description | Source |
|-------|-------------|--------|
| **skill-creator** | Create, evaluate, and optimize agent skills with iterative benchmarking | [anthropics/skills](https://github.com/anthropics/skills) |
| **paper-reading** | Read, summarize, and analyze academic papers (arxiv URLs, local PDFs) | — |
| **web-design-engineer** | Build visual web artifacts — pages, dashboards, prototypes, slide decks | [ConardLi/web-design-skill](https://github.com/ConardLi/web-design-skill) |
| **chrome-cdp** | Interact with local Chrome browser via DevTools Protocol (with iframe support) | [pasky/chrome-cdp-skill](https://github.com/pasky/chrome-cdp-skill) |

## Dependencies

- **paper-reading**: requires [uv](https://docs.astral.sh/uv/) and an OCR model runtime. Install dependencies with `uv sync` in the skill directory.
- **skill-creator**: scripts use [pi](https://github.com/badlogic/pi-mono) CLI (`pi -p`) for eval runs. Requires Python 3.10+, or ask your agent change the cli for whatever you use.

## License

MIT
