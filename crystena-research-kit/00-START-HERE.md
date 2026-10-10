# Crystena's Research Workflow Kit - Start Here

This kit sets up Claude as a research assistant for your PhD papers. It is built from Consensus's own
tutorial videos and webinar. After setup, Claude can search real peer-reviewed papers (through the
Consensus connector) and build you things like a literature-review starting guide or a list of funding
opportunities, with every citation linked to an actual paper.

**What it will not do:** write your literature review for you. It gives you a fast, organized starting map.
You still read the papers and decide what to cite.

## What you need
- A Claude account (free works for the basics; a paid plan makes the longer workflows smoother).
- A free Consensus account (consensus.app). Sign-in matters: without it you only get 3 papers per search.
- About 30 minutes.

## The 3-step setup

### Step 1 - Connect Consensus to Claude (5 minutes)
1. Open Claude (desktop app is best).
2. Click **+** in the message box, then **Connectors**, then **Manage connectors**, then **Browse**.
3. Search **Consensus** and click **Connect**.
4. A Consensus page opens. Sign in or create a free account, then click **Allow access**.
5. Go back to Claude. If you are not sure it worked, skip ahead: the next step tests it for you.

### Step 2 - Give Claude this kit
Pick the first option that works for you.

**Option A (best): install the builder as a skill**
1. Find the file `research-workflow-builder.zip` inside this kit.
2. In Claude, open **Customize** (left sidebar), then **Skills**, then **Add / Upload**, and choose that zip.
3. Start a new chat and type: **"Start my research workflow setup."**

**Option B: attach the file**
1. Start a new chat (use **Cowork** in the desktop app if you have it).
2. Attach this whole kit zip, or just `research-workflow-builder/SKILL.md`.
3. Type: **"Follow the instructions in the attached file to set up my research workflows."**

**Option C (works on any plan): paste**
1. Open `01-PASTE-THIS-PROMPT.md`, copy everything in it, and paste it into a new Claude chat.
2. Press Enter.

If a menu name does not match what you see, the app may have updated. Tell Claude what you see and it will
adjust.

### Step 3 - Answer the questions
Claude will check Consensus works, then ask you about your field, where you are in your PhD, what takes
the most time, and how you like your output. It will propose 2 to 4 workflows, build them as skills,
and test one on your real topic. You can stop and resume any time.

## Good to know (read this)
- **Search limits.** On a free Consensus account you get about 30 searches a month (the webinar's figure,
  which may change), and a standard literature review uses about 10. Pick your moments, or ask Claude for
  the "quick scan" depth (5 searches).
- **Free Claude plan.** You may not be able to get Word documents back. Tell Claude "write the results in
  chat" and it will.
- **Never cite a paper you have not opened.** Every link goes to the real paper page; open it and check
  the claim yourself.
- **Check your institution's AI policy** before using AI-assisted output in anything you submit.

## What is in this kit
| File | What it is |
|---|---|
| `00-START-HERE.md` | This page |
| `research-workflow-builder.zip` | The installable skill that interviews you and builds your workflows |
| `research-workflow-builder/` | The same skill as plain folders (SKILL.md plus reference files) |
| `01-PASTE-THIS-PROMPT.md` | A single paste-in version for any plan |
| `reference/what-the-videos-covered.md` | Plain summary of the Consensus videos this was built from |
| `research-workflow-builder/references/original-chatgpt-templates/` | Consensus's three original workflow files (written for ChatGPT) that the builder adapts for Claude |

## If something goes wrong
- **"Showing 3 papers":** you are not signed in to Consensus. Reconnect the connector and sign in.
- **Claude cannot find Consensus:** in the **+** menu, make sure Consensus is switched on for this chat.
- **Skill upload not available:** use Option B or C.
- **A search fails:** Claude retries once, then tells you. It should never silently skip a step. If it does,
  tell it so.
- Consensus support: use the Support link inside the Consensus app.
