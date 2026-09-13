# Repository Architecture and Boundaries

This document defines the responsibilities of the three repositories currently used by the Historical Fantasy RPG / Bruges 1488 project. Its purpose is to prevent research, software infrastructure, and game-product concerns from drifting into one another as the project grows.

## Guiding principle

The project has three distinct layers:

1. **Research programme and methodology** — what we know, how we know it, how uncertainty is classified, and how research may be reused across outputs.
2. **Research infrastructure** — the executable database/API that stores and exposes structured research data.
3. **Game product** — the Unity implementation, narrative, mechanics, assets, and player-facing experience for Bruges 1488.

These layers should interact, but they should not become interchangeable.

---

## Repository responsibilities

### `historical-research-project`

**Role:** Research programme, methodology, Digital Humanities framework, and shared research-system design.

This repository is the intellectual and organisational home of the historical research programme. It is deliberately independent of any single commercial or creative output.

Belongs here:

- research methodology and evidence standards;
- provenance and uncertainty rules;
- research-programme definitions such as BRG — Bruges 1488;
- research governance and workflows;
- Digital Humanities concepts and pilot designs;
- conceptual data-model and ontology design;
- CIDOC CRM / linked-data mapping work;
- research-data, publication, licensing, and IP-boundary discussions;
- academic collaboration and grant-development material;
- reusable research outputs that may support games, video, social media, or scholarly work.

Does **not** belong here:

- Spring Boot/JPA implementation details;
- Flyway migrations;
- Unity scenes, prefabs, scripts, or game assets;
- game-specific quest logic or mechanics unless discussed as a research/output-boundary question.

**Source-of-truth responsibility:** methodology, research governance, conceptual model, and programme-level documentation.

---

### `historical-research-api`

**Role:** Executable research infrastructure — PostgreSQL database, Spring Boot API, persistence model, ingestion, retrieval, and derived search/index layers.

This repository implements the structured research model defined conceptually by the research programme.

Belongs here:

- PostgreSQL schema;
- Flyway migrations;
- Spring Boot / Java application code;
- JPA entities, repositories, services, DTOs, and controllers;
- REST API contracts;
- structured entities, claims, sources, citations, evidence, dates, and relationships;
- ingestion and validation workflows;
- ordinary text/search APIs;
- pgvector and semantic retrieval when introduced;
- graph projections/export code when introduced;
- machine-readable CIDOC/RDF export if/when implemented;
- automated tests for the database/API and data-integrity rules.

Does **not** belong here:

- authoritative research methodology prose except concise implementation documentation;
- the Unity game itself;
- game-only quest state, combat systems, scene composition, or player mechanics;
- unsourced historical assertions inserted merely because a narrative requires them.

**Source-of-truth responsibility:** canonical machine-readable structured research records. PostgreSQL is the canonical data store; vector indexes, graph databases, caches, and exports are derived representations and must be rebuildable from canonical data.

Current implementation sequence:

- `KB-001` — Historical Knowledge Base Core;
- `KB-002` — Source ingestion and research search workflow;
- `KB-003` — Semantic retrieval with pgvector.

---

### `bruges-1488`

**Role:** Unity game product and player-facing implementation.

This repository contains the actual Bruges 1488 game and everything needed to build, test, and ship it.

Belongs here:

- Unity project files and settings;
- C# game code;
- scenes, prefabs, tilemaps, lighting, cameras, UI, and assets;
- game-specific design documents;
- quests, dialogue, branching narrative, companions, mechanics, encounter design, economy, progression, and endings;
- game-specific adaptations of historical research;
- integration code that consumes research/API data or exported datasets;
- game testing and build configuration.

Does **not** belong here:

- the canonical historical research database;
- primary source/citation management as an independent duplicate system;
- research methodology or Digital Humanities programme governance;
- a second, conflicting store of historical truth.

**Source-of-truth responsibility:** executable game behaviour, game-specific narrative/design decisions, and shipped player-facing content.

---

## Source-of-truth model

There is no single repository that is authoritative for every concern.

