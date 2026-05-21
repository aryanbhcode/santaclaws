<!-- Back to top link -->
<a id="readme-top"></a>

<!-- PROJECT SHIELDS -->
[![Contributors][contributors-shield]][contributors-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]

<br />
<div align="center">

  <h1>Santa Claws</h1>

  <p align="center">
    <br />
    AI agents that turn local business leads into live website mockups and personalized outreach.
    <br />
    <br />
    <a href="https://github.com/aryanbhcode/santaclaws">View Repository</a>
    &middot;
    <a href="https://github.com/aryanbhcode/santaclaws/issues/new?labels=bug">Report Bug</a>
    &middot;
    <a href="https://github.com/aryanbhcode/santaclaws/issues/new?labels=enhancement">Request Feature</a>
  </p>
</div>

---

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#about-the-project">About the Project</a></li>
    <li><a href="#features">Features</a></li>
    <li><a href="#agent-team">Agent Team</a></li>
    <li><a href="#built-with">Built With</a></li>
    <li><a href="#how-it-works">How It Works</a></li>
    <li><a href="#discord-controls">Discord Controls</a></li>
    <li><a href="#database-memory">Database Memory</a></li>
    <li><a href="#project-structure">Project Structure</a></li>
    <li><a href="#running-locally">Running Locally</a></li>
    <li><a href="#running-on-brev">Running On Brev</a></li>
    <li><a href="#troubleshooting">Troubleshooting</a></li>
    <li><a href="#what-we-learned">What We Learned</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>

---

<!-- ABOUT THE PROJECT -->
## About the Project

Santa Claws is a hackathon project that turns a small team of AI agents into an autonomous sales workshop for local businesses.

The system finds local business leads, builds polished website mockups, deploys those mockups to Vercel, drafts personalized outreach, routes approvals through Discord, and shows the full workflow in a live dashboard.

The core idea is simple:

> NemoClaw gives us secure always-on claws. Supabase gives them shared memory. Nemotron gives them reasoning.

The agents do not call each other directly. Supabase is the queue, shared memory layer, and audit log. Every important action writes a human-readable row to the `actions` table so the dashboard can show what the system is doing in real time.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- FEATURES -->
## Features

- **Four-agent pipeline:** Scout, Designer, Pitcher, and Closer each own one stage of the workflow
- **Persistent memory:** Supabase stores leads, generated sites, outreach, approvals, logs, replies, meetings, and agent memory
- **Lead discovery:** Scout uses Apify to find local businesses and qualify email-ready rows
- **Website generation:** Designer creates professional industry-specific mockups and deploys the selected version to Vercel
- **Personalized outreach:** Pitcher drafts outreach with the exact Vercel mockup URL copied from the database
- **Human approval loop:** Discord commands can approve, skip, edit, or run agents on demand
- **Autonomous mode:** `AUTONOMOUS_MODE=true` can auto-approve and send for demo runs
- **Live dashboard:** Next.js dashboard shows metrics, leads, agent pages, generated sites, activity logs, and memory
- **Demo fallbacks:** Seed data and fallback behavior keep the project presentable when a live API fails

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- AGENT TEAM -->
## Agent Team

| Claw | Role | What it does |
|---|---|---|
| Rudolph Scout | Lead Finder | Finds local businesses, enriches rows, and qualifies email-ready leads |
| Workshop Elves | Designer | Builds polished website mockups and deploys the winner to Vercel |
| Snowball Pitcher | Outreach Courier | Writes personalized emails and queues them for approval or sending |
| Cookie Closer | Reply Handler | Handles inbound email replies and moves warm leads toward meetings |

Each claw runs as a Python heartbeat through the NemoClaw and OpenClaw-compatible context files in `agents/<claw>/`.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- TECH STACK -->
## Built With

[![Next.js][Next.js-badge]][Next-url]
[![React][React-badge]][React-url]
[![TypeScript][TypeScript-badge]][TypeScript-url]
[![TailwindCSS][Tailwind-badge]][Tailwind-url]
[![Python][Python-badge]][Python-url]
[![Supabase][Supabase-badge]][Supabase-url]
[![PostgreSQL][PostgreSQL-badge]][PostgreSQL-url]
[![Vercel][Vercel-badge]][Vercel-url]
[![Discord][Discord-badge]][Discord-url]

Additional services:

- NemoClaw and OpenShell-compatible runtime files
- Nemotron 3 Nano Omni 30B reasoning
- Apify Google Places actor
- Resend or SMTP for email
- Brev Ubuntu instance for the live demo runtime

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- HOW IT WORKS -->
## How It Works

```text
Lead discovery -> Website mockup -> Personalized pitch -> Approval or send -> Reply handling -> Meeting workflow
```

