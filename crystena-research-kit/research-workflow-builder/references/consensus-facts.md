# Consensus facts used by this kit

Source: Consensus's own tutorial videos and launch webinar (transcripts supplied with this kit). These
describe the connector shortly after launch. **Limits and pricing can change. Verify at consensus.app before
promising anything.** Where a figure came from a spoken webinar, treat it as approximate.

## What it is
- The Consensus "connector" (technically an MCP server) lets Claude (or ChatGPT) run searches in Consensus
  and receive back ranked papers with title, authors, year, journal, citation count, abstract/metadata, and a
  link to the paper page in Consensus.
- Consensus describes its index as over 220 million peer-reviewed papers (older videos), drawn from sources
  such as Semantic Scholar, OpenAlex and PubMed, plus preprint servers. The connector's own listing in
  one Claude session said 400M+. The figure differs by source and date; do not quote a number to the user
  as fact.
- Each search returns a ranked shortlist (top 3, 10 or 20 depending on account), not an exhaustive list.

## Result and usage limits (from the webinar; may have changed)
| Consensus account state | Papers returned per search | Searches per month |
|---|---|---|
| Not signed in to Consensus | 3 | shared pool, rate limited |
| Free Consensus account | 10 (the result line says "Found 20 papers, showing top 10") | about 30 |
| Paid individual or institution/library account | 20 | about 1,000 |

- This is the Consensus account tier, not the Claude plan. The connector works on any Claude plan.
- A Standard literature review in this kit uses about 10 searches; a Deep dive about 20. On a free Consensus
  account (about 30 per month), that is a third to two thirds of the month. Warn the user.
- The webinar said Consensus planned to let users add more calls later. Unknown whether that has happened.

## How to confirm the connection works
Run any search and look for a line like "Found 20 papers, showing top 20".
- showing 3 means not signed in
- showing 10 means free account
- showing 20 means paid or institution account

## Rate limit
The templates require one Consensus query per second and sequential searches (no parallel calls). The
videos describe Claude "running searches in parallel", but the supplied skill files say to run them
one at a time. This kit follows the skill files, because parallel calls may be throttled.

## Honest caveats from the sources themselves
- Consensus says the tools "are not meant to replace research", they provide a starting point.
- The grant skill as shipped is NIH-only (it uses NIH RePORTER and NIH Guide notices, and grants.gov).
- Doc creation (Word files) may not work on the free Claude plan; output can be written in chat instead.
- Skills "may need a Claude Pro account" (speaker said "I believe"), with a workaround on free: attach the
  skill as a text file and ask Claude to follow it.
- "Cowork" (a mode with a to-do list and long-running tasks) is in the Claude desktop app and was described
  as better for the longer workflows; it is not required.
- Whether Consensus covers a given niche field well is unknown. Test with a small search first.
