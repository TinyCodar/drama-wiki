# LLM Wiki Stack

This directory is both:
- Hermes `llm-wiki` knowledge base
- Obsidian vault
- local Git repository root

Recommended workflow:
1. Open `/Users/tinycoder/wiki` in Obsidian
2. Use Graph View to inspect page relationships
3. Use Hermes with `llm-wiki` skill to ingest/update knowledge
4. Commit local changes with Git
5. Push to GitHub after remote auth/repo is configured

Key files:
- `SCHEMA.md`: wiki schema and taxonomy
- `index.md`: navigational catalog
- `log.md`: action log
- `raw/`: immutable sources
- `entities/ concepts/ comparisons/ queries/`: agent-maintained wiki pages
