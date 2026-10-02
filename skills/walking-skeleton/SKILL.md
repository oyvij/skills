---
name: walking-skeleton
description: Find the one small code pattern a repository repeats, and teach it so every other file reads as a variant of it. Use when the user asks how a codebase works, wants a mental model before extending unfamiliar code, or asks what pattern a repo follows.
---

# Core pattern

Most repositories repeat one small **unit** in breadth and depth: a handler, a component, a controller with its service, a node in a rule file. A single file looks isolated because its caller, its inputs and the consumer of its output live elsewhere, often in a framework or interpreter the repo never calls by name. Find the unit, strip it to its **skeleton**, and expose its **contract** and **wiring**. The rest of the repo then reads as the same skeleton filled in differently.

The goal is the reader's **aha**: "the system is basically a wiring of this, that and this, so to add X I touch these files." The "this, that and this" are the parts of the walking skeleton. The unit earns that only when it explains at least **75%** of the repo's source. That is a floor, and higher is better.

## Vocabulary

- **Unit**: the smallest shape the system repeats to do its work. It is one file kind, or a **chain** of kinds that appear together in nearly every feature, such as resource, service and model, or a component and its translations. A unit can be data the program interprets, such as BPMN, rule files or plan JSON, and can be an entry inside a larger file.
- **Variant**: units that share the skeleton but differ in where their data comes from.
- **Skeleton**: the unit as a shape with slots. It keeps the lines every instance has and puts a placeholder where instances differ.
- **Walking skeleton**: the thinnest set of files and lines that would bring one instance back to life end to end if every unit were deleted, and show an outcome. Think of a village of differently shaped houses. Remove them all, and the least you need to build one house again is walls, a roof, a door and a window. Those parts are the pattern, and every house in the village is the same parts in a different shape.
- **Caller**: what invokes the unit. That is a framework, a runtime outside the repo, an interpreter inside it, or a registry lookup.
- **Contract**: who calls the unit, what it receives and from where, what it produces, and who consumes that.
- **Wiring**: what connects caller and unit. That covers the shared name (a file name equal to a diagram id, a route path, a registry key, an injected type), the registry or interpreter code, and the bootstrap.
- **Carrier**: the data flowing between units: a state document, a store, a request context, a tree of values.
- **Helper**: code shared by several features but outside the skeleton, such as mappers, formatters and utilities. Sharing decides it, not the name: a file only one feature uses belongs to that feature's unit, even when it is called `Utils`.
- **Edge**: code that feeds or consumes the pattern from outside: UIs, API clients, integrations.
- **Deviation**: code that fits no class, including dead code.

## Measuring

**Source** is every tracked file the chosen program reads at build or run time and that features edit. When the repo holds several programs, measure the chosen one, and treat the others as edges outside the count. That covers code, data the program interprets, and translation files. Tests, mocks, fixtures, generated files, vendored code, lockfiles and build or deploy config sit outside source. Spot generated files by their content as well as their path, for example a "generated" or "do not edit" header. These all stay out of the count, though the recipe still lists them when features touch them. Dead code stays out of the count too, and its share is reported as a deviation.

**Coverage** is the share of source lines held by the unit (every variant and chain kind), the carrier and the wiring. Count raw lines with `wc -l` over `git ls-files`, blanks, comments and styles included. The denominator is all of the program's source, its own helpers and edges included. Classify each file with a script, because folders often mix classes. A file that mixes classes goes to the class holding most of its lines. Code that only runs inside a unit counts as unit. Classify each file by what it does, and let the number fall where it falls. Report one coverage number, plus the second pattern's share when step 2 needs one.

Analyse the working tree as it is. When it has uncommitted changes, say so.

All work stays local and read-only. Scratch files and builds go in a temporary folder, the repo stays untouched, and nothing is deployed or sent to a remote service.

## Process

### 1. History

Read what features touch before reading code. List commits with `git log --oneline`, skipping merges, releases and version bumps, until you have about 50 that change behaviour. Hold out two **replay commits**: the two most recent whose subject line alone says they add a feature, whatever its prefix, and that change at least three files according to `git show --shortstat`. Read nothing of them beyond the subject. Note their hashes and keep their file lists unseen until step 6.

For the other commits, list the files each one touches, one commit at a time, so the held-out files never print:

