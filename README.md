# MATACS-TH-2026-02: AI-Assisted Proposal Triage (Design)

**Name:** Muhammad Umar Raza
**Reference:** MATACS-TH-2026-02
**Role:** Associate Software Developer

Time actually spent: [FILL IN HONESTLY]

## What this is

This is a design-only submission. I did not build an app. I describe how I would build a tool that reads contractor proposals, checks them against criteria set by an admin, and gives a ranked list with a reason for each position. A person always makes the final award decision.

My core idea: AI only reads the proposals and pulls out facts with quotes. Plain code does all checks, scoring and ranking, so results are repeatable and easy to explain.

## How I used AI tools

I used an AI assistant to help draft the documents, check my score maths and generate the flow diagram. I reviewed and edited everything, and I made the design decisions myself. [FILL IN: one real place where I disagreed with or corrected the AI, and why]

## Files

- [README.md](README.md): this file.
- [WRITEUP.md](WRITEUP.md): my design answers, scoring rule, risks and assumptions.
- [PROMPTS.md](PROMPTS.md): the extraction prompt and the reason-text prompt.
- [ranking.json](ranking.json): my ranked result for proposals A to E.
- [diagram/flow.png](diagram/flow.png): process flow from upload to final decision.
- [.gitignore](.gitignore): files kept out of the repo.

## Note on configuration

No API keys are included. If this were built, settings would go in a local `.env` file, with only a `.env.example` shared.
