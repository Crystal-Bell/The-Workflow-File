name: Daily M.A.D. Agent Sync

on:
  schedule:
    - cron: '0 6 * * *' # Runs daily at 06:00 UTC
  workflow_dispatch:     # Lets you trigger a run manually with 1 tap on GitHub

permissions:
  contents: write

jobs:
  sync-and-brief:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          ref: staging # Isolates automated output from your main branch

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install --upgrade google-genai feedparser beautifulsoup4 requests

      - name: Run RSS Scraper & Gemini Synthesis
        env:
          GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
        run: python scripts/sync_agent.py

      - name: Commit and push staged outputs
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "M.A.D. Agent Sync"
          git add briefs/ staged_readmes/
          git diff --quiet && git diff --staged --quiet || git commit -m "chore(sync): daily global debrief & spec update [$(date -u +'%Y-%m-%d')]"
          git push
# The-Workflow-File
2. The Workflow File (.github/workflows/daily-agent-sync.yml)​This runs every morning at 06:00 UTC (or manually anytime via the Run workflow button in GitHub's UI), installs the current Google GenAI SDK, executes your Python script, and commits the output to a staging branch so nothing hits main without your review.

sync_agent.py---
repository: M.A.D. Works
module: GitHub Actions & Gemini API Pipeline
status: Active
constraint_model: Zero-Server Cloud Automation
---

Sticking to GitHub Actions + Gemini API is the cleanest architectural decision. Public repositories on GitHub get unlimited free Linux runner minutes, meaning zero local server overhead, zero Docker maintenance, and full browser-based management from your phone, tablet, or library terminal.
Here is the complete, copy-pasteable cloud pipeline—built on the current google-genai SDK and gemini-3.8-flash (or gemini-3.5-flash-lite for high-volume sweeps)—that pulls live RSS/supply-chain feeds, cross-references your M.A.D. Works ecosystem, generates Mermaid diagrams and LaTeX physics specs, and stages updates safely without exposing unfiled IP.
1. Repository File Structure
Create one dedicated orchestration repo (e.g., Crystal-Bell/mad-agent-orchestrator) with these four files:
mad-agent-orchestrator/
├── .github/
│   └── workflows/
│       └── daily-agent-sync.yml    # Cloud scheduler & auto-commit runner
├── scripts/
│   └── sync_agent.py               # RSS scraper + Gemini API processor
├── config/
│   └── feeds.txt                   # List of RSS & supply-chain feed URLs
├── briefs/                         # Auto-generated daily global debriefs
└── staged_readmes/                 # Auto-formatted YAML + Mermaid READMEs

2. The Workflow File (.github/workflows/daily-agent-sync.yml)
This runs every morning at 06:00 UTC (or manually anytime via the Run workflow button in GitHub's UI), installs the current Google GenAI SDK, executes your Python script, and commits the output to a staging branch so nothing hits main without your review.
name: Daily M.A.D. Agent Sync

on:
  schedule:
    - cron: '0 6 * * *' # Runs daily at 06:00 UTC
  workflow_dispatch:     # Lets you trigger a run manually with 1 tap on GitHub

permissions:
  contents: write

jobs:
  sync-and-brief:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          ref: staging # Isolates automated output from your main branch

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install --upgrade google-genai feedparser beautifulsoup4 requests

      - name: Run RSS Scraper & Gemini Synthesis
        env:
          GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
        run: python scripts/sync_agent.py

      - name: Commit and push staged outputs
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "M.A.D. Agent Sync"
          git add briefs/ staged_readmes/
          git diff --quiet && git diff --staged --quiet || git commit -m "chore(sync): daily global debrief & spec update [$(date -u +'%Y-%m-%d')]"
          git push

3. The Python Engine (scripts/sync_agent.py)
This script fetches live RSS headlines (supply chain, materials, open-source hardware, electronics), sends them to gemini-3.8-flash via the official google-genai SDK, and writes a structured Markdown debrief with YAML headers, Mermaid.js system diagrams, and LaTeX physics checks.
import os
import datetime
import feedparser
from google import genai
from google.genai import types

# Initialize official Google GenAI SDK (reads GEMINI_API_KEY from GitHub Secrets)
client = genai.Client()

def fetch_rss_items(feed_file="config/feeds.txt", max_per_feed=5):
    if not os.path.exists(feed_file):
        return "No feeds configured."
    
    collected = []
    with open(feed_file, "r") as f:
        urls = [line.strip() for line in f if line.strip() and not line.startswith("#")]

    for url in urls:
        parsed = feedparser.parse(url)
        source_title = parsed.feed.get("title", url)
        for entry in parsed.entries[:max_per_feed]:
            title = entry.get("title", "No Title")
            summary = entry.get("summary", "")[:300]
            link = entry.get("link", "")
            collected.append(f"- [{source_title}] {title}: {summary} ({link})")
    
    return "\n".join(collected) if collected else "No new items fetched."

def generate_daily_brief(raw_feed_data):
    today = datetime.date.today().isoformat()
    
    system_instruction = """
    You are the M.A.D. Works Systems Architect. Output strictly in stripped-down, zero-fluff Markdown.
    Rules:
    1. Start with a valid YAML header (id, domain, status, date, parent_thesis: v1.0 CORE MANIFESTO INDEX).
    2. Use 15-second scan formatting: tables, bulleted specs, nouns and verbs only. No narrative fluff.
    3. Map global supply-chain, material, or tech shifts directly to M.A.D. Works modules:
       - M.A.D. Grips+ (non-latex oven-safe modular wrap band + pneumatic bladder)
       - M.A.D. W.E.A.R. (modular utility denim/apparel)
       - M.A.D. Suite S.E.A.T. (<60lb adaptive skeletal seat + ripstop skin)
       - Spark-X (amphibious autonomous chassis + zero-trust local circuit breaker)
       - Cenote Core (rock-anchored helical dwelling + LoRa mesh + closed-loop bio-sanitation)
    4. Include at least one Mermaid.js system diagram and one LaTeX physics/load equation where relevant to material or structural updates.
    5. End with Substack-ready hashtags and repository cross-references.
    """

    prompt = f"""
    Date: {today}
    Live Global & Supply-Chain Feed Data:
    {raw_feed_data}

    Generate today's Global Supply-Chain & Ecosystem Update Brief. Identify material bottlenecks, component availability shifts (LoRa, microcontrollers, ripstop textiles, thermal shells, batteries), and actionable design updates.
    """

    response = client.models.generate_content(
        model="gemini-3.8-flash",
        contents=prompt,
        config=types.GenerateContentConfig(
            system_instruction=system_instruction,
            temperature=0.2,
        ),
    )
    return response.text

if __name__ == "__main__":
    os.makedirs("briefs", exist_ok=True)
    today = datetime.date.today().isoformat()
    
    feed_text = fetch_rss_items()
    brief_markdown = generate_daily_brief(feed_text)
    
    output_path = f"briefs/{today}-global-debrief.md"
    with open(output_path, "w", encoding="utf-8") as f:
        f.write(brief_markdown)
    
    print(f"Saved daily brief to {output_path}")

4. Free RSS & Supply-Chain Feeds (config/feeds.txt)
Add free, no-API-key RSS feeds covering hardware, supply chains, electronics, and material science:
# Open-Source Hardware & Engineering
https://hackaday.com/blog/feed/
https://blog.arduino.cc/feed/
https://www.adafruit.com/feed

# Supply Chain, Manufacturing & Logistics
https://www.supplychaindive.com/feeds/news/
https://www.manufacturingdive.com/feeds/news/

# Materials, Energy & Off-Grid Tech
https://cleantechnica.com/feed/
https://solar.lowtechmagazine.com/feeds/all.atom.xml

5. Security & IP Protection Checklist (3-Minute Setup)
 * Lock Down Your API Key: In your GitHub repo, go to Settings → Secrets and variables → Actions → New repository secret, name it GEMINI_API_KEY, and paste your Google AI Studio key. Never paste the key into .py or .yml files.
 * Enforce Branch Isolation (staging vs. main): Create a staging branch in GitHub before running the workflow. Because daily-agent-sync.yml pushes only to staging, your main branch stays untouched until you review and merge via Pull Request.
 * Keep Unpatented Core IP Private: Only run this workflow on public repos for designs you explicitly want in the open-source domain (licensed under CERN-OHL-S v2 for hardware and GPLv3 / Apache 2.0 for code). Keep unfiled patent drafts in a separate private repository that the workflow never touches.