| Concern | Authoritative location |
| --- | --- |
| Research methodology and evidence rules | `historical-research-project` |
| Conceptual research/data architecture | `historical-research-project` |
| Structured entities, claims, sources, citations, and provenance | PostgreSQL in `historical-research-api` |
| API contracts and database implementation | `historical-research-api` |
| Vector/graph/search indexes | Derived from `historical-research-api`; never canonical |
| Unity game implementation | `bruges-1488` |
| Quest/narrative/mechanics decisions specific to the game | `bruges-1488` |
| General reusable historical findings | Research programme + structured knowledge base, not only the game repo |

A useful shorthand is:

> **Research truth and methodology → `historical-research-project`**  
> **Machine representation and access → `historical-research-api`**  
> **Player experience → `bruges-1488`**

---

## Data and dependency flow

The preferred direction of dependency is:

```text
historical-research-project
        |
        | defines methodology, concepts, evidence rules
        v
historical-research-api
        |
        | exposes structured, provenance-aware research data
        v
bruges-1488
```

The game may request new research questions or reveal gaps, but it must not silently change the certainty of research claims. Narrative requirements can produce new research tasks or explicitly fictional/game-invention records; they do not turn inference into fact.

Likewise, graph databases, vector stores, RDF/CIDOC exports, caches, or generated game-data files should be treated as projections or derivatives of canonical research data unless a later architecture decision explicitly changes this rule.

---

## Where should a new item go?

Use these tests:

**Put it in `historical-research-project` when:** the question is about research standards, programme design, academic/DH work, conceptual ontology/data modeling, rights/governance, or reusable historical methodology.

**Put it in `historical-research-api` when:** the work changes how structured research is stored, validated, queried, searched, imported, exported, or exposed by software.

**Put it in `bruges-1488` when:** the work changes what Unity builds or what the player sees, hears, does, chooses, fights, explores, or experiences.

If work crosses boundaries, create linked issues in the relevant repositories rather than placing the entire concern in one repo. For example, a new historical relationship type may require a conceptual decision in `historical-research-project`, an implementation issue in `historical-research-api`, and later a game-consumption issue in `bruges-1488`.

---

## Identifier conventions

Repository issue numbers are local implementation/tracking identifiers and should not replace conceptual research IDs.

Current conventions:

- `HRP-###` — programme-wide research infrastructure, methodology, governance, or tooling in `historical-research-project`;
- `BRG-###` — Bruges 1488 research-programme development in `historical-research-project`;
- `RES-###` / `WKS-###` — existing subject-level historical research items;
- `KB-###` — Historical Knowledge Base implementation work in `historical-research-api`;
- game-specific issue conventions may be maintained separately in `bruges-1488` as that backlog grows.

The same conceptual research item may be referenced from more than one repository without being renumbered.

---

## Architectural guardrails

1. **Do not duplicate canonical research truth inside Unity.** The game may cache/export the subset it needs, but provenance-aware research remains upstream.
2. **Do not let narrative need determine historical certainty.** Game invention remains explicitly distinguishable from documented fact and inference.
3. **Do not make derived search/graph systems authoritative.** pgvector, Neo4j/AGE, RDF exports, and similar systems are projections unless explicitly promoted by a later architectural decision.
4. **Keep conceptual and implementation decisions traceable.** Significant changes to the research model should be documented in `historical-research-project` and linked to implementation issues in `historical-research-api`.
5. **Preserve reuse.** Research should remain usable for video, social media, scholarly/DH work, maps, timelines, and future games rather than being trapped inside Bruges 1488 game code.

---

## Current architecture

```text
+--------------------------------------+
| historical-research-project          |
| methodology · programmes · DH · IP   |
| conceptual data/ontology design      |
+-------------------+------------------+
                    |
                    v
+--------------------------------------+
| historical-research-api              |
| PostgreSQL · Spring Boot · Flyway     |
| claims · sources · citations · API   |
| later: pgvector / graph projections  |
+-------------------+------------------+
                    |
                    v
+--------------------------------------+
| bruges-1488                          |
| Unity · C# · scenes · quests · UI    |
| narrative · mechanics · game assets  |
+--------------------------------------+
```

This architecture may evolve, but changes should preserve the separation between research governance, research infrastructure, and product implementation unless there is a clear reason to merge responsibilities.