1. Scout finds local businesses and writes rows to `leads`
2. Scout enriches and qualifies email-ready leads for mockups
3. Designer picks a qualified lead, builds mockup variants, chooses the winner, and deploys to Vercel
4. Designer writes the Vercel URL to `generated_sites.vercel_url`
5. Pitcher drafts outreach and pastes the exact Vercel URL into the email
6. Discord approval or autonomous mode marks outreach as approved
7. Pitcher sends through Resend or SMTP
8. Replies land in `inbound`
9. Closer classifies replies, drafts follow-ups, and handles meeting workflow

The workflow is intentionally database-driven. Supabase is the queue:

```text
SELECT work -> claim row -> run tool -> write result -> log action -> next heartbeat
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- DISCORD CONTROLS -->
## Discord Controls

The Discord worker handles approvals and can run claws on command.

```text
HELP
RUN SCOUT
RUN DESIGNER
RUN PITCHER
RUN CLOSER
RUN ALL
APPROVE <outreach_id>
SKIP <outreach_id>
EDIT <outreach_id> <new body>
```

Start the worker:

```bash
python -m workers.discord_bridge
```

The worker loads the repo-root `.env` file, so it should be run from the same checked-out repo used by the agents.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- DATABASE MEMORY -->
## Database Memory

| Table | Purpose |
|---|---|
| `leads` | Businesses found by Scout and moved through the pipeline |
| `generated_sites` | Designer mockups, chosen winner metadata, and Vercel URLs |
| `outreach` | Pitcher drafts, approvals, send status, and email bodies |
| `actions` | Human-readable claw activity logs for the dashboard |
| `approvals` | Discord or fallback approval decisions |
| `inbound` | Inbound email replies for Closer |
| `meetings` | Booked or demo-fallback meetings |
| `agent_memory` | Durable per-agent memory patterns |

Apply the schema from:

```text
agents/scripts/setup_supabase.sql
```

Run that file in the Supabase SQL editor.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- PROJECT STRUCTURE -->
## Project Structure

```text
santaclaws/
├── agents/
│   ├── scout/                    # Lead discovery claw
│   ├── designer/                 # Mockup generation claw
│   ├── pitcher/                  # Outreach drafting and sending claw
│   ├── closer/                   # Inbound reply claw
│   ├── integrations/             # Apify, Vercel, Resend, SMTP, Google Calendar, Vapi clients
│   ├── prompts/                  # Nemotron prompt templates
│   ├── scripts/                  # Preflight, seed data, setup SQL, NemoClaw runner
│   └── shared/                   # Supabase client, logger, memory, runtime helpers
│
├── dashboard/
│   ├── app/                      # Next.js App Router pages and API routes
│   ├── components/               # Dashboard UI components
│   ├── lib/                      # Supabase client, types, branding
│   └── public/                   # Static dashboard assets
│
├── docs/                         # Execution spec, runbooks, progress log, fallback notes
├── nemoclaw/                     # NemoClaw and OpenShell setup notes
├── workers/                      # Discord and webhook workers
└── README.md
```

Useful docs:

- [Execution spec](docs/Mainstreet_NemoClaw_Codex_Execution_Spec.md)
- [Progress log](docs/PROGRESS.md)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- RUNNING LOCALLY -->
## Running Locally

**Prerequisites:** Python 3.11, Node.js 18+, npm, Supabase project, NemoClaw or direct Nemotron credentials, Apify token, and Vercel token.

Clone the repo:

```bash
git clone https://github.com/aryanbhcode/santaclaws.git mainstreet
cd mainstreet
```

Create the Python environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r agents/requirements.txt
```

Create the root env file:

```bash
cp .env.example .env
nano .env
```

Core `.env` values:

```bash
NEMOTRON_BASE_URL=https://inference.local/v1
NEMOTRON_MODEL=nvidia/nemotron-3-nano-omni-30b-a3b-reasoning
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_KEY=
APIFY_TOKEN=
VERCEL_TOKEN=
VERCEL_PROJECT_NAME=santa-claws
RESEND_API_KEY=
OUTREACH_FROM_ADDRESS=
DISCORD_BOT_TOKEN=
DISCORD_APPROVAL_CHANNEL_ID=
AUTONOMOUS_MODE=false
SCOUT_DEMO_FALLBACK=true
```

Install dashboard dependencies:

```bash
cd dashboard
npm install
```

Create `dashboard/.env.local`:

```bash
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_URL=
SUPABASE_SERVICE_KEY=
```

Run the dashboard:

```bash
npm run dev
```

Open:

```text
http://localhost:3002
```

Run one heartbeat at a time from the repo root:

```bash
source .venv/bin/activate
python -m agents.scripts.openclaw_run scout --once
python -m agents.scripts.openclaw_run designer --once
python -m agents.scripts.openclaw_run pitcher --once
python -m agents.scripts.openclaw_run closer --once
```

Run every claw once:

```bash
python -m agents.scripts.openclaw_run all --once
```

Validation:

