<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:7F00FF,100:E100FF&height=210&section=header&text=BountyCraft%20Skills&fontSize=52&fontColor=ffffff&animation=fadeIn&desc=Reusable%20Claude%20Code%20skills%2C%20one%20per%20vulnerability%20class&descSize=18&descAlignY=64" alt="BountyCraft Skills" width="100%"/>

[![Typing SVG](https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=21&duration=3000&pause=800&color=A855F7&center=true&vCenter=true&width=760&lines=Recognize+it.+Test+it.+Report+it.;One+skill+per+vulnerability+class.;Synthesized+from+real+disclosed+writeups.;Validated+against+PortSwigger+expert+labs.)](https://github.com/AmanJ24/claude-bountycraft-skills)

<br/>

![Phase](https://img.shields.io/badge/phase-research%20only-7F00FF?style=for-the-badge&logo=bugcrowd&logoColor=white)
![Skills](https://img.shields.io/badge/skills-7-E100FF?style=for-the-badge)
![Sources](https://img.shields.io/badge/sources-171-8B5CF6?style=for-the-badge)
![Notes](https://img.shields.io/badge/extracted%20notes-150-A855F7?style=for-the-badge)
![Validated](https://img.shields.io/badge/validated-PortSwigger%20labs-22C55E?style=for-the-badge&logo=portswigger&logoColor=white)

</div>

---

## What this is

A library of **reusable [Claude Code](https://claude.com/claude-code) skills, one per web vulnerability class.** Each skill is not paraphrased theory. It is distilled from real, disclosed bug bounty writeups (HackerOne reports, PortSwigger Research, Medium, Pentester Land) and then **validated against live PortSwigger labs** before it ships.

The goal: hand an agent (or a human hunter) a single lean file that teaches how to **recognize, test for, and report** a vuln class the way real hunters actually find it in the wild.

> [!NOTE]
> **Research phase only.** These skills produce methodology and guidance. They do not fire requests at live targets. Active, scope gated testing is a deliberate, later capability (see [Roadmap](#roadmap)), not part of this toolkit today.

---

## Skills at a glance

| Skill | Class | Status | Progress | Sources | Notes | Lab result |
|:------|:------|:------:|:---------|:-------:|:-----:|:-----------|
| 🛰️ **SSRF** | Server Side Request Forgery | ✅ Complete | `██████████` 100% | 30 | 26 | 4/5 green |
| 🔑 **IDOR** | Insecure Direct Object Ref | ✅ Complete | `██████████` 100% | 26 | 25 | **5/5 green** |
| 💉 **XSS** | Cross Site Scripting | ✅ Complete | `██████████` 100% | 31 | 30 | 4/5 green + 1 proven |
| 🧩 **Business Logic** | Price / workflow / payment | ✅ Complete | `██████████` 100% | 24 | 24 | 3/6 green |
| 🏁 **Race Conditions** | Single packet / TOCTOU | ✅ Complete | `██████████` 100% | 20 | 19 | technique proven |
| 🚪 **Access Control** | Auth bypass / privesc | ✅ Complete | `██████████` 100% | 24 | 24 | **5/5 green** |
| 🗄️ **SQL Injection** | In-band / blind / OOB / RCE | ✅ Complete | `██████████` 100% | 16 | 16 | 5/5 in-band green |

<sub>Lab results are PortSwigger labs driven end to end. "proven" / "technique proven" means the method was demonstrated but the final green banner was blocked by a harness limit (out of band callbacks, the single packet attack, or JS execution timeouts), not by a gap in the skill. Details live in each skill's `evals/` folder.</sub>

---

## The research pipeline

Every skill is built the same disciplined way. No shortcuts around sourcing.

```mermaid
flowchart LR
    F["🔍 Find"] --> FI["🧹 Filter"] --> SC["📥 Scrape"] --> EX["📝 Extract"]
    EX --> SY["🧠 Synthesize"] --> TE["🧪 Test / Eval"] --> IT["🔁 Iterate"] --> CO["✅ Commit"]

    style F fill:#7F00FF,stroke:#fff,color:#fff
    style FI fill:#8B5CF6,stroke:#fff,color:#fff
    style SC fill:#9333EA,stroke:#fff,color:#fff
    style EX fill:#A855F7,stroke:#fff,color:#fff
    style SY fill:#C026D3,stroke:#fff,color:#fff
    style TE fill:#E100FF,stroke:#fff,color:#fff
    style IT fill:#D946EF,stroke:#fff,color:#fff
    style CO fill:#22C55E,stroke:#fff,color:#fff
```

| Step | What happens |
|:-----|:-------------|
| **1. Find** | Search Medium, HackerOne hacktivity, PortSwigger Research, Pentester Land for the class. |
| **2. Filter** | Keep writeups with real root cause and impact, ideally a disclosed report. Quality over quantity. |
| **3. Scrape** | Pull full text and metadata (title, author, URL, date, bounty) into `sources/`. |
| **4. Extract** | Distill each writeup into one paraphrased, structured note in `extracted/`. |
| **5. Synthesize** | Merge all notes into one lean `SKILL.md`, push bulky detail into `references/`. |
| **6. Test** | Run the skill against a matching PortSwigger lab. Does it actually help find the bug? |
| **7. Iterate** | Fix the gaps the eval surfaces, re run, then commit. |

---

## Coverage

```mermaid
pie showData
    title Extracted notes per vuln class
    "SSRF" : 26
    "IDOR" : 25
    "XSS" : 30
    "Business Logic" : 24
    "Race Conditions" : 19
    "Access Control" : 24
    "SQL Injection" : 16
```

---

## Anatomy of a skill

Each `SKILL.md` is kept lean and fast to load. Seven sections, always the same shape:

```
1. Recognize it            what the vuln looks like in the wild, common trigger points
2. Where it hides          feature types and parameters that tend to carry it
3. Test methodology        step by step approach to probing for it
4. Known bypass patterns   summarized, pointing to references/ for the full list
5. Common false positives  what looks like this vuln but is not
6. Reporting               how to write up impact convincingly once confirmed
7. Sources                 pointer to sources.md so every claim traces back
```

Bulky payload lists, edge cases, and longer examples live under each skill's `references/`.

---

## Repository layout

```
claude-bountycraft-skills/
├── sources/<class>/        raw scraped writeups (markdown, with source URL + date)
├── extracted/<class>/      one structured, paraphrased note per writeup
├── skills/<class>/
│   ├── SKILL.md            lean, distilled skill definition
│   ├── references/         detailed payloads, edge cases, longer examples
│   └── sources.md          full, credited source list for the skill
├── evals/<class>/          PortSwigger lab cases + recorded outcomes
└── progress.md             live status of every vuln class
```

| Directory | Role |
|:----------|:-----|
| `sources/` | The raw material. Every writeup that fed a skill, traceable to its original URL. |
| `extracted/` | The distillation. One note per writeup, paraphrased, following a fixed template. |
| `skills/` | The product. What an agent actually loads and applies. |
| `evals/` | The proof. Each skill run against a real lab, outcome recorded honestly. |

---

## Hard rules (non negotiable)

These keep the work both useful and clean.

- 🧪 **Research phase only.** Methodology and guidance. No requests to live targets.
- ✅ **Authorized scope only**, always, before any future active testing phase.
- 🚫 **No paywall bypassing.** Paywalled writeups are retrieved legitimately or flagged, never circumvented.
- ✍️ **Paraphrase, never copy.** Source text is distilled into our own words. Verbatim copying is checked for and removed.
- 🔗 **Track every source.** Title, author, URL, and the disclosed report link, so every claim can be traced and credited.

---

## Roadmap

```mermaid
gantt
    title Vuln class skills
    dateFormat X
    axisFormat %s
    section Shipped
    SSRF              :done, 0, 1
    IDOR              :done, 0, 1
    XSS               :done, 0, 1
    Business Logic    :done, 0, 1
    Race Conditions   :done, 0, 1
    Access Control    :done, 0, 1
    SQL Injection     :done, 0, 1
    section Backlog
    SSTI              :0, 1
    NoSQL Injection   :0, 1
```

**Phase 2 and 3 (future, gated):** assistive mode where a skill suggests an exact request for a human to fire manually, then a tightly controlled semi autonomous mode where the target is passed in explicitly each run, a scope file is checked before anything fires, and hard rate limits apply. Not built unless deliberately requested.

---

<div align="center">

<sub>Built as a practical AppSec toolkit and a portfolio signal. Methodology distilled from the people who actually found these bugs, credited in every <code>sources.md</code>.</sub>

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:E100FF,100:7F00FF&height=120&section=footer" alt="" width="100%"/>

</div>
