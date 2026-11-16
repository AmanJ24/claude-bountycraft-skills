# Bug Bounty Vuln-Class Skills — Project Guide

## Goal
Build reusable Claude Code skills, one per vulnerability class, synthesized from
real-world bug bounty writeups (Medium, HackerOne disclosed reports, PortSwigger
Research, Pentester Land) — not from paraphrased theory. Each skill should teach an
agent how to recognize, test for, and report that vuln class the way real hunters
actually find it. Pilot vuln class: **SSRF**.

Context: I'm a cybersecurity job-seeker (AppSec / Security Engineer / Product
Security track) running a 6-week PortSwigger-based learning sprint and starting
bug bounty hunting. ~3 years hands-on via TryHackMe (top 1%, 280+ rooms). This
project is both a practical toolkit and a portfolio signal.

## Hard Rules (non-negotiable)
- **Research phase only, for now.** These skills produce methodology and
  guidance. They do not send requests to live targets. See "Phase 2/3" below for
  how active testing gets added later, deliberately and gated.
- **Authorized scope only**, always. Before any future active-testing phase is
  used against a real target, scope/program rules must be checked and recorded.
- **No paywall bypassing** when scraping Medium or elsewhere.
- **Paraphrase, never copy.** Writeup text gets distilled into our own words.
  Verbatim copying of source text into any skill file is not allowed.
- **Track sources.** Every skill keeps a source list (title, author, URL, and
  ideally the disclosed report link) so claims can be traced back and credited.
- **No automated bulk scanning** of any real site at this stage. We're building
  knowledge artifacts, not running tools against targets.

## Directory Layout
```
/sources/<vuln-class>/        raw scraped writeups (markdown, with source URL + date)
/extracted/<vuln-class>/      one structured note per writeup (see template below)
/skills/<vuln-class>/
    SKILL.md                  lean, distilled skill definition
    references/               detailed payload lists, edge cases, longer examples
    sources.md                full source list for this skill
/evals/<vuln-class>/          test cases (e.g. PortSwigger lab prompts + expected outcome)
progress.md                   checklist of vuln classes and their status
```

## Research Pipeline (per vuln class)
1. **Find** — search Medium, HackerOne Hacktivity/disclosed reports, PortSwigger
   Research blog, Pentester Land's writeup index for the vuln class.
2. **Filter** — keep writeups that: are reasonably recent, show real root cause
   and impact (not just "found reflected param"), and ideally link to a disclosed
   report. Aim for 15–25 good writeups per class; quality over quantity.
3. **Scrape** — pull full text + metadata (title, author, URL, date, bounty if
   stated) into `/sources/<vuln-class>/`.
4. **Extract** — for each writeup, fill the extraction template into
   `/extracted/<vuln-class>/`.
5. **Synthesize** — merge all extracted notes into one `SKILL.md` for the class
   (see Skill Format below). Push bulky detail into `references/`.
6. **Test** — validate the skill against a matching PortSwigger lab (or another
   known-vulnerable target). Does it actually help find/confirm the vuln?
7. **Iterate** — fix gaps the test surfaces, re-run.

Definition of done for a vuln class: SKILL.md exists, references are populated,
sources.md is complete, and the skill has been run against at least one eval case
with a passing result.

## Extraction Template (per writeup, in /extracted/)
- Vuln type / sub-variant (e.g. blind SSRF via webhook URL field)
- Target type (API, image upload, PDF generator, webhook, OAuth callback, etc.)
- Recon signal — what made the hunter suspect this spot
- Root cause — why the vuln exists
- Exploitation steps — what they actually did, in order
- Bypasses used (filter evasion, protocol tricks, redirect chains, DNS rebinding, etc.)
- Impact — what they actually achieved (metadata access, internal port scan, RCE, etc.)
- Bounty (if stated)
- One-line lesson / pattern this generalizes to
- Source: title, author, URL, date

## Skill Format (SKILL.md per vuln class)
Keep this lean — it should be fast to load and apply, not a dump of every writeup.
Sections:
1. **Recognize it** — what this vuln looks like in the wild, common trigger points
2. **Where it hides** — feature types/parameters that tend to carry this vuln
3. **Test methodology** — step-by-step approach to probing for it
4. **Known bypass / evasion patterns** — summarized, with a pointer to
   `references/bypasses.md` for the full list
5. **Common false positives** — what looks like this vuln but isn't
6. **Reporting** — how to write up impact convincingly once confirmed
7. **Sources** — pointer to `sources.md`

## Phase 2/3: Active Testing (future, not part of the pilot)
Active testing is a separate, later capability — not something this pilot builds.
When we get there:
- **Phase 2 (assistive):** skill suggests a specific request/payload (e.g. a curl
  command); a human fires it manually. No agent autonomy to send requests.
- **Phase 3 (semi-autonomous, trusted only):** target/domain is passed in
  explicitly by the human each run — never inferred or crawled. Agent checks a
  scope file before anything fires. Hard rate limits. No out-of-band/callback or
  cloud-metadata-touching requests without explicit human approval first. SSRF
  specifically needs extra caution here: "confirming" it can mean reaching real
  internal infrastructure, which crosses from PoC into unauthorized access if not
  tightly controlled.
Do not build toward this phase unless explicitly asked — flag it as a future step
and stop there.

## Working Conventions
- Default to Sonnet for scraping, filtering, and extraction (mechanical work).
- Switch to Opus for the synthesis step (merging many writeups into one
  well-judged skill) and for any future active-testing agent reasoning.
- Commit after each completed skill, not mid-pipeline.
- Ask before scraping more than ~25 pages in one run.
- Update `progress.md` after each pipeline step completes.

## Progress Tracker (seed — update as we go)
- [ ] SSRF — pilot, in progress
- [ ] IDOR
- [ ] XSS (stored/reflected/DOM)
- [ ] Auth bypass / broken access control
- [ ] Race conditions
- [ ] (add more as we go)
