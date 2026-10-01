---
name: refactor
description: Refactor and structure a project into its natural habitat, improving scannability and readability.
disable-model-invocation: true
---

# Refactor

Restructure a project's source into folders of isolated modules and features, so a developer or agent can find every capability, function, and mutation from the tree alone.

## Rules

- **Framework structure**: keep the framework's own recommended folder structure intact. Every step below works inside it.
- **Language naming**: name folders and files the project's domain does not cover with the language's own lingo. Java uses `models` or `dto`, so a Java folder is named `models`; Go uses `struct`, so a Go folder is named `structs`.
- **Shared code**: code, or any aspect of it, shared between modules or features goes in a `shared` folder at the level that covers all of them.
- **Broken boundaries**: when placing shared code breaks a module or feature boundary, stop and ask the developer. Explain the problem in plain English and state at most three solutions that fit the repo. The developer chooses one; apply that one.
- **Black box**: each module's and feature's top-level file is a signpost: a very few functions and types that convey its purpose, so a reader knows whether to step into the levels below without reading their details.
- **Depth**: the tree is at most 4 levels deep, counted from `src/` (or the project's source root).
- **Behaviour**: the refactor changes structure only. Every behaviour, public API, and test result stays exactly as it was.

## Process

### 1. Scan

Dispatch the `refactor-scanner` agent to scan each of the project's folders and files and find its modules: decoupled, isolated, and cohesive units. Save its findings for the next steps.

Done when every source file belongs to a module or is marked as shared.

### 2. Propose

Present 1–3 solutions for the refactored structure, and mark the one you recommend. For each solution give:

- **Tree**: the source root down to the feature level.
- **Upside**: one line.
- **Downside**: one line.

Ask the developer to pick one, and wait for their choice. Steps 4–9 build the chosen solution.

### 3. Baseline

Record the current commit, then run the project's build, typecheck, and tests and record their results. If any of them fail, report the failures to the developer and stop.

### 4. Modules

Create one folder per isolated module, named after the module's purpose or domain. Move the module's source into a file within that folder.

### 5. Features

For each module, find its isolated features. Create one subfolder per feature, named after the feature's purpose, and place the feature's source code in a file within it.

### 6. Utilities

For each module and feature source, extract the utility functions into a utility file within that module's or feature's folder.

### 7. Models

For each module and feature source, extract the types, models, and structures into a models folder within that module's or feature's folder, one file per type.

### 8. Public interface

Separate each module's and feature's public interface functions from its private and protected functions. The public interface goes in the top-level file, following **Black box**.

### 9. State

Separate state mutations from pure functions.

### 10. Verify

Dispatch the `refactor-verifier` agent with the baseline from step 3. It fixes the refactor until the project compiles, the tests are green, and behaviour matches the baseline.

Done when the verifier reports every check passing. Report anything it could not fix to the developer.

### 11. Record the reasoning

Write the reasoning behind the structure to `DESIGN.md` at the repo root, so later agents keep applying it when they add or change code. State it as durable rules that still hold after a module or feature is added: how the source is divided into modules and features, what each kind of folder holds, the **Black box** signposts, where shared code goes, and each boundary decision the developer made with its reason. Leave concrete paths to the tree.

Add a pointer to `DESIGN.md` in the project's always-loaded agent doc (`AGENTS.md`, `CLAUDE.md`, or equivalent): "Before adding, moving, or naming files under `src/`, read `DESIGN.md`."

## Evaluate each step

After each of steps 4–9, answer each question below. Fix what fails before starting the next step.

- Does the refactor convey everything a developer or agent needs to find its capabilities, functions, and mutations?
- Does the tree provide the path to any query a developer or agent might have, e.g. "Where does the code do that thing I'm wondering about?"
- Reading only each module's and feature's top-level file, can a reader tell whether what they're looking for lies below it?
- Is the tree at most 4 levels deep from `src/`, and can it be shallower?
- Does the project balance the number of levels, the number of files, and the lines of code per file?

## Example

This is only an example. In a real analysis, name folders and files after the project's language, domain, modules, and features.

```
src/
├── main.ts
├── module1/
│   ├── module1.ts
│   ├── feature1/
│   │   └── feature1.ts
│   └── feature2/
└── module2/
    ├── models/
    │   ├── model1.ts
    │   └── model2.ts
    ├── feature1/
    │   └── models/
    │       ├── model1.ts
    │       └── model2.ts
    └── feature2/
```
