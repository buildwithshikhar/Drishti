DRISHTI — Master Implementation Plan

AI-Powered IPsec VPN Protocol Analysis Platform — SIH 2026, PS26160

Engineering Conventions (apply to every phase, no exceptions)

Commits: Conventional Commits format — type(scope): description. Types: feat, fix, docs, refactor, test, chore, perf. One logical change per commit, never a giant "phase complete" dump. Example: feat(testbed): add strongSwan container orchestration for IKEv2 tunnels. Commit after every working increment, not just at phase end — judges and reviewers read commit history as a proxy for real, incremental development, not a last-minute dump.

Branching: trunk-based — work on main with small, frequent commits. A hackathon timeline doesn't benefit from long-lived feature branches; it benefits from a commit history that visibly shows steady progress across days.

Docs — update every phase, not at the end:

README.md — kept current from Phase 0 onward (see spec below)
docs/ARCHITECTURE.md — updated whenever a new component is added
docs/API.md — updated whenever a backend endpoint is added/changed
docs/DATASET.md — updated in Phase 4 and kept current
docs/MODEL_CARD.md — added in Phase 5, updated whenever a model changes (accuracy, training data, limitations — standard ML documentation practice)
CHANGELOG.md — one line per meaningful commit, grouped by phase

Definition of Done for every phase includes: working code + passing basic tests + docs updated + committed + (from Phase 1 onward) deployed and verified live at the public URL.

README.md spec (write this properly in Phase 0, refine every phase after)

Must include: one-line pitch, problem statement summary (PS26160), architecture diagram (embed the Mermaid diagram from earlier), tech stack table, setup/run instructions (local + Docker), screenshots (added as each screen ships), dataset description + download link, model performance summary, live demo link, demo video link, team credits, license. This is the first thing judges open — it should read as a finished product's README, not a TODO list.

