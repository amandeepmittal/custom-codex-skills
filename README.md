# Custom Codex Skills

## Using these skills with codex-cli

- Start codex-cli from the repo root (e.g., `codex chat`); skills in `.codex/skills` are auto-discovered at startup if you have copied them into your `.codex/skills` directory or manually at your project's root.
- Invoke a skill by mentioning its name or describing a task that matches its description; the agent will follow its `SKILL.md` instructions.
- Inspect a skill directly with `cat .codex/skills/<skill-name>/SKILL.md`; follow any linked references or scripts instead of retyping steps.
- Keep context lean: open only the referenced files/sections a skill points to; reuse provided assets/templates.
- When skills have variants or references, load only what the `SKILL.md` calls for to stay fast and focused.

## Resources

- [Codex Skills documentation](https://developers.openai.com/codex/skills#where-to-save-skills)
- [Agent Skills documentation](https://agentskills.io/integrate-skills)
