# What the Consensus videos covered (plain summary)

This is a summary of ten transcripts supplied for this kit: a launch video, eight short tutorials, and one long webinar. All
are from Consensus (consensus.app), the company behind the search tool, so they are promotional as well as
instructional. The tutorials are about their own product.

## The big idea
General AI chatbots are good at planning and writing but weak at academic search: they search the open web,
miss papers, and sometimes invent sources. The fix: connect Claude (or ChatGPT) to Consensus through a
"connector" (technically an MCP server), so Claude does the planning and writing while Consensus does the
searching and returns real papers with links.

## Setup (Claude)
Plus (+) > Connectors > Manage connectors > Browse > Consensus > Connect > sign in > Allow access. Test
with a question like "What is the latest research on caffeine and dementia?" A working connection shows
"Found 20 papers, showing top N". N is 3 if not signed in, 10 on a free Consensus account, 20 on paid.
Setup for ChatGPT: App Store (or Plugins) > Consensus > Connect, then tag it in the chat.

## Level 1 versus level 2
A single search is better done in the Consensus website itself (filters, tables, visualizations). The
connector's value is "level 2": combining Claude's planning with several Consensus searches, driven by a
saved set of instructions called a **skill** (a plain text/markdown file: a recipe with steps and rules).

## The three example skills
1. **Recommended Reading List:** upload a course syllabus; Claude extracts topics and learning outcomes,
   searches Consensus per topic for recent papers (last year), and produces a document with a summary and a
   discussion question per paper.
2. **Literature Review Helper:** initial search, reads the top 20, picks a framework (PICO by default),
   proposes sub-areas, asks how deep to go (5, 10 or 20 searches), runs searches, and produces a "launchpad"
   document: priority reading order, timeline of milestones, sub-area guides with search terms and Boolean
   strings, key research groups, gaps, bibliography, and an audit log.
3. **Grant Finder (NIH only):** asks intake questions (seniority, preliminary data, institution), runs five
   Consensus searches to find gaps and draft positioning language, then queries NIH RePORTER for funded
   projects, finds institutes, study sections, NOSIs and funding announcements, and produces a report with
   strategy and program-officer advice.

## Customizing
Skills are just text. Open one under Customize > Skills and use **Edit with Claude** to change it (for
example: swap NIH for another funder, add a preprint disclaimer, add an approval step). The speakers said
the best results come from treating a template as a starting point and adapting it to your own process.

## Plan notes from the videos
- The connector works on any Claude plan. Skills may need a Claude paid plan (speaker was unsure; a
  free-plan workaround is to attach the skill file and ask Claude to follow it, with output in chat).
- Word-document output may not work on the free plan.
- Cowork (desktop app) is recommended for the longer workflows. It lets Claude keep a to-do list and run
  longer tasks, and supports scheduled tasks.
- Webinar-stated limits: not signed in 3 papers per search; free 10 papers and about 30 searches/month; paid
  20 papers and about 1,000 searches/month. These can change.

## Things to keep in mind
- Do not read the output as a finished literature review. All speakers said so.
- The grant skill is built for NIH. Non-NIH funders need adaptation and verification.
- Numbers like "220 million papers" are the company's marketing claims at the time of recording.
