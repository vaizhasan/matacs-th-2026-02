# AI Proposal Triage Design

**Name:** Vaiz Hasan  
**Assessment reference:** MATACS-TH-2026-02  
**Time spent:** 47 minutes

## Repository contents

- [Process-flow diagram](diagram/process-flow.png)
- [Design write-up](WRITEUP.md)
- [AI prompt](PROMPTS.md)
- [Expected proposal ranking](ranking.json)

## How I used AI tools

I used ChatGPT to help structure the design, draft an extraction prompt, evaluate scoring options, and identify edge cases such as missing timeline evidence and instructions embedded in proposal text. I identified and addressed the risk of allowing the model to produce the final weighted ranking. Instead, deterministic code should apply the administrator's versioned criteria and calculate scores, ensuring the results are auditable and repeatable. I reviewed the proposed scoring assumptions and sample outcomes rather than treating AI-generated suggestions as authoritative.

## Scoring assumptions

The brief does not prescribe formulas for the weighted criteria, so this design states explicit, configurable formulas in `WRITEUP.md`. The ranking is an illustrative result under those assumptions, not a claim that the brief uniquely determines one correct ranking. Proposal D is flagged and left unranked because its completion timeline is not stated; Proposal B is ineligible because the bidder's own Ontario electrical contractor licence is not confirmed.
