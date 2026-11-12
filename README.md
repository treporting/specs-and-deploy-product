# specs-and-deploy-product

The specification of treporting, the product: what it is needed for, what it has to do, how it is
broken down, and what can go wrong with it. It is written for whoever specifies, reviews or builds
the product, and for the auditor who looks for the evidence.

| File | What |
|---|---|
| [requirements/user-needs.yaml](requirements/user-needs.yaml) | the user needs (r1), explained in [user-needs.md](requirements/user-needs.md) |
| [requirements/system-requirements.yaml](requirements/system-requirements.yaml) | the system requirements (r2), explained in [system-requirements.md](requirements/system-requirements.md) |
| [architecture/system-architecture.yaml](architecture/system-architecture.yaml) | the system architecture (r3), explained in [system-architecture.md](architecture/system-architecture.md) |
| [safety/hazards.yaml](safety/hazards.yaml) | the hazards and threats (h1) and their controls (h2), explained in [hazard-and-risk-analysis.md](safety/hazard-and-risk-analysis.md) |
| [levels.yaml](levels.yaml) | the levels of the product; the repos of the subsystems take theirs from here |
| [reviews/](reviews/) | the criteria of each level, and the result of every review made in this repo |
| [documents.yaml](documents.yaml) | every Markdown file, with its version and status |

The rules of the work, the formats, the schemas and the regulatory requirements lie in
[specs-and-deploy](https://github.com/treporting/specs-and-deploy), the repo above. The tools that
check this repo lie in [tools](https://github.com/treporting/tools); its CI runs them against both.

Until 2026-11-12 these files lay in specs-and-deploy; their reviews up to then are there, as the
evidence of the history they name (specs-and-deploy#637).
