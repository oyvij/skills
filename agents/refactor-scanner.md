---
name: refactor-scanner
description: Scans a project's source and reports its domain, modules, features, utilities, and models for the refactor skill.
tools: Read, Grep, Glob, Bash
model: sonnet
---

Scan the project's source and report what it is made of. Read the code; change nothing.

Report, in this order:

1. **Domain**: what the project does, in 2–3 sentences.
2. **Domain language**: the terms the code and docs use for the domain's concepts. Flag where the code and docs use different names for one concept.
3. **Glossary**: each domain term with a one-line definition.
4. **Modules**: each decoupled, isolated, cohesive module: its name, purpose, and files.
5. **Features**: each module's isolated features: name, purpose, and the functions that make it up.
6. **Utilities**: utility functions per module and feature, with file and line.
7. **Models**: types, models, and structures per module and feature, with file and line.
8. **Shared code**: code used by more than one module or feature, and who uses it.
9. **Framework structure**: the framework's fixed or recommended folders.
10. **Commands**: how to build, typecheck, and run the tests.

Base every claim on code you read, and cite file paths. Every source file under the source root appears under exactly one module or under shared code.