Phase 0 — Scaffolding & First Deploy (Day 1)
Monorepo: frontend/ (React + Vite + Tailwind), backend/ (FastAPI), ml/ (Python training/inference), testbed/ (Docker Compose, strongSwan containers)
GitHub Actions: lint + build check on every push
Deploy empty shells immediately — frontend → Vercel, backend → Railway/Render — so a public URL exists from commit #1
Write initial README (pitch, PS summary, architecture placeholder, setup instructions)
Commit checkpoint: chore: initial monorepo scaffold + CI + deploy pipeline, docs: initial README
DoD: public URL live, /health returns 200, README readable
Phase 1 — MVP UI Shell (Days 2–4)
Sidebar/topbar shell matching your established design system (navy/orange palette, consistent with CyberRakshak)
Testbed Manager screen: config form (mode, cipher, DH group, PFS toggle, IP version, traffic type) — saves config to DB, no real tunnel yet
Dashboard shell: placeholder cards (Sessions Analyzed, Avg Risk Score, Model Confidence, Recent Captures)
Commit checkpoint: feat(frontend): dashboard + testbed manager UI shell
DoD: you can create and list a testbed config through the UI, deployed and clickable
Phase 2 — Real Testbed Generation (Days 5–8)
Docker Compose with strongSwan/Libreswan IPsec peer containers, parameterized by Phase 1's config
Backend endpoint that launches a real container pair and brings the tunnel up with chosen parameters
Traffic generators per container: iperf/curl (web), SIPp (VoIP), ping (ICMP), a video-stream stub
Commit checkpoint: feat(testbed): strongSwan orchestration + parameterized tunnel launch, docs: update ARCHITECTURE.md with testbed layer
DoD: "Launch Testbed" in the UI stands up a real, live IPsec tunnel
Phase 3 — Live Traffic Capture (Days 8–10)
tcpdump/Scapy capture service per container, writing PCAPs to shared storage
Traffic Capture screen: start/stop, live IKE/ESP/AH packet counters, session list with PCAP download
Note: since this is traffic between your own containers on a network you control, this runs live in your hosted deployment — not local-only demo mode
Commit checkpoint: feat(capture): live PCAP capture pipeline
DoD: every testbed run produces a downloadable, ground-truth-labeled PCAP
Phase 4 — Dataset Assembly (Days 10–12)
Scapy parser: extract IKE negotiation fields + ESP/AH flow statistics (size, inter-arrival timing, direction) into feature CSVs
Join with ground-truth labels from Phase 2
Run the config matrix (~50–150 sessions across mode × cipher × DH group × PFS × IP version × traffic type)
Pull in USBVPN2022 and ISCXVPN2016 for cross-validation
Dataset Library screen: browse sessions, label distribution, train/val/test export
Write docs/DATASET.md — schema, generation methodology, splits, external datasets used and why
Commit checkpoint: feat(dataset): labeled feature extraction pipeline + config matrix run, docs: add DATASET.md
DoD: versioned, downloadable dataset satisfying the PS's explicit dataset deliverable
Phase 5 — AI Classification Engine (Days 12–17)
Baseline: IKE-field parser (version, mode, cipher, auth, DH group) — deterministic, fast, high-accuracy win
Traffic-type-in-ESP classifier: build both a GBM-on-features baseline and a FlowPic-style CNN (2D packet-size × inter-arrival-time histogram image classifier) — report the comparison
Implementation fingerprinting: JA3-style hash of IKE proposal ordering/timing to identify vendor stack (strongSwan/Cisco/Huawei/etc.) — ties directly to the Bleichenbacher-oracle vulnerability research (Felsch et al., USENIX 2018)
Calibrate confidence scores (temperature scaling minimum; conformal prediction if time allows) — don't ship raw softmax as "AI Confidence Score"
Model Studio screen: confusion matrices, accuracy/F1 per label, confidence calibration plot
Live Classifier screen: upload PCAP or point at running session → real-time predictions with calibrated confidence
Write docs/MODEL_CARD.md
Commit checkpoint: feat(ml): IKE parser + FlowPic traffic classifier + implementation fingerprinting, docs: add MODEL_CARD.md
DoD: upload any test-split PCAP, get correct predictions with a defensible confidence score
Phase 6 — Security Assessment Engine (Days 17–20)
Rule + CVE-grounded scoring: crypto strength, config compliance, key lifetime, replay protection, PFS status, cipher-suite strength, metadata exposure
Map detected configs to named vulnerabilities — small DH groups → Logjam-class weak-DH, Aggressive Mode + PSK → offline dictionary exposure, fingerprinted vendor stack → Bleichenbacher-oracle exposure
Metadata exposure sub-score: quantify what leaks despite encryption (outer IP addresses in Tunnel mode, packet-size/timing fingerprintability)
Assessment Console screen: per-session scorecard, pass/warn/fail per check
Threat Matrix screen: attack class × config weakness grid, citing real vulnerability names
Commit checkpoint: feat(assessment): CVE-grounded security scoring engine + threat matrix
DoD: every session gets a Risk Score backed by named, real vulnerability classes, not generic labels
Phase 7 — Reporting (Days 20–22)
Reports screen: Executive Report (plain-language + Risk Score) and Technical Report (full findings), PDF export
Embed Threat Matrix + calibrated AI Confidence Score in both
Commit checkpoint: feat(reports): executive + technical PDF report generation
DoD: one-click PDF per session
Phase 8 — Autonomous Red-Team Agent (Days 22–25)
Agent (LangGraph or similar) that self-directs exploration of the config space: given a goal ("find the weakest configurations"), it launches testbed combos, captures, classifies, scores, and returns a ranked report of worst offenders — unsupervised, across the full ~700-combination space
Red-Team Agent screen: set a goal, watch it run, review ranked findings
Commit checkpoint: feat(agent): autonomous config-space exploration agent
DoD: agent run produces a ranked vulnerability report without manual per-config testing
Phase 9 — Polish, Docs, Demo, Final Deploy (Days 25–27)
Auth, empty-state handling, error states, loading states — no broken empty screens during a live demo
Finalize all /docs files, finalize README with screenshots and live results
Record demonstration video: Testbed Manager → Live Classifier → Red-Team Agent → Assessment → Report, in that narrative order
Final deployment verification: full pipeline works end-to-end on the public URL
Commit checkpoint: chore: final polish + demo prep, docs: finalize README and all docs
DoD: public URL, working demo video, complete docs, full PS deliverable checklist satisfied