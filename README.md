<div align="center">

![Claude Code](https://img.shields.io/badge/Claude%20Code-D97757?style=for-the-badge&logo=claude&logoColor=white)
![Gemini](https://img.shields.io/badge/Gemini-4285F4?style=for-the-badge&logo=google&logoColor=white)
![License MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)

# Don't Ask Me

**Cross-check Claude's answer with a second AI before you trust it.**

</div>

---

![](assets/memes/01-trap.png)

![](assets/memes/02-forest.png)

![](assets/memes/03-deploy.png)

![](assets/memes/04-advice.png)

---

## Install

```
/plugin marketplace add awrshift/skill-dont-ask-me
```

Then set your Gemini key (free tier at [aistudio.google.com](https://aistudio.google.com)):

```bash
echo 'GOOGLE_API_KEY=your_key_here' >> ~/.env
pip install google-genai
```

## Three modes — Claude picks one based on what you type

| Mode | Trigger phrase | What happens |
|---|---|---|
| **Second opinion** | *"sanity check"*, *"am I missing something"* | One reviewer (Gemini or isolated Claude). ~3¢ |
| **Full review** | *"this is important"*, *"run a full review"* | Both reviewers in parallel. ~7¢ |
| **Group discussion** | *"help me choose"*, *"brainstorm options"* | 3-round debate. ~25¢ |

## Full docs

[**SKILL.md**](SKILL.md) — how it works, when to invoke, anti-patterns, CLI reference.

## License

MIT. PRs welcome.
