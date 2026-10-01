# Reorganising

Apply the approved structure without changing what the application does. The bar is **green**: every build, typecheck, lint, and test command that passed before the move passes after it.

## 1. Baseline

Check `git status`. If the working tree has changes, ask the user to commit or stash them first, so the reorganisation lands as its own reviewable diff.

Run the project's build, typecheck, lint, and test commands found during the scan, and record each result. Commands already failing before the move are recorded as such; they are your baseline, and the bar is no _new_ failures.

## 2. Write `DESIGN.md`

Create `DESIGN.md` at the repo root before moving anything; it is the record later agents build on. It explains the decision and its purpose as **durable rules**: statements that stay true as slices are added, renamed, or removed. The tree itself is the single source of truth for what exists, so `DESIGN.md` names folder _kinds_ (a feature slice, the shared folder) and leaves concrete paths and listings to the tree. Test each line: would it still hold after someone adds a new feature? If not, rewrite it as the rule behind it.

```markdown
# Design

<One line: the established design the structure follows and its source, e.g. Vertical Slice Architecture (Bogard, 2018).>

<One paragraph: what the structure optimises for, and the principle it leads with.>

## Organising principle

<How the source root is divided (by feature, by layer, hybrid) and what each kind of folder means. The rule, stated so a reader could place a folder that doesn't exist yet.>

## Dependency direction

<Which kinds of folder may import which, and the public surface convention a slice exposes.>

## Naming

<Which vocabulary each kind of folder draws from (domain, feature, design), where the domain terms are defined, and the casing and plurality convention.>

## Where new code goes

<Placement rules: a new feature, a helper used by one slice, a helper used by several, a new type, a test.>

## Alternatives considered

<Each rejected approach in one line, with the reason it lost.>
```

## 3. Plan the moves

Write a move map: every file under the source root, old path → new path. Files that stay put are listed as staying. The map is complete when it covers every file the scan named.

## 4. Move slice by slice

For each slice in the map:

1. Move its files with `git mv`, so history follows them.
2. Update every reference to a moved path: import statements, path aliases (`tsconfig.json`, `jsconfig.json`, bundler config), manifest path fields (`main`, `exports`, `bin`, a library path, console scripts), dynamic imports and `require` strings, lazy-loaded routes, test-runner config and mock paths, coverage config, CI workflows, Dockerfiles, scripts, and doc links. Prefer the language's refactoring tooling over hand edits where it exists.
3. Add the slice's public surface file if the approved structure has one, and route imports from other slices through it. Check the import graph for new cycles the re-exports introduced.
4. Run the typecheck (or build, where there is no typecheck). Get it green before starting the next slice.

## 5. Verify

Re-run every baseline command and compare against the recorded results. Then search the whole repo for each old path from the move map; every hit is a reference you missed. If the project has a runnable entry point (dev server, CLI, main module), start it and confirm it boots.

The move is done when every baseline command matches its baseline result and the old-path search returns nothing.

## 6. Point to `DESIGN.md`

Add a context pointer to the always-loaded agent doc the project already uses (`AGENTS.md`, `CLAUDE.md`, or an equivalent the project's agents read on every run). If `AGENTS.md` and `CLAUDE.md` both exist and one doesn't already include the other, add it to both. If the project has none, ask the user where it should go. Fit the wording to the doc's style, carrying the trigger:

> Before adding, moving, or naming files under `src/`, read `DESIGN.md`: where code belongs and which folders may import which.

## 7. Report

Tell the user: the structure applied, the number of files moved, each baseline command with its before/after result, and anything left for their judgement (a fuzzy boundary, a skipped reference you couldn't verify).