```bash
python -m compileall agents workers
python -m agents.scripts.preflight_check --skip-live
cd dashboard && npm run typecheck
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- RUNNING ON BREV -->
## Running On Brev

Open the Brev instance:

```bash
brev shell santaclaws1
```

Update the repo:

```bash
cd ~/mainstreet
git pull
source .venv/bin/activate
```

Run the dashboard:

```bash
cd ~/mainstreet/dashboard
npm run dev
```

Run agents from another Brev shell:

```bash
cd ~/mainstreet
source .venv/bin/activate
python -m agents.scripts.openclaw_run scout --once
python -m agents.scripts.openclaw_run designer --once
python -m agents.scripts.openclaw_run pitcher --once
```

Run Discord:

```bash
cd ~/mainstreet
source .venv/bin/activate
python -m workers.discord_bridge
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- TROUBLESHOOTING -->
## Troubleshooting

**Discord worker says it is not running**

Make sure these are in the root `.env` on the machine running the bot:

```bash
DISCORD_BOT_TOKEN=
DISCORD_APPROVAL_CHANNEL_ID=
```

Then restart:

```bash
python -m workers.discord_bridge
```

**Scout gets Apify HTTP 402**

Confirm the new token is loaded without printing the secret:

```bash
python - <<'PY'
from dotenv import load_dotenv
from pathlib import Path
import os
load_dotenv(Path.cwd() / ".env")
token = os.getenv("APIFY_TOKEN", "").strip()
print("APIFY_TOKEN set:", bool(token), "len:", len(token), "last4:", token[-4:])
PY
```

Restart any running Discord bot or Scout process after changing `.env`.

**Designer finds no qualified lead**

Designer only works on email-ready leads that Scout has qualified:

```sql
select qualification_status, worked_by_designer, count(*)
from leads
group by qualification_status, worked_by_designer;
```

**Pitcher drafts but does not send**

Pitcher sends only approved outreach. Approve through Discord:

```text
APPROVE <outreach_id>
```

Then run:

```bash
python -m agents.scripts.openclaw_run pitcher --once
```

**Dashboard env missing**

Use `dashboard/.env.local`, not the root `.env`, for browser-safe dashboard variables:

```bash
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- WHAT WE LEARNED -->
## What We Learned

The hardest part of Santa Claws was not making one model call. It was making a multi-agent system reliable enough to demo.

We learned that agentic products need persistent memory, visible logs, clear queues, exact external links, careful environment loading, and simple human controls.

The dashboard became just as important as the agents because it made the autonomous work legible.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- LICENSE -->
## License

Private hackathon project, not licensed for external use.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- CONTACT -->
## Contact

**Aryan Bhatia**

[![LinkedIn][linkedin-shield]][linkedin-url]
[![GitHub][github-shield]][github-url]

Project: [https://github.com/aryanbhcode/santaclaws](https://github.com/aryanbhcode/santaclaws)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- ACKNOWLEDGMENTS -->
## Acknowledgments

Built during a 24-hour hackathon with a focus on demo reliability, persistent agent memory, and making autonomous work visible.

Thanks to the NemoClaw, Nemotron, Supabase, Vercel, Apify, Discord, and Brev ecosystems for the pieces that made the demo possible.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- MARKDOWN LINKS -->
[contributors-shield]: https://img.shields.io/github/contributors/aryanbhcode/santaclaws.svg?style=for-the-badge
[contributors-url]: https://github.com/aryanbhcode/santaclaws/graphs/contributors
[stars-shield]: https://img.shields.io/github/stars/aryanbhcode/santaclaws.svg?style=for-the-badge
[stars-url]: https://github.com/aryanbhcode/santaclaws/stargazers
[issues-shield]: https://img.shields.io/github/issues/aryanbhcode/santaclaws.svg?style=for-the-badge
[issues-url]: https://github.com/aryanbhcode/santaclaws/issues

[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: [https://www.linkedin.com/in/parthmdoshi/](https://www.linkedin.com/in/aryan-bhatia-828925239/)
[github-shield]: https://img.shields.io/badge/-GitHub-black.svg?style=for-the-badge&logo=github&colorB=555
[github-url]: https://github.com/aryanbhcode

[Next.js-badge]: https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=nextdotjs&logoColor=white
[Next-url]: https://nextjs.org/
[React-badge]: https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB
[React-url]: https://react.dev/
[TypeScript-badge]: https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white
[TypeScript-url]: https://www.typescriptlang.org/
[Tailwind-badge]: https://img.shields.io/badge/TailwindCSS-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white
[Tailwind-url]: https://tailwindcss.com/
[Python-badge]: https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white
[Python-url]: https://www.python.org/
[Supabase-badge]: https://img.shields.io/badge/Supabase-3FCF8E?style=for-the-badge&logo=supabase&logoColor=white
[Supabase-url]: https://supabase.com/
[PostgreSQL-badge]: https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white
[PostgreSQL-url]: https://www.postgresql.org/
[Vercel-badge]: https://img.shields.io/badge/Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white
[Vercel-url]: https://vercel.com/
[Discord-badge]: https://img.shields.io/badge/Discord-5865F2?style=for-the-badge&logo=discord&logoColor=white
[Discord-url]: https://discord.com/
