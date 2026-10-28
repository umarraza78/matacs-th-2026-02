# Design Write-up: AI-Assisted Proposal Triage

## Assumptions

- Proposals are PDFs in English. Some may be scans.
- The admin has set up the category's criteria before any upload.
- Price means the total installed price in CAD.
- "Last 5 years" counts back from the upload date.
- The bidder must hold the licence itself. A promise to find a licensed subcontractor later does not pass.

## 1. AI vs rules

I use AI for two jobs only:

- **Reading.** The AI turns messy proposal text into a fixed JSON format. Every value has a short quote and a page number, or it is null.
- **Wording.** The AI turns the final scores into a one-line reason a reviewer can read.

Plain code does everything else: checking the JSON format, confirming each quote really appears in the PDF text, mandatory pass/fail, budget and deadline checks, scoring with the admin's weights, ranking and tie-breaks.

I drew the line here because AI is good at reading free text and rules are not. For judging and arithmetic it is the opposite. Code gives the same answer every time and I can test it.

## 2. Consistency and explainability

- Extraction runs once per file at temperature 0. The result is stored. Scoring reads the stored facts, never the AI.
- Same facts plus same criteria version always give the same score. Ranking twice gives the same order.
- Ties are broken by fixed rules: lower price first, then more relevant installs. If still tied, a person decides.
- The reviewer sees points per criterion, the fact used, and its quote and page. One click opens that PDF page.

**My scoring rule** (used for ranking.json):

- For each weighted criterion, the best value among proposals that pass the mandatory checks gets full points. Flagged proposals still count when finding the best value.
- Higher is better (experience, warranty): points = weight x value / best value.
- Lower is better (price, timeline): points = weight x best value / value.
- Warranty value = panel years + workmanship years.
- Team and approach: four items (named lead, crew size, safety plan, monitoring plan), 3.75 points each.

| Criterion | Weight | Best value | A | E |
|---|---|---|---|---|
| Experience | 30 | 14 installs | 14: 30.00 | 8: 17.14 |
| Price | 25 | $231,000 (D) | $238,000: 24.26 | $242,500: 23.81 |
| Timeline | 15 | 12 weeks | 12: 15.00 | 13: 13.85 |
| Warranty | 15 | 37 years | 35: 14.19 | 37: 15.00 |
| Team | 15 | 4 of 4 | 4: 15.00 | 4: 15.00 |
| **Total** | 100 | | **98.5** | **84.8** |

B fails the licence check, so it is ineligible. C and D are flagged and not scored until a person reviews them.

## 3. Human in the loop

- A reviewer must look at every flagged and ineligible proposal before the list is shared.
- The reviewer can confirm or correct any fact and change any status. Every override needs a written reason.
- Corrected facts go through the same scoring code. Nobody types a score by hand.
- The final award is a separate action by a person.
- The audit log records: file hash, prompt version, model name and settings (temperature 0), the raw extracted JSON, criteria version, every score and flag, every override with its reason, and who did what and when. The log is append-only.

## 4. Changing criteria

- Criteria are versioned. Every edit makes a new version. Old versions are never changed.
- Every score is saved with the criteria version it used.
- Old results stay tied to the old version. A reviewer clicks "re-score" to apply the new one. This only re-runs scoring code, with no AI cost.
- A new criterion that needs a new fact triggers an extraction pass for that one field only.
- The screen shows old and new results side by side, with rank changes highlighted.

## 5. Risks and safeguards

| Risk | Safeguard |
|---|---|
| Wrong extraction | Every value has a quote and page. Low confidence is flagged. |
| Made-up facts | Code checks that each quote exists in the source text. If not, the value becomes null and is flagged. |
| Bias | The AI never scores. Scoring uses only fixed fields, never the bidder's name or writing style. Admins review criteria for rules only big firms can meet. |
| Prompt injection | Proposal text is labelled as untrusted data. The AI reports any instructions in an "injection_suspected" field. Code also scans for phrases like "ignore previous instructions". Flagged proposals are never ranked automatically. The AI cannot change scores. |
| Cost | Extract once per file, cached by file hash. Re-scoring is free. Use a smaller model for reason text. |
| Privacy | Proposals hold pricing and staff names. I would use a provider that does not train on our data, keep keys out of code (only a .env.example is shared), limit access by role, and set a retention period. |

## How the design adapts to changes

- **New weights:** the admin saves a new version. Scoring code re-runs. No AI needed.
- **New mandatory criterion:** if the fact is already extracted, only code runs. If not, one extraction pass for that field.
- **Lower budget ceiling:** the ceiling is a stored value. The budget check re-runs and bids above it become ineligible.
- **New category:** the admin creates a new criteria set. The extraction prompt is built from that set. No code change.
- **Scanned PDFs:** code detects missing text, runs OCR, and stores the OCR text. The quote check runs against that text. Low OCR confidence is flagged.