```bash
git log --format=%h -n 80 --no-merges | grep -v -e <held1> -e <held2> |
  while read h; do echo "== $h"; git show --name-only --format= -M "$h"; done
  ```
  
  Note which file kinds appear together, commit after commit. When the history predates a restructure of the working tree, map its old paths to the new ones before counting.
  
  Done when you have two held-out hashes and a count of the file kinds that recur across the other commits.
  
  ### 2. Census
  
  Group source files by role. Use whatever marks the role: folder name, declared type or annotation, name suffix, export style (`export default`, `module.exports`), or the set of keys in a data file. Count files and lines per group with scripts. Read whole files only from step 3 on.
  
  The unit is the shape the caller locates by itself. A larger group that only ever runs inside another unit, such as steps inside a screen, is composition. Start from the kinds that history showed recurring, and grow the chain along the data flow, from the unit towards the carrier, with kinds that features touch together. Between candidate shapes, take the one with higher coverage, and on a tie the one with the smaller skeleton.
  
  When the repo holds several programs, take the one the user works in, or else the one with the most feature commits, and treat the others as edges.
  
  When no shape reaches 75%, the repo holds a **second pattern**. Find it in the largest area left unclassified, and give it the same treatment in brief: one sentence, a skeleton of a few lines, and its share. The two patterns together reach 75%.
  
  When two patterns still fall short, stop looking. Report the honest number and say what holds the rest.
  
  Done when you have named the unit, its variants and the carrier, and coverage is at least 75%, the second pattern brings it there, or you have reported why it can't.
  
  ### 3. Skeleton and walking skeleton
  
  Read one instance of each variant side by side, plus the smallest and largest instance of each kind. Write the skeleton for the dominant variant, keep optional sections as commented slots, and move what other variants do differently into the variants table.
  
  Then imagine deleting every unit in the repo along with its registrations, keeping the framework, shared libraries and build tooling. Write the walking skeleton that brings one unit back. List each file it needs with its minimal content: the unit with the smallest bodies that still run, its registration, the carrier fields it touches, and the config that makes it reachable. Name the outcome you would observe, such as a task appearing, a response returned or a screen rendered. A part is **necessary** when the build or the runtime fails or ignores the instance without it. Prove it by deleting that part from a scratch copy and running whatever local check exists: the compiler, the build script, the framework's validator, a test harness. The framework's own source often sits in `node_modules` or in a local container image, ready to read. When no local check covers a part, cite the runtime code that needs it.
  
  Done when all of these hold:
  
  - The skeleton fits in about 15 to 20 lines per file kind that holds logic, with plain data kinds such as models, mocks and fixtures folded into one-line comments, and every instance you read either fits it or is logged as a deviation.
  - Every part of the walking skeleton is necessary, each backed by a failing check or by the runtime code that needs it.
  - Every part maps to a kind in the chain, to the carrier or to wiring, where build config that makes the unit reachable counts as wiring. A part that maps to nothing means the chain is missing a kind, so return to step 2.
  
  ### 4. Contract
  
  For each kind in the chain, answer four questions:
  
  1. **Who calls it?** Grep for callers. When nothing in the repo calls it, a framework or runtime does. When code in the repo reads it as data, an interpreter does.
  2. **What does it receive, and from where?** Trace injected dependencies to their provider, and data to the carrier.
  3. **What does it produce?** For a component, that is what it renders and the events it emits.
  4. **Who consumes that, and what happens to it?**
  
  Evidence is a file path, a grep result, a type definition or framework documentation, inside or outside the repo. When the framework's source is out of reach, its observable behaviour counts: the call that sets it up, and what units receive from it that nothing in the repo provides. When the consumer lives outside the repo, cite the boundary where the data leaves.
  
  Done when every answer cites evidence.
  
  ### 5. Wiring and composition
  
  Find the shared name that links each unit to its caller. That name is the grep key connecting the layers. It can take more than one form, such as an element id, a topic or a message name, so list every form the caller accepts. Check it with a script across all units: every unit has a registration, and every registration has a unit. When registration is implicit, such as a directory scan, check the names units use to reference each other instead: every reference resolves to something that exists. Mismatches are deviations, and often dead code or bugs.
  
  Then check composition: does a unit start or contain other units through the same contract one level up? When it does, the pattern is fractal, and one level of nesting explains them all.
  
  Done when the script has checked every unit and its mismatches are listed.
  
  ### 6. Recipe and replay
  
  Write a recipe for each kind of change the history showed, usually adding a unit to an existing container and adding a new container. Each step names a concrete file or folder pattern from this repo. Include tests, mocks, fixtures, translations and version bumps wherever history shows features touching them.
  
  Then replay the first held-out commit. From the recipe, write down the files you expect it to touch. Only then open its diff with `git show --stat -M`.
  
  - **A missed file** is a missing step. Add it.
  - **A predicted file the commit left untouched** is an optional step or a wrong one. Mark it optional or remove it.
  - **A changed file outside source and tests**, such as `.gitignore`, is noise. Leave it out of the comparison.
  - **History older than a restructure** needs its old paths mapped to the new ones. Say so in the answer.
  
  After any correction, replay the second held-out commit the same way. When it still misses files, report them as gaps in the recipe and stop replaying.
  
  Done when the second replay matches the recipe with no missed files, or its misses are reported.
  
  ### 7. Explain
  
  Write the answer in this order, using the repo's own names and real file paths, in about 180 lines at most:
  
  1. **The pattern in one sentence**, naming the unit, the carrier and the caller. Put the coverage on its own line after it. When a second pattern was needed, add its sentence and share right after.
  2. **The walking skeleton**: each file it needs as a short code block, with comments saying what each part does, followed by the outcome you would see. The unit's code block is the skeleton.
  3. **The variants** as a table with at most six rows: variant, where it lives, where its data comes from, what it produces. Fold rare variants into one row.
  4. **Composition**: how units nest, when they do.
  5. **An analogy** to an architecture the reader knows, mapped row by row in a table. Use the one the user named. Otherwise pick the closest widely known match, such as a Redux store, an MVC controller, a middleware chain, a spreadsheet or a Unix pipe.
  6. **The recipe**: one realistic feature request walked through as numbered steps, each naming the file it touches.
  7. **Navigation**: the grep keys that connect the layers.
  8. **Deviations**: at most eight, ranked by how badly each would trip someone following the recipe. Bugs and dead code come first.
  
  Describe the one shape, and name individual modules only as examples of it. Deviations name exact files. The reader should finish able to add a feature without reading anything else.
  Ask the user if he want a html rendering of the explanation which will also include a beatiful animation in timeframes which illustrate the explanation using shapes, lines, orbs, gates, flows, input, output, moving data.
  This can really help a human internalise and visualie the walking skeleton. 