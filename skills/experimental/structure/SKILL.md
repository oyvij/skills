---
name: structure
description: Propose meaningful folder structures for a project's source, then reorganise it into the one you approve and record the decision in DESIGN.md.
disable-model-invocation: true
---

# Structure

Reshape a project's source root (`src/`, or the ecosystem's equivalent: a Python package, `app/`, `lib/`) so the folder tree tells a developer or agent what the application _does_, which way data flows, and where new code belongs.

## Principles

Judge every structure, current or proposed, against these:

- **Screaming**: the top-level folders scream the domain ("billing", "catalog"), not the framework ("controllers", "components").
- **Slices**: files that change together live together. A feature slice holds its own UI, logic, and data access; deleting the feature deletes one folder. Only code used by several slices goes in a shared folder.
- **Cohesion over layering**: group by technical layer only inside a slice, or where the project is too small for slices to earn their folders.
- **Public surface**: each slice exposes what other slices may use through its module root (`index.ts`, `__init__.py`, a Go package's exported names). Everything else stays private to the slice.
- **Dependency direction**: imports flow one way (UI → domain logic → data/infrastructure; slices → shared; never back). Nesting shows it: a parent folder's code uses its children, and a child never reaches up to its parent.
  - An **entry point** is what starts the program: a binary's `main`, the server bootstrap, the app root. Entry points sit at the top of the source root, because they depend on everything and nothing depends on them. Never move an entry point below the code it uses just to fence it off. Separate the edge by giving _the code the entry point uses_ its own folders, or by leaving it flat and documented.
  - A **module root** (`lib.rs`, `__init__.py`, a package's `index.ts`) is the front door of a group of files, not an entry point. It moves with the modules it declares.
  - Dependency direction decides where each group goes. When separating two groups, evaluate both placements (A below B, and B below A) and keep the one where no child imports upward. Churn is a drawback you report, never the reason for a placement.
- **Framework-fixed paths**: paths the framework or tooling resolves by convention (Next.js `app/`, Rails `app/models`, Django app layout, test discovery globs) stay where the framework expects them. A convention is binding only for the case it was designed for. Before relying on one (Cargo's `src/bin/`, a monorepo `packages/`), confirm the project actually is that case: one binary is not "several binaries", and one app is not a monorepo. A language's default location (a crate's `src/lib.rs`, `package.json` `main`, a `pyproject` package directory) is fixed only if the manifest or build config cannot point elsewhere; check before treating it as fixed.

## Naming

A folder's name is its interface: a reader decides what lives inside, and whether to open it, from the name alone. Every folder name belongs to one of three vocabularies, and each has its source of truth:

- **Domain names** come from the glossary (`CONTEXT.md` or equivalent) and the business's own language: `invoice`, `shipment`, `policy-holder`. The glossary term wins over the code's current name.
- **Feature names** name a capability the user reaches for: `checkout`, `search`, `onboarding`. Name the capability, and keep it distinct from the domain entity it acts on.
- **Design names** name an architectural role: `shared`, `infrastructure`, `adapters`, `ui`. Each design word carries exactly one meaning across the whole tree, and a reader who knows the pattern should guess the contents correctly.

Grade every folder name against these:

- **Precise**: the name predicts the contents. A folder whose files a reader couldn't guess from its name is misnamed or holds mixed concerns.
- **Junk drawer**: `utils`, `helpers`, `common`, `misc`, `lib`, `stuff`, `manager` name nothing. Rename after what the contents do (`date-formatting`, `money`), or dissolve the folder into the slices that use it.
- **One word per concept**: one concept keeps one name everywhere (not `user` here, `account` there, `customer` elsewhere), and one name never covers two concepts.
- **Ecosystem conventions**: casing, plurality, and test colocation follow the language's conventions and the project's existing majority.

A proposed name that introduces a domain term missing from the glossary is a glossary gap: flag it in the proposal.

## Process

### 1. Scan

Read, in order: the glossary (`CONTEXT.md` or equivalent) and domain docs, `README`, `AGENTS.md`/`CLAUDE.md`, the package manifests and build config (to identify language, frameworks, entry points, path aliases, and the build, typecheck, lint, and test commands), then the source tree itself.

Record the project's own rules on structure: a ban on re-export or barrel files, a limit of one module per feature, a rule that each new file needs a stated reason, an instruction not to restructure. These constrain the candidates as much as the framework does. When a rule blocks the language's usual way of exposing a public surface or grouping files, look for another route to the same structure: a manifest or build-config path, a path alias, an inline declaration, a different package root. Rule a candidate out only when no route exists, and then tell the user which rule blocked it and which routes you checked.

Build the **import graph** of the source root: for each file, what it imports and what imports it. Use the language's tooling where it exists (`tsc --listFiles`, `madge`, `pydeps`, `go list -deps`), otherwise grep the import statements.

The scan is done when you can name, for every file under the source root, the domain feature it serves, or that it is shared, or that its location is framework-fixed, and you know the project's vocabulary: the glossary terms, the names the code uses for each concept, and where the two disagree.

### 2. Analyse

Map the scan onto the principles. Find the feature clusters in the import graph, the imports that run against the dependency direction, the cycles, and the shared folders that hold single-slice code.

Grade every folder name under the source root against [Naming](#naming): its vocabulary (domain, feature, or design), and whether it is precise, a junk drawer, or a synonym clash. The analysis is done when every folder has a grade and every failing name has a better candidate or a reason it stays.

Ask the user a question only when its answer would change which structures you propose: a feature boundary the code leaves ambiguous, an upcoming feature that would reshape the slices, a framework constraint you can't confirm from config. Ask them one at a time. Settle everything else from the code.

### 3. Propose

If the current structure already honours the principles and every folder name passes [Naming](#naming), say so, name what it does well, and stop. A sound structure with failing names gets a rename-only candidate.

Otherwise present 1–4 candidate structures in chat, the strongest first. The strongest usually follows Dependency direction: the entry point on top, the code it uses in folders below. If that candidate is not in your list, say why before presenting the others. For each:

- **Tree**: the source root to the depth where the meaning shows (usually 2–3 levels), with a short comment on each top-level folder.
- **Design**: one line naming the established design the candidate follows and its source from [RESEARCH.md](RESEARCH.md), e.g. _Vertical Slice Architecture (Bogard, 2018)_.
- **Defence**: 1–3 sentences on why this structure makes the project easier to understand, citing the principles it serves.
- **Drawbacks**: 1–3 honest costs (churn, deeper nesting, a boundary that stays fuzzy, a barrel file that risks import cycles). Measure churn in import paths rewritten, not files moved: a module root moved together with its children leaves their internal paths unchanged.
- **Dependency direction**: one line showing which folders may import which.
- **Direction check**: for each proposed folder, list its imports from the scan's import graph, mapped to the proposed paths. Count the imports that point upward (to a file in an ancestor folder) and the imports that break the candidate's own Dependency direction line. Report both counts; any nonzero count fails the candidate. Then confirm the entry point sits at the top of the source root, where a newcomer looks first.
- **Renames**: each folder renamed, old → new, with the vocabulary it now draws from. Flag glossary gaps here.

Offer only candidates that differ in a real design choice and that you would defend. If only one survives, present one. End by asking which one to apply, or whether to adjust one first.

### 4. Reorganise

When the user approves a candidate, follow [REORGANISING.md](REORGANISING.md).
